# Java, Spring Boot, Microservices & Kafka Interview Notes

## 1. Java 9 Features

Java 9 introduced the Java Platform Module System (JPMS).

### Important Features
- Module System (JPMS)
- JShell
- `List.of()`, `Set.of()`, `Map.of()`
- Private methods in interfaces
- Stream API: `takeWhile()`, `dropWhile()`
- Optional improvements
- CompletableFuture improvements
- Try-with-resources improvements

### Module Example
```java
module user.service {
    exports com.example.user;
    requires java.sql;
}
```

### Collection Factory Methods
```java
List<String> names = List.of("John", "Alice");
Set<String> users = Set.of("A", "B");
Map<Integer, String> map = Map.of(1, "John", 2, "Alice");
```

These collections are unmodifiable.

### Interview Answer
> Java 9's most significant feature was the Java Platform Module System. It also introduced JShell, collection factory methods, private interface methods, Stream improvements such as `takeWhile()`/`dropWhile()`, and Optional/CompletableFuture enhancements.

---

## 2. Java 17 Features

Java 17 is an LTS release and is widely used with Spring Boot 3.

### Important Features
- Records
- Sealed Classes
- Pattern Matching for `instanceof`
- Text Blocks
- Switch Expressions
- Helpful NullPointerExceptions
- Strong encapsulation
- Enhanced random number generators

### Record
```java
public record UserResponse(
    Long id,
    String name,
    String email
) {}
```

### Sealed Classes
```java
public sealed class Payment
        permits CreditCardPayment, UpiPayment {
}

public final class CreditCardPayment extends Payment {
}

public final class UpiPayment extends Payment {
}
```

### Pattern Matching
```java
if (obj instanceof String str) {
    System.out.println(str.length());
}
```

### Text Blocks
```java
String json = """
        {
          "name": "John",
          "age": 30
        }
        """;
```

### Interview Answer
> Java 17 is an LTS release that introduced Records, Sealed Classes, Pattern Matching for `instanceof`, Text Blocks, and modern Switch Expressions. It is also the baseline Java version for Spring Boot 3.

---

## 3. Java 21 Features

Java 21 is an LTS release.

### Important Features
- Virtual Threads
- Sequenced Collections
- Record Patterns
- Pattern Matching for `switch`
- String Templates (preview)
- Scoped Values (preview)
- Structured Concurrency (preview)
- Unnamed Classes and Instance Main Methods (preview)

### Virtual Threads
```java
Thread.startVirtualThread(() -> {
    System.out.println("Processing request");
});
```

Or:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> callUserService());
    executor.submit(() -> callOrderService());
}
```

Spring Boot can enable virtual threads with:

```properties
spring.threads.virtual.enabled=true
```

### Interview Answer
> Java 21 is an LTS release. Its major features include Virtual Threads, Pattern Matching for switch, Record Patterns, and Sequenced Collections. Virtual Threads are especially useful for high-concurrency, I/O-heavy backend and microservices applications.

### Java Version Memory Trick
```text
Java 8  -> Lambda, Stream, Optional
Java 9  -> Module System, List.of()
Java 11 -> HTTP Client, String improvements
Java 17 -> Records, Sealed Classes
Java 21 -> Virtual Threads, Pattern Matching
```

---

# 4. Java Multithreading

Multithreading means executing multiple threads concurrently within an application.

### Runnable
```java
Runnable task = () -> {
    System.out.println("Task running");
};

Thread thread = new Thread(task);
thread.start();
```

### ExecutorService
```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

executor.submit(() -> {
    System.out.println("Processing task");
});

executor.shutdown();
```

### Important Interview Topics
- Thread vs Process
- Runnable vs Callable
- `start()` vs `run()`
- `synchronized`
- Race condition
- Deadlock
- `volatile`
- AtomicInteger
- ExecutorService
- CompletableFuture
- Future
- Thread pools
- Virtual Threads
- Spring `@Async`
- ConcurrentHashMap

---

# 5. Shared Counter / Race Condition

Suppose two threads each increment a shared counter 1000 times:

```java
class Counter {
    int count = 0;

    public void increment() {
        count++;
    }
}
```

Expected result:

```text
2000
```

Actual result can be less than 2000 because `count++` is not atomic.

It is effectively:

```text
READ count
ADD 1
WRITE count
```

Two threads can read the same value and overwrite each other's updates.

This is called a **race condition**.

### Solution: synchronized
```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }
}
```

### Synchronized Block
```java
public void increment() {
    synchronized (this) {
        count++;
    }
}
```

### AtomicInteger
```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

### ReentrantLock
```java
private final ReentrantLock lock = new ReentrantLock();

public void increment() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock();
    }
}
```

### LongAdder
Useful for highly concurrent counters/metrics:

```java
private final LongAdder count = new LongAdder();

count.increment();
long result = count.sum();
```

---

# 6. volatile

`volatile` guarantees **visibility**, but not **atomicity**.

### Good use case
```java
private volatile boolean running = true;
```

Thread 1:

```java
while (running) {
    // work
}
```

Thread 2:

```java
running = false;
```

Thread 1 is guaranteed to see the updated value.

### Important
This is NOT thread-safe:

```java
volatile int count;

count++;
```

Because `count++` is a compound read-modify-write operation.

### Interview Answer
> `volatile` ensures that changes made by one thread to a variable are visible to other threads. It does not make compound operations such as `count++` atomic.

---

# 7. Combining Two Employee Lists Using Stream API

Employee:

```java
class Employee {
    private String name;
    private double salary;
    private String dept;

    public Employee(String name, double salary, String dept) {
        this.name = name;
        this.salary = salary;
        this.dept = dept;
    }

    public String getName() {
        return name;
    }

    public double getSalary() {
        return salary;
    }

    public String getDept() {
        return dept;
    }
}
```

Combine two lists and sort by name, then salary:

```java
List<Employee> result =
    Stream.concat(empList1.stream(), empList2.stream())
        .sorted(
            Comparator.comparing(Employee::getName)
                      .thenComparing(Employee::getSalary)
        )
        .toList();
```

This means:
1. Combine both lists.
2. Sort by name ascending.
3. If names are equal, sort by salary ascending.

### Name ascending + salary descending
```java
List<Employee> result =
    Stream.concat(empList1.stream(), empList2.stream())
        .sorted(
            Comparator.comparing(Employee::getName)
                .thenComparing(
                    Employee::getSalary,
                    Comparator.reverseOrder()
                )
        )
        .toList();
```

---

# 8. Async API Calls in Spring Boot

### Enable Async
```java
@SpringBootApplication
@EnableAsync
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Feign Client
```java
@FeignClient(name = "user-service", url = "${user.service.url}")
public interface UserClient {

    @GetMapping("/users/{id}")
    UserResponse getUser(@PathVariable Long id);
}
```

### Async Service
```java
@Service
public class UserService {

    private final UserClient userClient;

    public UserService(UserClient userClient) {
        this.userClient = userClient;
    }

    @Async
    public CompletableFuture<UserResponse> getUser(Long id) {
        UserResponse response = userClient.getUser(id);
        return CompletableFuture.completedFuture(response);
    }
}
```

### Important Point

`@Async` does not make the Feign HTTP call itself non-blocking. It moves the blocking Feign call to another thread.

For true non-blocking I/O, use Spring WebClient:

```java
public Mono<UserResponse> getUser(Long id) {
    return webClient
        .get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(UserResponse.class);
}
```

---

# 9. Two Databases in One Spring Boot Application

A single Spring Boot application can connect to two databases.

Example:

```text
Spring Boot
  |
  +-- DataSource 1 -> MySQL
  |
  +-- DataSource 2 -> PostgreSQL
```

### Configuration
```yaml
spring:
  datasource:
    db1:
      url: jdbc:mysql://localhost:3306/employee_db
      username: root
      password: password

    db2:
      url: jdbc:postgresql://localhost:5432/order_db
      username: postgres
      password: password
```

Typically configure:
- Two DataSource beans
- Two EntityManagerFactory beans
- Two TransactionManager beans
- Separate repository packages

Example:

```java
@EnableJpaRepositories(
    basePackages = "com.example.employee.repository",
    entityManagerFactoryRef = "employeeEntityManagerFactory",
    transactionManagerRef = "employeeTransactionManager"
)
```

And for the second DB:

```java
@EnableJpaRepositories(
    basePackages = "com.example.order.repository",
    entityManagerFactoryRef = "orderEntityManagerFactory",
    transactionManagerRef = "orderTransactionManager"
)
```

### Transaction Across Both Databases

If both DB operations must commit/rollback together, normal `@Transactional` on one database is not enough. This becomes a distributed transaction problem, typically involving XA/JTA or a design such as Saga.

---

# 10. Saga Design Pattern

Saga manages distributed transactions across microservices without one global ACID transaction.

Example:

```text
Order Service
     |
     v
Payment Service
     |
     v
Inventory Service
     |
     v
Shipping Service
```

Each service performs a local transaction.

If a later operation fails, previous successful operations are compensated.

Example:

```text
Create Order       SUCCESS
Process Payment    SUCCESS
Reserve Inventory  FAILED
        |
        v
Refund Payment
        |
        v
Cancel Order
```

## Saga Types

### Choreography

Services communicate using events.

```text
Order Service
     |
 OrderCreated
     v
 Kafka
     |
     v
Payment Service
     |
 PaymentCompleted
     v
 Kafka
     |
     v
Inventory Service
```

Advantages:
- Loose coupling
- No central coordinator
- Good scalability

Disadvantages:
- Workflow can become difficult to understand
- Debugging can be harder

### Orchestration

A central Saga Orchestrator controls the workflow.

```text
              Saga Orchestrator
              /       |       \
             v        v        v
          Order    Payment  Inventory
          Service   Service   Service
```

If inventory fails:

```text
Inventory FAILED
       |
       v
Orchestrator
       |
       +--> Refund Payment
       |
       +--> Cancel Order
```

Advantages:
- Centralized workflow
- Easier compensation and error handling
- Easier to understand business flow

Disadvantages:
- Orchestrator becomes an important component
- Adds workflow coupling/infrastructure

### Important Saga Concepts

Production implementations should consider:
- Eventual consistency
- Idempotency
- Retries
- Timeouts
- Compensation
- Durable Saga state
- Dead Letter Topics
- Transactional Outbox

### Interview Answer

> Saga is a distributed transaction pattern for microservices. Each service executes a local transaction, and if a later step fails, compensating transactions are executed for previously completed steps. It can be implemented using choreography or orchestration.

---

# 11. Kafka Architecture for Fault Tolerance

A fault-tolerant Kafka setup uses:
- Multiple brokers
- Partition replication
- Producer acknowledgements
- Consumer groups
- Offset management
- Retries
- Dead Letter Topics
- Idempotent producers
- Monitoring

### Architecture

```text
                    Spring Boot Producer
                            |
                            v
                 +-----------------------+
                 |     Kafka Cluster     |
                 |                       |
                 | Topic: order-events   |
                 |                       |
                 | P0: B1 Leader         |
                 |     B2 Replica        |
                 |     B3 Replica        |
                 |                       |
                 | P1: B2 Leader         |
                 |     B3 Replica        |
                 |     B1 Replica        |
                 +-----------+-----------+
                             |
                       Consumer Group
                             |
            +----------------+----------------+
            v                v                v
      Order Consumer   Payment Consumer  Inventory Consumer
```

## Replication

For:

```text
replication.factor = 3
```

A partition can have:

```text
Broker 1 -> Leader
Broker 2 -> Replica
Broker 3 -> Replica
```

If Broker 1 fails, another in-sync replica can become leader.

---

## Producer Durability

For critical events:

```yaml
spring:
  kafka:
    producer:
      acks: all
      properties:
        enable.idempotence: true
```

A useful durability setup:

```text
replication.factor = 3
min.insync.replicas = 2
acks = all
```

If one broker fails, two in-sync replicas can still satisfy the write requirement.

---

## Consumer Groups

```text
              order-events
                   |
          +--------+--------+
          |                 |
       Payment           Inventory
       Group              Group
          |                 |
    Payment-1          Inventory-1
    Payment-2          Inventory-2
```

Within a consumer group, a partition is assigned to one consumer at a time.

If a consumer fails, Kafka rebalances partitions to another consumer in the same group.

---

## Offset Management

Kafka consumers track offsets.

For critical processing, avoid blindly committing the offset before the business operation has succeeded.

Example:

```yaml
spring:
  kafka:
    consumer:
      enable-auto-commit: false
      group-id: payment-service
```

---

# 12. Kafka + Saga + Fault Tolerance

A common production architecture:

```text
React
  |
  v
API Gateway
  |
  v
Order Service
  |
  v
Saga Orchestrator
  |
  +----> Payment Service
  |
  +----> Inventory Service
  |
  +----> Shipping Service
              |
              v
            Kafka
              |
       Events / Retry / DLT
```

For critical business events, combine Kafka with the **Transactional Outbox Pattern**.

### Why Outbox?

Without an outbox:

```text
Database update -> SUCCESS
Kafka publish   -> FAILURE
```

The database and Kafka can become inconsistent.

With an outbox:

```text
Spring Boot Service
       |
       +--> Business DB
       |
       +--> Outbox Table
                |
                v
              Kafka
```

The business change and outbox event are written in the same local database transaction. A separate publisher sends the outbox event to Kafka.

---

# Quick Interview Revision

```text
Java 9
  -> Modules, JShell, List.of()

Java 17
  -> Records, Sealed Classes, Pattern Matching

Java 21
  -> Virtual Threads, Record Patterns, Pattern Matching switch

Multithreading
  -> Race condition, synchronized, Lock, AtomicInteger, LongAdder

volatile
  -> Visibility, NOT atomicity

Spring Async
  -> @Async + CompletableFuture
  -> WebClient for reactive/non-blocking I/O

Multiple DBs
  -> Multiple DataSources + EntityManagers + TransactionManagers

Saga
  -> Local transactions + compensating transactions
  -> Choreography / Orchestration

Kafka Fault Tolerance
  -> Multiple brokers
  -> Replication factor 3
  -> min.insync.replicas 2
  -> acks=all
  -> Idempotent producer
  -> Consumer groups
  -> Offset management
  -> Retry + DLT

Kafka + Saga
  -> Events + compensation
  -> Transactional Outbox for DB/Kafka consistency
```
