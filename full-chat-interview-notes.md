# Full Chat — Interview Preparation Notes

## Java 8 — Prime Numbers

### First 5 prime numbers

```java
IntStream.iterate(2, n -> n + 1)
    .filter(Main::isPrime)
    .limit(5)
    .forEach(System.out::println);
```

Output: `2 3 5 7 11`

Prime check:

```java
static boolean isPrime(int n) {
    return n > 1 &&
        IntStream.rangeClosed(2, (int) Math.sqrt(n))
            .noneMatch(i -> n % i == 0);
}
```

### Why `sqrt(n)`?

Factors come in pairs. If `n = a × b`, at least one factor must be `<= sqrt(n)`.

Example:

```text
36 = 2 × 18
   = 3 × 12
   = 4 × 9
   = 6 × 6
```

Therefore, checking divisors beyond `sqrt(n)` is unnecessary.

### `range()` vs `rangeClosed()`

```text
range(1, 5)       → 1 2 3 4
rangeClosed(1, 5) → 1 2 3 4 5
```

`rangeClosed()` includes the end value.

### `noneMatch()`

```java
.noneMatch(i -> n % i == 0)
```

Returns `true` when no stream element satisfies the condition. For prime checking, it means no number from `2` through `sqrt(n)` divides `n`.

---

## Print One Prime and Skip One Prime

Desired pattern:

```text
2 → print
3 → skip
5 → print
7 → skip
11 → print
```

Using a mutable counter:

```java
AtomicInteger index = new AtomicInteger();

.filter(n -> index.getAndIncrement() % 2 == 0)
```

A normal local variable cannot be modified inside a lambda because captured local variables must be final or effectively final.

`AtomicInteger.getAndIncrement()` returns the current value and then increments it.

---

# React — DOM vs Virtual DOM

### DOM

**DOM = Document Object Model.**

It represents the actual document structure maintained by the browser.

### Virtual DOM

The Virtual DOM is React's JavaScript representation of the UI used to determine what needs to change.

### When state updates

```text
State update
    ↓
Component re-render
    ↓
New UI representation
    ↓
Compare with previous representation
    ↓
Determine changes
    ↓
Update required DOM
```

This comparison/update process is part of React reconciliation.

---

# AWS S3 — Preventing Deletion

### S3 Versioning

Versioning keeps previous object versions available. A delete normally creates a delete marker rather than immediately removing the previous version.

### MFA Delete

Provides an additional protection layer for certain destructive operations involving object versions.

### S3 Object Lock

Can prevent objects from being deleted or overwritten during a retention period.

Modes/features include:

- Governance mode
- Compliance mode
- Legal holds

### IAM / Bucket Policy

Deletion can be restricted with permissions such as:

```json
{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

**Interview answer:** Enable Versioning for recovery from accidental deletion; use Object Lock, MFA Delete where appropriate, and IAM/bucket policies for stronger deletion controls.

---

# Kafka vs Amazon SQS

| Kafka | SQS |
|---|---|
| Event streaming platform | Message queue |
| Topics and partitions | Queues |
| Events retained according to topic retention | Messages normally removed after successful processing |
| Consumers can re-read/replay retained events | Primarily task/message processing |
| Multiple independent consumer groups | Consumers generally compete for queue messages |
| Ordering within a partition | FIFO queues provide ordering |
| Good for event streaming, CDC, analytics | Good for background jobs and service decoupling |

### Simple distinction

```text
SQS   → "Give this task to a worker."
Kafka → "Publish this event for independent consumers."
```

Example Kafka architecture:

```text
                 → Payment
Producer → Kafka → Inventory
                 → Notification
                 → Analytics
```

---

# Java — Immutable Class

An immutable object cannot have its state changed after construction.

### Rules

1. Make the class `final`.
2. Make fields `private final`.
3. Initialize fields through the constructor.
4. Don't provide setters.
5. Make defensive copies of mutable objects.
6. Don't expose mutable internal objects directly.

Example:

```java
public final class Employee {

    private final int id;
    private final String name;
    private final List<String> skills;

    public Employee(int id, String name, List<String> skills) {
        this.id = id;
        this.name = name;
        this.skills = new ArrayList<>(skills);
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public List<String> getSkills() {
        return Collections.unmodifiableList(skills);
    }
}
```

**Important:** `final` alone does not make an object immutable. A `final List` reference cannot change, but the list contents can still change unless protected.

**Interview answer:** Make the class final, fields private/final, initialize them in the constructor, provide no setters, and use defensive copies for mutable fields.

---

# Java — Callable

`Callable` is used for tasks that:

- Return a result.
- Can throw checked exceptions.

It is in `java.util.concurrent`.

### Runnable vs Callable

| Runnable | Callable |
|---|---|
| `run()` | `call()` |
| Returns `void` | Returns a value |
| Cannot directly throw checked exceptions | Can throw checked exceptions |

Example:

```java
ExecutorService executor =
    Executors.newSingleThreadExecutor();

Callable<Integer> task = () -> 10 + 20;

Future<Integer> future = executor.submit(task);

System.out.println(future.get());

executor.shutdown();
```

Output:

```text
30
```

Flow:

```text
Callable
   ↓
ExecutorService.submit()
   ↓
Future
   ↓
future.get()
   ↓
Result
```

**Interview answer:** Callable is similar to Runnable but can return a value and throw checked exceptions. It is submitted to an ExecutorService and the result is obtained through a Future.

---

# Java 8 — Asynchronous API Calls

A common Java 8 approach is `CompletableFuture`.

### Basic example

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> {
        // API call
        return "API Response";
    });

future.thenAccept(response -> {
    System.out.println("Response: " + response);
});

System.out.println("Main thread continues...");
```

### Error handling

```java
future
    .thenAccept(response -> {
        System.out.println(response);
    })
    .exceptionally(error -> {
        System.out.println("API failed");
        return null;
    });
```

### Multiple API calls in parallel

```java
CompletableFuture<String> user =
    CompletableFuture.supplyAsync(() -> getUser());

CompletableFuture<String> orders =
    CompletableFuture.supplyAsync(() -> getOrders());

CompletableFuture.allOf(user, orders)
    .thenRun(() -> {
        System.out.println(user.join());
        System.out.println(orders.join());
    });
```

### Useful methods

- `supplyAsync()` → async task returning a value
- `runAsync()` → async task with no return value
- `thenApply()` → transform a result
- `thenCompose()` → chain dependent async operations
- `thenAccept()` → consume a result
- `allOf()` → wait for multiple futures
- `exceptionally()` → handle errors
- `handle()` → handle success and failure

**Interview answer:** In Java 8, use `CompletableFuture.supplyAsync()` for asynchronous work, `thenApply`/`thenCompose` for chaining, `allOf` for multiple calls, and `exceptionally` or `handle` for error handling.

---

# Quick Interview Revision

```text
JAVA
----
range()               → end excluded
rangeClosed()         → end included
noneMatch()            → true if nothing matches
sqrt(n)               → sufficient limit for prime checking
AtomicInteger          → mutable counter usable inside lambda
Callable               → returns result + checked exceptions
Future                 → receives Callable result
CompletableFuture      → asynchronous programming

IMMUTABLE CLASS
---------------
final class
private final fields
constructor initialization
no setters
defensive copies
don't expose mutable internal state

REACT
-----
DOM                    → actual browser document
Virtual DOM            → React's UI representation
State update           → re-render → reconciliation → required DOM updates

AWS S3
------
Versioning             → recovery from deleted/overwritten objects
MFA Delete             → additional protection for destructive operations
Object Lock            → retention/deletion protection
IAM/Bucket Policy      → restrict DeleteObject

KAFKA vs SQS
------------
SQS                    → queue / task processing
Kafka                  → event streaming
SQS                    → messages normally removed after processing
Kafka                  → events retained and can be replayed
Kafka                  → multiple independent consumer groups
SQS                    → consumers compete for queue messages
