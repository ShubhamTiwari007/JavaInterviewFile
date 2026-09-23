# Java Multithreading: 20 Fresh Tricky Interview Questions — Answers

## 1. What happens if a thread acquires a `ReentrantLock` and never calls `unlock()`?

The lock remains held by that thread. Other threads attempting to acquire the lock may block indefinitely.

With `ReentrantLock`, the lock is associated with the owning thread. If the owning thread finishes without unlocking, the lock is not automatically released merely because the thread terminated. This can effectively make the protected section inaccessible to other threads.

**Interview point:** Always design lock ownership and release carefully.

---

## 2. Why should `unlock()` normally be placed inside a `finally` block?

Because exceptions or early returns can otherwise leave the lock held.

```java
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

The `finally` block runs when control leaves the `try` block, including because of an exception.

**Interview point:** Acquire outside `try`; release in `finally`.

---

## 3. What is the difference between `lock()`, `tryLock()`, and `lockInterruptibly()`?

### `lock()`
Waits until the lock becomes available.

```java
lock.lock();
```

The waiting thread is not interrupted out of the acquisition operation.

### `tryLock()`
Attempts to acquire immediately and returns a boolean.

```java
if (lock.tryLock()) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
}
```

There is also a timed form:

```java
lock.tryLock(2, TimeUnit.SECONDS);
```

This can prevent indefinite waiting.

### `lockInterruptibly()`
Waits for the lock but allows the waiting thread to be interrupted.

```java
lock.lockInterruptibly();
```

**Interview point:** `tryLock()` is useful for bounded/non-blocking acquisition; `lockInterruptibly()` is useful when cancellation of lock acquisition matters.

---

## 4. Can `ReentrantLock` be configured for fairness? What is the performance trade-off?

Yes.

```java
ReentrantLock lock = new ReentrantLock(true);
```

The `true` argument requests fair ordering among waiting threads.

A fair lock generally reduces the opportunity for barging and can provide more predictable access under contention. The trade-off is typically lower throughput and potentially higher scheduling overhead compared with the default non-fair lock.

**Important:** Fairness is about lock acquisition ordering, not a guarantee that every thread will receive perfectly equal CPU time.

---

## 5. What is a Read-Write Lock starvation scenario?

A `ReadWriteLock` allows multiple readers simultaneously but normally allows only one writer.

A writer can experience starvation when reader activity is continuous and the implementation/scheduling policy repeatedly permits readers to acquire the read lock while the writer waits.

Example:

```text
Reader 1 -> read
Reader 2 -> read
Reader 3 -> read
Reader 4 -> read
...
Writer -> waiting
...
more readers keep arriving
```

Whether this happens depends on the lock implementation and workload.

**Interview point:** Read-heavy concurrency can create writer starvation if writers are not given sufficient acquisition opportunity.

---

## 6. Why can a writer starve when there are continuous readers?

A writer requires exclusive access. Even though individual readers may hold the lock only briefly, a continuous stream of readers can prevent a waiting writer from obtaining the exclusive lock if new readers continue to be admitted.

This is one reason lock policy matters in read/write synchronization.

---

## 7. What is the difference between optimistic locking and pessimistic locking?

### Optimistic locking

Assumes conflicts are relatively uncommon.

A common database technique uses a version column:

```text
id | balance | version
1  | 1000    | 7
```

An update can require:

```sql
UPDATE account
SET balance = 900, version = 8
WHERE id = 1 AND version = 7;
```

If zero rows are updated, another transaction changed the record first.

### Pessimistic locking

Assumes conflicts are possible and acquires a lock before modifying the data.

For example:

```sql
SELECT *
FROM account
WHERE id = 1
FOR UPDATE;
```

The exact behavior depends on the database and transaction isolation configuration.

**Interview point:** Optimistic locking detects conflicts; pessimistic locking attempts to prevent conflicting access while the lock is held.

---

## 8. How is database optimistic locking different from Java synchronization?

Java synchronization coordinates threads that share the same JVM.

Database optimistic locking coordinates concurrent updates to persistent data and can detect conflicts across different JVMs, services, and application instances.

For example:

```text
JVM 1 -> version 10 -> update succeeds -> version 11
JVM 2 -> version 10 -> update affects 0 rows -> conflict
```

The database is the coordination point.

---

## 9. Can `synchronized` protect data stored in a database?

Not across multiple application instances.

A `synchronized` block protects threads using the same Java monitor in the same JVM.

For example:

```java
synchronized (this) {
    updateOrder();
}
```

This does not coordinate:

```text
JVM 1 -> synchronized lock A
JVM 2 -> synchronized lock B
```

Both JVMs can execute the code concurrently while updating the same database.

**Interview point:** A JVM lock is not a database lock.

---

## 10. What happens when two application instances update the same database record simultaneously?

The result depends on the database, transaction isolation, SQL statements, locking strategy, and application logic.

Possible outcomes include:

- One update overwrites another.
- One transaction waits for another.
- A transaction fails because of a lock/deadlock/conflict.
- Optimistic locking detects the conflict.
- A database constraint rejects one operation.
- Under suitable transactional design, one operation succeeds and the other is retried or rejected.

The important point is that application-level synchronization alone does not coordinate the two instances.

---

## 11. Why does a distributed application require a different approach to locking than a single JVM?

Because each JVM has its own memory and synchronization primitives.

A monitor or `ReentrantLock` in JVM A has no inherent relationship with a lock object in JVM B.

Distributed systems therefore usually coordinate through a shared authority such as:

- A relational database
- Redis
- ZooKeeper
- etcd
- A service-specific coordination mechanism

The coordination mechanism must also handle failures, timeouts, ownership, and stale locks.

---

## 12. What is a distributed lock, and when would you actually need one?

A distributed lock is a coordination mechanism that attempts to ensure that only one participant in a distributed system owns a particular lock at a time.

It can be useful when multiple application instances must perform an operation that cannot safely execute concurrently and the coordination cannot be reduced to an atomic database operation or idempotency mechanism.

Examples:

- Coordinating a singleton scheduled job
- Preventing concurrent processing of a specific resource
- Coordinating a migration or maintenance operation

**Senior-level point:** Do not introduce a distributed lock automatically. Prefer database constraints, atomic operations, transactions, or idempotency when those provide the required guarantee more simply.

---

## 13. Why can Redis-based distributed locking be tricky to implement correctly?

Because distributed locking is not just `SET key`.

A robust design must consider:

- Atomic acquisition
- Unique lock ownership/token
- Expiration
- Safe release by the owner
- Process pauses or long GC pauses
- Network failures
- Clock/timing assumptions
- Lock expiration while work is still running
- Retry behavior
- What guarantee the system actually requires

For example, blindly executing:

```text
DEL lock
```

could be dangerous if the lock expired and another process acquired it before the original process attempted the delete.

The original process could accidentally delete another owner's lock.

**Interview point:** A distributed lock requires a protocol, not merely a shared key.

---

## 14. What happens if a JVM crashes while holding a distributed lock?

It depends on the lock implementation.

If the lock has a lease/TTL, the lock can eventually expire and become available.

Without an expiration or failure-recovery mechanism, the lock could remain stuck indefinitely.

But TTL introduces another issue: the process might still be executing when the lease expires. Another process could then acquire the lock.

**Senior-level lesson:** Failure recovery and lease semantics are central to distributed locking.

---

## 15. How can a timeout on a lock prevent one failure from blocking the entire application?

Instead of waiting forever:

```java
lock.lock();
```

a timed attempt can be used:

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} else {
    // fail, retry, return a response, or use another strategy
}
```

The application can then make a bounded decision instead of allowing a permanently unavailable lock to create unbounded waiting.

**Important:** A timeout does not fix the underlying concurrency problem; it limits how long callers wait.

---

## 16. What is lock escalation, and can excessive locking create a scalability bottleneck?

**Lock escalation** is a database concept where a database may replace many fine-grained locks with a coarser-grained lock, depending on the database engine and circumstances.

More broadly, excessive locking can reduce scalability because:

```text
More contention
    -> more waiting
    -> fewer concurrent operations
    -> lower throughput
```

Large critical sections are particularly problematic.

In application code, unnecessarily locking a large section can serialize work that could safely execute concurrently.

---

## 17. Why can holding a lock while performing a network call be dangerous?

Network calls can be slow or unpredictable.

Consider:

```java
lock.lock();
try {
    callRemoteService(); // could take seconds
} finally {
    lock.unlock();
}
```

While the network call is running, other threads may be blocked.

If the remote service is slow or unavailable, lock contention can cascade into:

- Thread starvation
- Increased latency
- Request queue growth
- Connection-pool exhaustion
- Reduced throughput

**Better approach:** Keep critical sections small and avoid holding locks across slow I/O when the design permits.

---

## 18. What happens when a synchronized method calls another synchronized method on the same object?

Java monitors are reentrant.

If a thread already owns an object's monitor, it can enter another synchronized method/block guarded by that same monitor.

```java
public synchronized void methodA() {
    methodB();
}

public synchronized void methodB() {
    // allowed
}
```

The same thread can reacquire the monitor without deadlocking itself.

**Interview point:** `synchronized` is reentrant.

---

## 19. How can nested locks across different services/classes create an unexpected deadlock?

Deadlock can occur when two threads acquire multiple locks in different orders.

Example:

```text
Thread A:
    acquire Lock 1
    acquire Lock 2

Thread B:
    acquire Lock 2
    acquire Lock 1
```

Now:

```text
A holds 1 and waits for 2
B holds 2 and waits for 1
```

Neither can proceed.

The problem becomes harder to identify when the locks are hidden inside different services or library calls.

**Common prevention techniques:**

- Establish a consistent global lock ordering.
- Avoid unnecessary nested locks.
- Use timed acquisition where appropriate.
- Reduce lock scope.
- Instrument and monitor lock contention.

---

## 20. Production scenario: API has no obvious race condition, but duplicate orders occasionally appear under high traffic. How would you investigate?

Treat this as an end-to-end concurrency and data-integrity problem rather than only a Java-threading problem.

### Step 1: Reproduce and trace

Capture a correlation/request ID and an order/business key.

Determine whether two requests are genuinely creating the same logical order concurrently.

Check:

```text
request
 -> controller
 -> service
 -> database
 -> external systems
```

### Step 2: Inspect JVM-level synchronization

Look for:

- `synchronized`
- `ReentrantLock`
- Concurrent collections
- Shared mutable state
- Multiple executor threads
- Retry mechanisms

Then ask: **Is the synchronization local to one JVM?**

If there are multiple application instances, a Java lock cannot coordinate them.

### Step 3: Check database constraints

For business uniqueness, the database should generally enforce the invariant.

For example:

```sql
CREATE UNIQUE INDEX ux_order_business_key
ON orders(customer_id, idempotency_key);
```

The exact constraint depends on the business rule.

A unique constraint turns a concurrency bug into a database-enforced invariant rather than relying solely on application timing.

### Step 4: Inspect transactions

Verify:

- Transaction boundaries
- Commit timing
- Isolation level
- Whether reads and writes occur in the same transaction
- Whether transactions are unexpectedly split
- Whether retry behavior can repeat writes

### Step 5: Consider optimistic locking

If multiple actors modify the same existing record, inspect version columns and conditional updates.

Example:

```sql
UPDATE orders
SET status = ?, version = version + 1
WHERE id = ?
  AND version = ?;
```

Check the affected-row count and handle a conflict explicitly.

### Step 6: Consider pessimistic locking where appropriate

For operations that require serialized access to a database row, a database-supported row lock may be appropriate.

But locking must be used carefully because it can increase contention and deadlock risk.

### Step 7: Check idempotency

For APIs that may receive retries, use an idempotency key tied to the logical operation.

Conceptually:

```text
same idempotency key
       |
       v
same logical operation
       |
       +--> first request creates result
       |
       +--> retry returns/reuses the same result
```

This is especially important when clients, gateways, or services can retry requests.

### Step 8: Investigate distributed concurrency

With multiple application instances:

```text
Request A -> JVM 1
Request B -> JVM 2
             |
             v
         same database
```

A JVM-local lock cannot coordinate A and B.

Only introduce distributed coordination when the business operation genuinely requires it and simpler mechanisms do not provide the required guarantee.

### Step 9: Inspect external retries and messaging

Duplicate orders may originate outside the Java method itself:

- HTTP retries
- Load balancer retries
- Client retries
- Message redelivery
- Consumer retries
- Timeout followed by retry
- At-least-once delivery

The investigation should establish whether the operation is safely repeatable.

### Step 10: Verify the invariant

Define precisely what "duplicate order" means.

For example:

> For a given customer and idempotency key, at most one logical order may exist.

Then enforce that invariant at an authoritative layer, usually with a combination of:

```text
Database constraint
+
Transaction design
+
Idempotency
+
Appropriate locking/concurrency control
```

### Senior-level answer

A strong production diagnosis does not stop at:

> "Add synchronized."

Instead, ask:

1. What is the business invariant?
2. Where can concurrent requests originate?
3. Does the synchronization scope cover all competing actors?
4. Does the database enforce uniqueness?
5. Are transactions correctly scoped?
6. Can retries repeat the operation?
7. Is the operation idempotent?
8. Do we actually need distributed coordination?

---

# Interview Trap

## Q: If I use `synchronized` in my Spring Boot service, will it prevent duplicate orders?

**Not necessarily.**

`synchronized` protects code guarded by the same monitor within one JVM.

Suppose there are:

```text
3 application instances
+
multiple concurrent requests
+
one shared database
```

Each JVM has its own monitor:

```text
JVM 1 -> Lock A
JVM 2 -> Lock B
JVM 3 -> Lock C
```

Those locks do not coordinate with each other.

For cross-instance consistency, depending on the business invariant, you may need:

- Database unique constraints
- Transactions
- Optimistic locking
- Pessimistic locking
- Idempotency keys
- Distributed coordination

## Senior-Level Lesson

> **Thread safety inside one JVM ≠ distributed-system consistency.**

A Java lock answers:

> "Can these threads in this JVM enter this critical section concurrently?"

A database constraint or distributed coordination mechanism answers a different question:

> "Can multiple application instances or independent actors violate this business invariant concurrently?"

For production systems, the second question is often the more important one.
