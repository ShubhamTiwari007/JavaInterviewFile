# Java Interview Questions 1--30 --- Questions and Solutions

> **Note:** Questions 1--10 and 22--30 are included from the images/text
> provided in this conversation. The exact text for Questions 11--21 is
> not available in the files currently provided, so those sections are
> marked as missing rather than inventing question text.

------------------------------------------------------------------------

## Question 1 --- ThreadPoolExecutor

### Question

> How does ThreadPoolExecutor fundamentally manage concurrency and work
> submission?

### Solution

`ThreadPoolExecutor` manages concurrency by maintaining a **pool of
worker threads** and a **task queue**.

-   Tasks submitted via `submit()`/`execute()` are placed in a queue.
-   Available worker threads take tasks from the queue and execute them
    concurrently.
-   The pool controls the **maximum number of concurrent threads**,
    avoiding creation of a new thread for every task.
-   It can also manage excess tasks through its queue and rejection
    policy.
-   When a task finishes, the worker thread can execute another queued
    task.

**In short:** it separates **task submission** from **task execution**,
using a fixed/controlled set of reusable threads to efficiently handle
concurrent work.

------------------------------------------------------------------------

## Question 2 --- Deadlock Prevention

### Question

> What are practical deadlock prevention techniques beyond textbook
> definitions?

### Solution

Practical deadlock prevention techniques include:

-   **Consistent lock ordering:** Always acquire multiple locks in the
    same predefined order.
-   **Keep critical sections short:** Hold locks only for the minimum
    required time.
-   **Avoid nested locks:** Minimize acquiring one lock while already
    holding another.
-   **Use `tryLock()` with a timeout:** If a lock cannot be acquired
    within a limit, release acquired locks and retry or abort.
-   **Lock timeouts/watchdogs:** Detect threads waiting too long and
    recover instead of waiting indefinitely.
-   **Use higher-level concurrency utilities:** Prefer thread-safe
    collections, `ExecutorService`, semaphores, or concurrent data
    structures where appropriate instead of manually managing many
    locks.
-   **Avoid blocking operations while holding locks:** Don't perform
    I/O, network calls, or long computations inside a
    synchronized/locked section.
-   **Design for immutability/message passing:** Reduce shared mutable
    state so fewer locks are needed.

**Interview-style answer:**

> "The most effective approach is consistent lock ordering, minimizing
> lock scope, avoiding nested locks, and using timed lock acquisition
> such as `tryLock()` so a thread can back out and recover rather than
> waiting forever."

------------------------------------------------------------------------

## Question 3 --- `equals()` and `hashCode()` for Entities vs Value Objects

### Question

> What practical rules do you follow to implement equals and hashCode
> for entities versus value objects in enterprise Java?

### Solution

**For entities**, equality is usually based on a stable, unique
identity, such as a database ID. Avoid using mutable business fields in
`equals()`/`hashCode()` because changing them while the object is in a
`HashSet` or `HashMap` can break lookup behavior.

**For value objects**, equality is based on their complete value/state,
not identity. If two instances have the same relevant fields, they
should be equal. Value objects should preferably be immutable.

In both cases, follow the Java contract:

-   If `a.equals(b)` is `true`, `a.hashCode()` must equal
    `b.hashCode()`.
-   Use the same equality-defining fields in both methods.
-   `Objects.equals()` and `Objects.hash()` are useful implementations.

For JPA entities, be especially careful with generated IDs because a
transient entity may not have an ID yet. Use a well-defined stable
identity strategy rather than blindly including mutable fields.

### Example value object

``` java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Money other)) return false;
    return amount == other.amount &&
           Objects.equals(currency, other.currency);
}

@Override
public int hashCode() {
    return Objects.hash(amount, currency);
}
```

**Key distinction:** Entity → **"who is this?" (identity)**; Value
object → **"what is this?" (value).**

------------------------------------------------------------------------

## Question 4 --- List Strategy

### Question

> You must process a large stream with frequent removals from the middle
> and occasional index lookups. Which list strategy would you choose and
> why?

### Solution

For a workload dominated by **frequent middle removals** and only
**occasional index-based lookups**, a `LinkedList` can be appropriate
**if the removal is performed through an iterator or when the
node/reference is already known**.

-   `LinkedList`: removal at a known node is **O(1)**.
-   `ArrayList`: middle removal is **O(n)** because subsequent elements
    must be shifted.
-   `LinkedList`: index lookup is **O(n)**.
-   `ArrayList`: index lookup is **O(1)** but middle removal is
    **O(n)**.

**Interview answer:**

> "I'd choose `LinkedList` when the workload is dominated by middle
> removals and I can reach the element through an iterator or known
> node. The removal itself is O(1), whereas `ArrayList` must shift
> elements. Since index lookups are only occasional, the O(n) indexed
> access can be acceptable. If indexed access became frequent, I'd
> reconsider the data structure."

**Important nuance:** Calling `linkedList.remove(index)` still requires
locating the index first, so the overall operation is not simply O(1).

------------------------------------------------------------------------

## Question 5 --- HashMap Internals

### Question

> Explain the key internal concepts of HashMap that matter to everyday
> engineering decisions.

### Solution

`HashMap` stores key-value pairs using a hash table. It calculates the
key's `hashCode()` and uses the hash to determine the bucket where the
entry should be stored.

If multiple keys map to the same bucket, Java handles the collision
using a linked structure, and in modern Java, sufficiently populated
buckets can be converted to a **red-black tree** to improve lookup
performance.

Important concepts:

-   **Hashing:** `hashCode()` helps determine the bucket.
-   **Collision handling:** Multiple keys can occupy the same bucket.
-   **Treeification:** Large collision chains can become tree bins under
    the required conditions.
-   **Load factor:** Default is normally `0.75`.
-   **Resizing:** When the threshold is exceeded, the table grows and
    entries are redistributed.
-   **Complexity:** Average `get()`/`put()` is O(1) with good hashing;
    collisions can degrade performance.
-   **Equality contract:** Equal keys must have equal hash codes.
-   **Key immutability:** Fields used by `equals()`/`hashCode()` should
    not change while a key is in the map.
-   **Thread safety:** `HashMap` is not thread-safe.

For concurrent access, consider `ConcurrentHashMap`.

**Key sequence to remember:**

`hashCode → bucket → collision handling → resize/load factor → equals/hashCode contract → O(1) average → not thread-safe`

------------------------------------------------------------------------

## Question 6 --- `synchronized` vs `ReentrantLock`

### Question

> How do synchronized and ReentrantLock differ in capabilities and
> maintenance cost?

### Solution

`synchronized` is simpler and usually has lower maintenance overhead.
Java automatically acquires and releases the monitor, and the lock is
released when leaving the synchronized block even if an exception
occurs. It is reentrant.

`ReentrantLock` provides additional capabilities:

-   `tryLock()`
-   Timed lock acquisition
-   Interruptible lock acquisition
-   Optional fairness
-   Multiple `Condition` objects

The trade-off is that `ReentrantLock` requires explicit unlocking,
normally in a `finally` block.

``` java
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

**Interview answer:**

> "I use `synchronized` by default for straightforward mutual exclusion
> because it is simple and harder to misuse. I choose `ReentrantLock`
> when I specifically need features such as timed acquisition,
> interruptible locking, fairness, or multiple conditions."

**Key difference:** `synchronized` = simpler/less maintenance;
`ReentrantLock` = more control/flexibility.

------------------------------------------------------------------------

## Question 7 --- Benefits of Sealed Classes

### Question

> What real benefits do sealed classes bring to maintainability and
> correctness?

### Solution

A `sealed` class or interface restricts which classes can extend or
implement it.

This improves:

-   **Correctness:** Unexpected subclasses cannot be introduced.
-   **Maintainability:** The permitted hierarchy is explicit in the type
    definition.
-   **Exhaustive handling:** Pattern matching and `switch` statements
    can use the known set of permitted implementations.
-   **Domain modelling:** Useful when a domain intentionally has a
    closed set of variants.

Example:

``` java
public sealed interface Payment
        permits CardPayment, CashPayment {
}
```

Only the permitted classes can implement `Payment`.

**In short:** sealed classes provide controlled inheritance, stronger
compile-time guarantees, and make the intended design easier to
understand.

------------------------------------------------------------------------

## Question 8 --- String Immutability and Heap Usage

### Question

> After a refactor, memory usage increased and duplicate textual data
> appears in heap dumps. How does String immutability help you reason
> about a fix?

### Solution

`String` is immutable in Java. That means once a `String` instance is
created, its contents cannot be changed.

Immutability makes sharing safe:

-   String literals are stored in the **String pool**.
-   Identical strings can safely be shared because no caller can modify
    the shared object.
-   Java may also support string deduplication through JVM mechanisms.

After a refactor, I would:

1.  Inspect the heap dump for duplicate strings.
2.  Identify their retaining paths and allocation sources.
3.  Determine whether the refactor creates unnecessary dynamic strings.
4.  Consider reuse, caching, or interning where appropriate.
5.  Avoid blindly calling `intern()` on high-cardinality data because
    the pool itself can create memory pressure.

The important point is:

> **Immutability enables safe sharing; it does not automatically
> eliminate duplicate String objects.**

------------------------------------------------------------------------

## Question 9 --- Fail-Fast vs Fail-Safe Iterators

### Question

> What is the practical difference between fail-fast and fail-safe
> iterators?

### Solution

A **fail-fast iterator** detects structural modification of a collection
during iteration and typically throws `ConcurrentModificationException`.

Examples include iterators from:

-   `ArrayList`
-   `HashMap`

A **snapshot/weakly consistent iterator** does not necessarily fail when
the underlying collection changes.

Examples:

-   `CopyOnWriteArrayList` iterates over a snapshot.
-   `ConcurrentHashMap` provides weakly consistent iteration.

Example:

``` java
for (String s : list) {
    list.add("x"); // may cause ConcurrentModificationException
}
```

With `CopyOnWriteArrayList`, the iterator operates on a snapshot, so
concurrent modifications do not invalidate the current iterator.

**Practical distinction:** fail-fast behavior is useful for detecting
incorrect modification patterns, while concurrent/snapshot iteration is
useful when modifications are expected.

------------------------------------------------------------------------

## Question 10 --- Rebalancing Sealed Class Usage

### Question

> A team used sealed classes everywhere and now struggles to add new
> variants quickly. How do you rebalance?

### Solution

Do not make every hierarchy sealed automatically. First identify where
the closed-world guarantee provides real value.

Use `sealed` when:

-   The domain intentionally has a fixed set of variants.
-   Exhaustive handling is important.
-   Preventing third-party or accidental implementations is desirable.

Use a normal interface or abstract class when:

-   New implementations are expected frequently.
-   Multiple teams or modules need to extend the hierarchy.
-   The type is an explicit extension point.

A practical migration approach is:

1.  Identify hierarchies that genuinely need to be closed.
2.  Remove unnecessary sealing from frequently changing extension
    points.
3.  Preserve sealed types for stable domain variants.
4.  Update exhaustive `switch` statements and tests.
5.  Document whether a hierarchy is intentionally closed or
    intentionally extensible.

**In short:** use `sealed` for controlled, stable variants; use open
interfaces/abstract classes where extensibility is a requirement.


------------------------------------------------------------------------

## Question 11 --- HashMap vs TreeMap

**Question:**\
What practical criteria guide HashMap versus TreeMap selection in
backend services?

**Solution:**\
The choice should be driven by the required operations and ordering
semantics.

-   **HashMap:** use when the primary requirement is fast key-based
    lookup/insertion and no ordering is required. Average `get`/`put`
    performance is O(1).
-   **TreeMap:** use when keys must remain sorted or when you need
    ordered operations such as `firstKey()`, `lastKey()`, `floorKey()`,
    `ceilingKey()`, or range views. These operations are generally O(log
    n).
-   `TreeMap` requires keys to have a natural ordering or a supplied
    `Comparator`.
-   `HashMap` usually has lower constant overhead for straightforward
    key lookup.
-   If concurrent access is required, consider `ConcurrentHashMap`; for
    a concurrent sorted map, consider `ConcurrentSkipListMap`.

I would choose based on **access patterns, ordering/range requirements,
memory characteristics, and concurrency needs**, rather than simply
choosing one based on Big-O.

------------------------------------------------------------------------

## Question 12 --- ArrayList vs LinkedList

**Question:**\
How do you compare ArrayList and LinkedList beyond Big-O notation?

**Solution:**\
Beyond asymptotic complexity, memory layout and CPU-cache behavior
matter significantly.

`ArrayList` stores elements in a contiguous backing array, giving
excellent locality and fast indexed access. It generally has lower
per-element memory overhead and performs well for iteration.

`LinkedList` stores each element in a separate node containing the
element plus link references. This creates allocation overhead, pointer
chasing, and poorer cache locality.

Therefore: - Use `ArrayList` for most general-purpose list workloads. -
Use `LinkedList` only when its specific linked-node behavior is actually
useful, and benchmark the workload. - If frequent insertion/removal is
required in the middle, first consider whether a different data
structure better represents the workload.

In practice, `ArrayList` is often preferable even when both structures
have similar asymptotic complexity for an operation because of **cache
locality, allocations, and constant factors**.

------------------------------------------------------------------------

## Question 13 --- Thread-Safe Singletons

**Question:**\
What are robust options to implement thread-safe singletons in Java, and
when do you choose each?

**Solution:**\
Several approaches are robust:

### 1. Enum singleton

``` java
public enum AppConfig {
    INSTANCE;
}
```

This is concise and provides strong guarantees around instance creation
and serialization.

### 2. Eager initialization

``` java
public final class Config {
    private static final Config INSTANCE = new Config();

    private Config() {}

    public static Config getInstance() {
        return INSTANCE;
    }
}
```

Simple and thread-safe because class initialization is thread-safe.

### 3. Initialization-on-demand holder

``` java
public final class Config {
    private Config() {}

    private static class Holder {
        private static final Config INSTANCE = new Config();
    }

    public static Config getInstance() {
        return Holder.INSTANCE;
    }
}
```

Provides lazy initialization without explicit synchronization.

### 4. Double-checked locking

``` java
private static volatile Config instance;

public static Config getInstance() {
    if (instance == null) {
        synchronized (Config.class) {
            if (instance == null) {
                instance = new Config();
            }
        }
    }
    return instance;
}
```

`volatile` is essential for safe publication.

I would normally favor the simplest implementation that meets the
lifecycle requirement, and avoid synchronization-heavy designs unless
lazy initialization or other constraints justify them.

------------------------------------------------------------------------

## Question 14 --- Testability with Framework Annotations

**Question:**\
How do you improve testability when framework annotations and lifecycle
callbacks are deeply mixed into business logic?

**Solution:**\
Separate framework concerns from domain/business logic.

I would: 1. Move business rules into plain Java classes with minimal
framework dependencies. 2. Keep controllers, lifecycle callbacks,
annotations, and configuration in thin adapter layers. 3. Inject
collaborators through constructors rather than constructing them
internally. 4. Make lifecycle methods delegate to ordinary application
services. 5. Test business logic with fast unit tests without starting
the framework. 6. Keep a smaller number of integration tests to verify
framework wiring and lifecycle behavior.

For example, instead of putting substantial logic in `@PostConstruct`,
have it call a service method whose behavior can be tested
independently.

This creates a clear boundary between **framework integration** and
**business behavior**, improving test speed, determinism, and
maintainability.

------------------------------------------------------------------------

## Question 15 --- List Choice as a Scalability Bottleneck

**Question:**\
What signs indicate your list choice has become a scalability
bottleneck?

**Solution:**\
I would look for measurable symptoms rather than assuming the collection
is the problem.

Warning signs include: - Increasing p95/p99 latency as collection size
grows. - CPU increasing disproportionately during searches or
removals. - Large allocation and GC overhead. - Frequent resizing or
copying of arrays. - Excessive memory usage. - Profiling showing
collection operations on a hot path. - Throughput degrading as the
dataset grows.

I would correlate these observations with the operation pattern: -
Frequent random lookup may suggest a map/set. - Queue behavior may
suggest `ArrayDeque`. - Ordered/range queries may require a sorted
structure. - Frequent middle insertion/removal may need a different
representation.

Then benchmark realistic workloads before changing the data structure.

------------------------------------------------------------------------

## Question 16 --- Mutable HashMap Keys

**Question:**\
A production bug appears because a key object in a HashMap was mutated
after insertion. How do you fix the immediate issue and prevent
recurrence?

**Solution:**\
The problem is that `HashMap` uses the key's `hashCode()` to determine
its bucket. If a field participating in `equals()`/`hashCode()` changes
after insertion, the object can remain in its old bucket while its new
hash points elsewhere.

### Immediate fix

-   Identify affected entries.
-   Remove/rebuild/reinsert them using valid immutable key state, where
    possible.
-   If necessary, reconstruct the map from authoritative data.
-   Fix the code path that mutates the key.

### Prevention

Make map keys immutable:

``` java
public final class Key {
    private final String id;

    public Key(String id) {
        this.id = id;
    }

    // equals/hashCode based on id
}
```

Avoid setters or mutable fields used by `equals()`/`hashCode()`. Also
add tests that verify key behavior and use static analysis/code review
rules where appropriate.

------------------------------------------------------------------------

## Question 17 --- ReentrantLock vs synchronized

**Question:**\
A high-throughput service uses synchronized blocks heavily and suffers
contention. You consider ReentrantLock. What decision framework do you
use?

**Solution:**\
I would not replace `synchronized` automatically. First profile to
identify where contention actually occurs.

`ReentrantLock` can be useful when I need capabilities that
`synchronized` does not provide conveniently, such as: - `tryLock()` for
bounded/non-blocking acquisition. - Timed lock acquisition. -
Interruptible lock acquisition. - Fairness configuration. - Multiple
`Condition` objects.

For straightforward mutual exclusion, `synchronized` is simpler and less
error-prone.

I would: 1. Measure lock contention and critical-section duration. 2.
Reduce the amount of work performed while holding the lock. 3. Consider
finer-grained locking or partitioning. 4. Evaluate whether a concurrent
collection or lock-free/atomic approach can remove the lock. 5. Use
`ReentrantLock` only where its semantics provide a concrete benefit. 6.
Benchmark under realistic concurrency.

The decision should be based on **contention profile, required locking
semantics, correctness, and maintainability**, not simply perceived
performance.

------------------------------------------------------------------------

## Question 18 --- Fail-Safe Iterators

**Question:**\
How can misuse of fail-safe iterators create subtle data quality issues?

**Solution:**\
Fail-safe or weakly consistent iteration can avoid
`ConcurrentModificationException`, but that does not mean the iteration
represents a stable snapshot.

For example, iterating over a concurrent collection while another thread
modifies it may result in: - Newly added elements being omitted. -
Removed elements being observed or skipped depending on timing. - A
result that represents no single consistent point in time.

This can silently produce incomplete or inconsistent calculations.

I would determine whether the operation requires: - A consistent
snapshot. - Weakly consistent traversal. - Explicit synchronization.

For snapshot semantics, copy the data or use an appropriate
snapshot/immutable structure. For concurrent processing, document the
iterator's consistency guarantees and test concurrent scenarios
explicitly.

The key lesson is that **avoiding an exception is not the same as
guaranteeing data consistency**.

------------------------------------------------------------------------

## Question 19 --- Hidden Costs of Singletons

**Question:**\
What are the hidden costs of singletons in modern applications?

**Solution:**\
Singletons can introduce several architectural costs:

-   **Global mutable state:** behavior becomes dependent on shared
    state.
-   **Hidden dependencies:** classes can access the singleton without
    declaring the dependency explicitly.
-   **Poor test isolation:** state can leak between tests.
-   **Concurrency issues:** shared mutable singleton state requires
    synchronization.
-   **Lifecycle complexity:** initialization and shutdown become harder
    to manage.
-   **Reduced flexibility:** replacing the implementation or having
    multiple instances becomes difficult.
-   **Tight coupling:** consumers depend on a concrete global instance
    rather than an abstraction.

Dependency Injection usually makes dependencies explicit and allows
test-specific implementations.

Singleton scope can still be appropriate for genuinely application-wide,
stateless or carefully managed resources, but it should be an
intentional lifecycle decision rather than a default design pattern.

------------------------------------------------------------------------

## Question 20 --- Replacing Integration Tests with Mocks

**Question:**\
A candidate proposes replacing all integration tests with mock-based
tests once DI is introduced. How do you evaluate that plan?

**Solution:**\
DI improves testability, but it does not make integration tests
unnecessary.

I would keep a testing pyramid: - **Unit tests:** extensive, fast,
isolated, and heavily mocked/stubbed where appropriate. - **Integration
tests:** verify real component boundaries, database behavior, messaging,
serialization, transactions, and configuration. - **End-to-end tests:**
cover a smaller number of critical user/business flows.

Mocks verify that our code interacts with an assumed contract.
Integration tests verify that the actual components satisfy that
contract together.

I would therefore use DI to make unit tests easier, while retaining
targeted integration tests for infrastructure and boundary behavior. The
right question is not "unit or integration?" but **which failure mode
each test layer is intended to detect**.

------------------------------------------------------------------------

## Question 21 --- Nested Service Locks and DB Transaction Lock

**Question:**\
You need to review a design that nests two service-level locks and one
DB transaction lock. What red flag do you look for?

**Solution:**\
The primary red flag is a **lock-ordering and lock-scope problem**.

If different code paths acquire the service locks in different orders,
they can deadlock. Holding application locks while waiting for a
database transaction/row lock can make this worse by creating a
cross-layer dependency.

I would inspect: - Whether locks are acquired in a globally consistent
order. - Whether locks are held across network or database calls. - The
duration of each critical section. - Transaction boundaries. - Whether
the application lock and DB lock protect overlapping resources. -
Whether concurrency can be redesigned to avoid nested locks.

Prefer short critical sections, consistent lock ordering, and avoiding
external I/O while holding application locks. Where possible, let the
database enforce data consistency using appropriate constraints and
transactions rather than duplicating locking at multiple layers.

------------------------------------------------------------------------

## Question 22 --- Concurrency Incident Under Peak Load

### Question

> A concurrency-related incident appears only under peak load and
> disappears in debug runs. What debugging approach do you take?

### Solution

This strongly suggests a **timing-sensitive concurrency bug**, such as a
race condition, visibility problem, incorrect synchronization, lock
contention, or ordering issue. Debugging can hide the problem because
breakpoints and slower execution change thread scheduling.

A systematic approach:

1.  **Reproduce under realistic load**
    -   Use stress/load tests rather than a single-threaded debugger.
    -   Increase concurrency and vary timing.
2.  **Capture evidence without changing timing too much**
    -   Thread dumps.
    -   Java Flight Recorder (JFR).
    -   JVM/application metrics.
    -   Structured logs containing thread IDs, request/event IDs and
        timestamps.
3.  **Look for shared mutable state**
    -   Non-thread-safe collections.
    -   Check-then-act sequences.
    -   Unsynchronized counters/state.
    -   Unsafe publication.
    -   Missing `volatile`, locks, or atomic operations where required.
4.  **Inspect lock behavior**
    -   Look for blocked threads and lock ownership.
    -   Check for lock-order problems and excessive contention.
5.  **Create a deterministic stress test**
    -   Run the smallest possible piece of code with many threads and
        repeated iterations.
    -   Add barriers/latches to force interesting interleavings.
6.  **Fix the synchronization model**
    -   Prefer immutable state and thread-safe data structures.
    -   Use atomic operations or correctly scoped locks.
    -   Avoid synchronization that is broader than necessary.
7.  **Regression-test under load**
    -   A successful debug run is not evidence that the race is fixed.

**Interview answer:**

> "I would avoid relying on a debugger because it changes thread timing.
> I'd reproduce the issue with a stress test, capture thread dumps/JFR
> and structured concurrency logs, identify shared mutable state and
> unsafe interleavings, then create a deterministic test that exposes
> the race. After fixing the synchronization or state-management issue,
> I'd verify it under realistic load."

------------------------------------------------------------------------

## Question 23 --- Upper and Lower Wildcard Bounds

### Question

> You need to design a reusable API for processing collections from
> different producer and consumer types. A bug appears due to incorrect
> wildcard bounds. How do you fix using upper and lower bounds?

### Solution

Use **PECS: Producer Extends, Consumer Super**.

-   `? extends T` is appropriate when the collection **produces/read
    values** of type `T`.
-   `? super T` is appropriate when the collection **consumes/accepts
    values** of type `T`.

Example:

``` java
void copy(
    List<? extends Number> source,
    List<? super Number> destination
) {
    for (Number value : source) {
        destination.add(value);
    }
}
```

The source can be:

``` java
List<Integer>
List<Double>
List<Long>
```

The destination can be:

``` java
List<Number>
List<Object>
```

### Upper bound

``` java
List<? extends Number>
```

You can safely read values as `Number`, but you generally cannot add a
specific `Number` because the actual list could be `List<Integer>`.

### Lower bound

``` java
List<? super Integer>
```

You can safely add `Integer` values, but reading gives only `Object`
safely.

**Interview answer:**

> "If the API reads from a generic collection, use `extends`. If it
> writes values into a collection, use `super`. The PECS rule prevents
> unsafe wildcard bounds while keeping the API reusable across subtype
> relationships."

------------------------------------------------------------------------

## Question 24 --- Testing Payment Gateways and Message Brokers

### Question

> Your team struggles to test code that calls payment gateways and
> message brokers directly. How would DI specifically improve testing
> strategy and reliability?

### Solution

Dependency Injection allows the application service to depend on an
**abstraction** rather than directly constructing or calling
infrastructure components.

For example:

``` java
interface PaymentGateway {
    PaymentResult charge(PaymentRequest request);
}
```

Production:

``` java
class StripePaymentGateway implements PaymentGateway {
    // real integration
}
```

Test:

``` java
class FakePaymentGateway implements PaymentGateway {
    // deterministic test behavior
}
```

Then inject the dependency:

``` java
class OrderService {
    private final PaymentGateway gateway;

    OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

This enables:

-   Fast unit tests using fakes/mocks.
-   Deterministic failure scenarios.
-   Testing timeouts, retries and declined payments.
-   Isolated message-broker tests.
-   Integration tests against real services separately.
-   Contract tests to verify assumptions about external APIs.
-   Reduced coupling between business logic and infrastructure.

A robust strategy is:

1.  Unit test business logic with test doubles.
2.  Run contract/integration tests against real gateway/broker behavior.
3.  Use tools such as Testcontainers where appropriate.
4.  Add resilience tests for timeout, retry, duplicate-message and
    failure scenarios.
5.  Keep external integration boundaries small and explicit.

**Key idea:** DI does not eliminate integration testing; it makes it
possible to **separate fast deterministic unit tests from slower real
integration tests**.

------------------------------------------------------------------------

## Question 25 --- Deadlock vs Livelock

### Question

> How do deadlock and livelock differ operationally, and why does that
> distinction matter?

### Solution

A **deadlock** occurs when threads are blocked waiting for resources
held by each other, so none can proceed.

Example:

-   Thread A holds Lock 1 and waits for Lock 2.
-   Thread B holds Lock 2 and waits for Lock 1.

Neither thread can continue.

A **livelock** occurs when threads are not blocked, but they
continuously react to one another and still make no useful progress.

For example, two threads repeatedly release and retry locks in response
to each other.

  -----------------------------------------------------------------------
  Property                Deadlock                Livelock
  ----------------------- ----------------------- -----------------------
  Thread state            Usually blocked/waiting Usually running

  CPU usage               Often low               Can be high

  Progress                None                    None

  Typical cause           Circular resource       Excessive
                          dependency              retry/coordination

  Detection               Thread dumps/lock       Repeated activity
                          graphs                  without progress
  -----------------------------------------------------------------------

Why it matters:

-   Deadlocks require breaking a resource dependency.
-   Livelocks require changing retry/backoff/coordination behavior.

Typical fixes include consistent lock ordering for deadlocks and
**bounded retries, randomized/exponential backoff, or coordination
changes** for livelocks.

------------------------------------------------------------------------

## Question 26 --- Mutable HashSet Key

### Question

> Consider this scenario. You have a Product class where both equals()
> and hashCode() are based on the productName field. You create a
> Product with name "Widget", add it to a HashSet, and then change the
> productName to "Gadget" using a setter. Now when you call contains()
> with the original reference --- the exact same object sitting in the
> set --- it returns false. You also call remove() with that same
> reference and it returns false too. The object is stuck in the set ---
> you cannot find it and you cannot remove it. What happened, and how do
> you prevent this?

### Solution

The problem is that the object was used as a `HashSet` key while a field
participating in `equals()`/`hashCode()` was mutable.

Initially:

``` text
productName = "Widget"
hashCode() = hash("Widget")
```

When the object is inserted, `HashSet` uses that hash to determine the
bucket.

After:

``` java
product.setProductName("Gadget");
```

the object's hash code changes:

``` text
hashCode() = hash("Gadget")
```

But the object is still physically stored in the bucket selected using
the old `"Widget"` hash.

When `contains()` or `remove()` is called, `HashSet` calculates the
**new** hash and looks in the bucket for `"Gadget"`. The object is still
in the old bucket, so it cannot be found.

This can happen even with the **exact same object reference**.

### Prevention

**Best solution: make hash-relevant state immutable.**

``` java
public final class Product {
    private final String productName;

    public Product(String productName) {
        this.productName = productName;
    }

    public String getProductName() {
        return productName;
    }

    @Override
    public boolean equals(Object o) {
        // equality based on immutable state
    }

    @Override
    public int hashCode() {
        return Objects.hash(productName);
    }
}
```

If mutation is unavoidable:

1.  Remove the object from the set.
2.  Change the field.
3.  Add it back.

``` java
set.remove(product);
product.setProductName("Gadget");
set.add(product);
```

However, this only works if you can reliably remove it before mutation.

**Interview takeaway:**

> Never mutate fields that participate in `equals()` or `hashCode()`
> while an object is being used as a key in a hash-based collection.

------------------------------------------------------------------------

## Question 27 --- Dependency Injection and Testability

### Question

> Your team is tasked with ensuring that a Java application can be
> easily tested across different environments. How can Dependency
> Injection facilitate this requirement?

### Solution

Dependency Injection separates an object's business logic from the
concrete dependencies it uses.

Instead of:

``` java
class OrderService {
    private final PaymentClient client = new PaymentClient();
}
```

use:

``` java
class OrderService {
    private final PaymentClient client;

    OrderService(PaymentClient client) {
        this.client = client;
    }
}
```

Even better, depend on an abstraction:

``` java
interface PaymentClient {
    PaymentResult pay(Order order);
}
```

Production can inject:

``` java
new RealPaymentClient(...)
```

Tests can inject:

``` java
new FakePaymentClient(...)
```

or a mock.

This allows:

-   Unit tests without external systems.
-   Environment-specific implementations.
-   Easy substitution of database, HTTP, messaging and filesystem
    dependencies.
-   Deterministic failure simulation.
-   Reduced coupling.
-   Easier configuration management.

A good design keeps the application core independent of infrastructure
and injects infrastructure at the boundary.

**Interview answer:**

> "DI makes dependencies explicit and replaceable. I can inject real
> implementations in production and mocks, fakes or stubs in tests,
> allowing the same business logic to be tested across environments
> without changing the code under test."

------------------------------------------------------------------------

## Question 28 --- `Map.of()` and Mutable Maps

### Question

> A developer writes a configuration loader that creates a map using
> Map.of("retry", 3, "timeout", 5000). Later in the application, a
> module tries to add a new entry --- map.put("maxConnections", 10) ---
> and the application crashes in production with an
> UnsupportedOperationException. The developer is confused because "it's
> just a regular Map." Walk me through exactly why this happens, what
> Map.of() actually returns, how it differs from new HashMap(), and what
> the complete landscape of immutable, unmodifiable, and mutable maps
> looks like in Java.

### Solution

`Map.of(...)` creates an **unmodifiable map**. It implements `Map`, but
that does not mean every `Map` implementation supports mutation.

Therefore:

``` java
Map<String, Integer> config =
    Map.of("retry", 3, "timeout", 5000);

config.put("maxConnections", 10);
```

throws:

``` text
UnsupportedOperationException
```

### `Map.of()` vs `new HashMap<>()`

`Map.of()`:

``` java
Map<String, Integer> map =
    Map.of("retry", 3, "timeout", 5000);
```

-   Unmodifiable.
-   `put`, `remove`, `clear`, etc. are unsupported.
-   Does not permit null keys or null values.
-   Useful for fixed configuration.
-   Designed for convenient creation of small immutable-style maps.

`HashMap`:

``` java
Map<String, Integer> map = new HashMap<>();
map.put("retry", 3);
map.put("timeout", 5000);
map.put("maxConnections", 10);
```

-   Mutable.
-   Allows normal additions/removals.
-   Permits one null key and null values.

### `Collections.unmodifiableMap()`

``` java
Map<String, Integer> original = new HashMap<>();
Map<String, Integer> view =
    Collections.unmodifiableMap(original);
```

The returned map is an **unmodifiable view**. You cannot modify it
through `view`, but changes to `original` are visible through `view`.

That differs from an independently constructed immutable map.

### `Map.copyOf()`

``` java
Map<String, Integer> copy = Map.copyOf(original);
```

Creates an unmodifiable map based on the source contents. It is useful
when exposing a map without allowing callers to mutate it.

### Landscape

  ------------------------------------------------------------------------------------
  Map type                                              Mutable? Typical use
  --------------------------------- ---------------------------- ---------------------
  `HashMap`                                                  Yes General-purpose
                                                                 mutable map

  `LinkedHashMap`                                            Yes Mutable map with
                                                                 predictable iteration
                                                                 order

  `TreeMap`                                                  Yes Sorted keys

  `ConcurrentHashMap`                                        Yes Concurrent access

  `Map.of()`                                                  No Small fixed maps

  `Map.ofEntries()`                                           No Fixed maps with many
                                                                 entries

  `Map.copyOf()`                                              No Unmodifiable copy

  `Collections.unmodifiableMap()`            No through the view Read-only view over
                                                                 another map
  ------------------------------------------------------------------------------------

A useful design rule is:

> If configuration should never change, expose an unmodifiable/immutable
> representation. If application logic must add or remove entries,
> create a mutable map such as `HashMap`.

------------------------------------------------------------------------

## Question 29 --- HashMap Collision and High Load

### Question

> You are developing a high-performance application that heavily relies
> on Java's HashMap for storing and retrieving data. During a code
> review, a colleague points out that multiple keys in your HashMap are
> hashing to the same bucket. What steps would you take to address
> potential performance issues, and how would you ensure the application
> remains efficient under high load?

### Solution

First, I would distinguish between **normal collisions** and
pathological hash distribution.

Hash collisions are expected in a hash table. The important question is
whether the distribution is poor enough to affect performance.

### 1. Inspect the key's `hashCode()`

Ensure the implementation:

-   Uses fields that distinguish keys well.
-   Produces a good distribution.
-   Is consistent with `equals()`.

If custom keys have a poor `hashCode()`, many entries can end up in the
same bucket.

### 2. Understand Java's collision handling

Modern Java `HashMap` can convert sufficiently large collision chains
into tree bins, improving worst-case lookup behavior compared with a
long linked list.

However, this does not mean poor hashing is free. Excessive collisions
still increase work and can indicate a bad key design.

### 3. Size the map appropriately

If the approximate number of entries is known, initialize the map with
an appropriate capacity to reduce repeated resizing.

``` java
Map<Key, Value> map = new HashMap<>(expectedCapacity);
```

Account for the load factor when choosing capacity so the map does not
resize immediately.

### 4. Profile rather than guess

Under realistic load, measure:

-   `get()`/`put()` latency.
-   CPU usage.
-   Allocation rate.
-   GC behavior.
-   Throughput.
-   Map size.
-   Application latency under concurrency.

Use JFR/profilers and benchmarks rather than assuming collisions are the
bottleneck.

### 5. Check key mutability

Never mutate fields used by `equals()`/`hashCode()` while a key is
stored in the map. That can make entries effectively unreachable.

### 6. Consider concurrency separately

`HashMap` is not safe for concurrent mutation.

If multiple threads need concurrent access, consider:

``` java
ConcurrentHashMap<Key, Value>
```

rather than adding external synchronization without understanding the
workload.

### 7. Test under production-like load

Use realistic key distributions and traffic patterns. A map can look
fine with a small synthetic dataset and behave differently at production
scale.

**Interview answer:**

> "I would first verify the key's `hashCode()` distribution and the
> `equals()` contract, because collisions are normal but pathological
> clustering is not. I'd size the map appropriately, profile under
> realistic load, check for mutable keys, and use `ConcurrentHashMap`
> when concurrent access is required. I would validate the change with
> throughput and latency measurements rather than optimizing based only
> on the fact that collisions exist."

------------------------------------------------------------------------

## Question 30 --- HashSet Deduplication Failure

### Question

> Here is a real production debugging scenario. Your team has a service
> that deduplicates incoming events using a HashSet of EventKey objects.
> EventKey has three fields: sourceSystem (String), eventId (String),
> and timestamp (Instant). In production, the deduplication is failing
> --- duplicate events are getting through even though the logs show
> EventKey objects with identical field values. You inspect the equals()
> and hashCode() implementations and they look correct --- using all
> three fields. You add logging and confirm that equals() returns true
> when comparing the duplicate keys, and hashCode() returns the same
> value. Yet the HashSet contains() returns false. What could cause
> this, and how do you systematically debug it?

### Solution

If `equals()` is true and `hashCode()` is equal **at the moment of the
comparison**, a normal, correctly functioning `HashSet` should be able
to find an unchanged matching key.

Therefore, I would investigate **state changes, collection identity,
concurrency, and runtime/class-version issues** rather than immediately
rewriting `equals()`.

### 1. Check whether the key was mutated after insertion

This is the first thing to investigate.

If `EventKey` is mutable and one of these changes after insertion:

-   `sourceSystem`
-   `eventId`
-   `timestamp`

then its hash code can change.

The object remains in the bucket selected using its old hash, while
`contains()` searches using the new hash.

This produces the classic "object is in the set but cannot be found"
problem.

**Fix:** make `EventKey` immutable.

``` java
public record EventKey(
    String sourceSystem,
    String eventId,
    Instant timestamp
) {}
```

`String` and `Instant` are immutable, and a record gives value-based
`equals()`/`hashCode()` for its components.

### 2. Log hash codes at insertion and lookup

Do not only log the current values.

Capture:

``` text
EventKey
field values
identityHashCode(object)
hashCode()
timestamp
thread name
set identity
```

Compare the key's state:

``` text
before add
after add
before contains
```

If the hash code changed between insertion and lookup, you have found a
mutation/state problem.

### 3. Verify that it is actually the same `HashSet`

In real applications, the apparent problem can come from using different
collection instances.

Log:

``` java
System.identityHashCode(set)
```

for both insertion and lookup.

Check for:

-   Different service instances.
-   Request-local vs shared state.
-   Reinitialization.
-   Cache replacement.
-   Incorrect dependency scope.
-   Multiple application contexts.

### 4. Check concurrent access

`HashSet` is not thread-safe.

If multiple threads mutate or access it concurrently, the application
has a concurrency bug. Even if logging shows logically equal keys, the
observed collection state may not be reliable.

For concurrent use, consider:

``` java
ConcurrentHashMap<EventKey, Boolean>
```

or a properly synchronized/concurrent design.

For atomic deduplication, operations should be designed so the **check
and insertion are one logical operation**, rather than:

``` java
if (!set.contains(key)) {
    set.add(key);
}
```

across multiple threads.

### 5. Verify the actual runtime class

Check that the objects are really the expected `EventKey` class:

``` java
System.out.println(key.getClass());
System.out.println(key.getClass().getClassLoader());
```

This can help expose unusual class-loader/version problems in containers
or modular applications.

### 6. Check the deployed bytecode/version

Production can sometimes run a different build from the source being
inspected.

Verify:

-   Artifact version.
-   Container image.
-   Deployment revision.
-   Class checksum/build metadata.
-   Whether all nodes run the same version.

### 7. Inspect null and normalization rules

Logs can make values look identical while underlying representations
differ.

For example:

-   Whitespace.
-   Case normalization.
-   Different Unicode representations.
-   Timestamp precision.
-   Time-zone conversion before creating the `Instant`.

Since `Instant` represents an absolute point in time, verify the exact
stored value rather than relying on formatted logs.

### 8. Build a minimal reproduction

Create a small test:

``` java
EventKey a = new EventKey("PAYMENTS", "123", timestamp);
EventKey b = new EventKey("PAYMENTS", "123", timestamp);

Set<EventKey> set = new HashSet<>();
set.add(a);

assert a.equals(b);
assert a.hashCode() == b.hashCode();
assert set.contains(b);
```

If this passes, the Java `HashSet` implementation is unlikely to be the
problem. Focus on production state, mutation, concurrency, collection
identity, or deployment differences.

### Systematic debugging checklist

1.  Verify the actual `HashSet` instance.
2.  Log insertion and lookup using the same key fields.
3.  Compare hash codes **at insertion and lookup**, not just once.
4.  Check whether `EventKey` is mutable.
5.  Check for concurrent access to the `HashSet`.
6.  Verify runtime class and deployed version.
7.  Check timestamp/value normalization.
8.  Reproduce with a minimal test.
9.  Replace mutable keys with immutable value objects/records.
10. For concurrent deduplication, make the membership/update operation
    atomic.

**Interview answer:**

> "If `equals()` and `hashCode()` are genuinely correct and stable,
> `HashSet.contains()` should work. I would therefore investigate
> whether the key was mutated after insertion, whether the lookup is
> using the same set instance, whether the set is being accessed
> concurrently, and whether production is running the expected class
> version. I'd log the key and hash at both insertion and lookup,
> reproduce with a minimal test, and make `EventKey` immutable---ideally
> a record---once the root cause is confirmed."

------------------------------------------------------------------------

# Quick Revision Summary

  ----------------------------------------------------------------------------
                            \# Topic                  Core idea
  ---------------------------- ---------------------- ------------------------
                             1 `ThreadPoolExecutor`   Worker pool + task
                                                      queue + bounded
                                                      concurrency

                             2 Deadlock prevention    Lock ordering, short
                                                      critical sections,
                                                      `tryLock`, fewer locks

                             3 `equals`/`hashCode`    Entity identity vs
                                                      value-object state

                             4 List strategy          Choose based on
                                                      access/removal workload

                             5 `HashMap` internals    Hash → bucket →
                                                      collision handling →
                                                      resize

                             6 `synchronized` vs      Simplicity vs advanced
                               `ReentrantLock`        locking control

                             7 Sealed classes         Controlled inheritance
                                                      and exhaustive handling

                             8 String immutability    Enables safe sharing;
                                                      investigate duplicate
                                                      allocations

                             9 Iterators              Fail-fast vs
                                                      snapshot/weakly
                                                      consistent behavior

                            10 Sealed class rebalance Closed hierarchy vs open
                                                      extension point

                        11--21 ---                    Original images needed

                            22 Concurrency debugging  Stress test, JFR/thread
                                                      dumps, deterministic
                                                      reproduction

                            23 Wildcards              PECS: Producer Extends,
                                                      Consumer Super

                            24 DI testing             Replace external
                                                      dependencies with test
                                                      doubles

                            25 Deadlock vs livelock   Blocked vs
                                                      active-but-no-progress

                            26 Mutable `HashSet` key  Hash changes can make an
                                                      entry unreachable

                            27 DI across environments Inject abstractions and
                                                      swap implementations

                            28 `Map.of`               Unmodifiable map; use
                                                      `HashMap` when mutation
                                                      is required

                            29 HashMap collisions     Good hashing, sizing,
                                                      profiling, concurrency
                                                      considerations

                            30 HashSet deduplication  Check mutation,
                                                      collection identity,
                                                      concurrency, runtime
                                                      version
  ----------------------------------------------------------------------------

------------------------------------------------------------------------

## Important Java Contracts to Remember

### `equals()` / `hashCode()`

``` text
a.equals(b) == true
        ↓
a.hashCode() == b.hashCode()
```

The reverse is **not** guaranteed:

``` text
same hashCode ≠ objects are equal
```

### Hash-based collection rule

Do not mutate fields used by `equals()`/`hashCode()` while an object is
stored in:

-   `HashMap`
-   `HashSet`
-   `LinkedHashMap`
-   `LinkedHashSet`

### Wildcard rule

``` text
Producer → extends
Consumer → super
```

### Concurrency rule

Prefer:

``` text
immutable state
    ↓
thread-safe data structures
    ↓
small, well-defined synchronization
```

over unnecessary shared mutable state and large synchronized regions.
