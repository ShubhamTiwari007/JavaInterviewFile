# Java & Spring Boot Interview Notes

## 1. OOP Concepts

### Four Pillars
- **Encapsulation** — wrap data and methods together; use `private` fields and controlled access.
- **Inheritance** — child class acquires properties/methods of a parent using `extends`.
- **Polymorphism** — one interface/name with multiple forms.
  - Compile-time: method overloading
  - Runtime: method overriding
- **Abstraction** — hide implementation details using abstract classes and interfaces.

### Other Important Concepts
- **Class** — blueprint for objects.
- **Object** — instance of a class.
- **Constructor** — initializes an object.
- `this` — current object.
- `super` — parent class.
- **IS-A** — inheritance, e.g. `Dog IS-A Animal`.
- **HAS-A** — aggregation/composition, e.g. `Car HAS-A Engine`.

---

## 2. Abstract Class

You **cannot directly instantiate** an abstract class:

```java
abstract class Animal {
    abstract void sound();
}

// Animal a = new Animal(); // Error
```

But you can use an abstract-class reference for a concrete child:

```java
Animal a = new Dog();
```

The abstract class's constructor is still called when a child object is created.

---

## 3. String Immutability

`String` is an **immutable class**.

Once a String object is created, its value cannot be changed.

```java
String s = "Hello";
s = s.concat(" World");
```

`concat()` creates a new String; it does not modify the original `"Hello"` object.

### String Pool
- String literals are stored/reused through the String pool.
- The pool is responsible for interning/reusing Strings.
- **Garbage Collector** reclaims String objects that are no longer reachable.
- The String pool itself does not perform garbage collection.

---

## 4. Creating a Custom Immutable Class

Main rules:
1. Make the class `final`.
2. Make fields `private final`.
3. Initialize fields through the constructor.
4. Do not provide setters.
5. Use defensive copies for mutable fields such as `List`.

Example:

```java
final class Employee {
    private final int id;
    private final String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

For mutable fields:

```java
this.skills = List.copyOf(skills);
```

---

## 5. Collection vs Collections

### `Collection`
`Collection` is an **interface** representing a group of objects.

Examples:
- `List`
- `Set`
- `Queue`

Common methods:
- `add()`
- `remove()`
- `contains()`
- `size()`
- `isEmpty()`
- `clear()`

### `Collections`
`Collections` is a **utility class** containing static methods for manipulating collections.

Examples:

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.max(list);
Collections.min(list);
```

**Remember:**
- `Collection` = interface/data abstraction
- `Collections` = utility/helper class

---

# 6. Java Stream API

A Stream is a pipeline for processing data. It does not store data itself.

### Basic workflow

```text
Source
  ↓
Intermediate operations
  ↓
Terminal operation
  ↓
Result
```

Example:

```java
List<Integer> result = numbers.stream()
        .filter(n -> n > 20)
        .map(n -> n * 2)
        .toList();
```

### Source
Examples:
```java
list.stream();
Arrays.stream(array);
Stream.of(1, 2, 3);
```

### Intermediate operations
Generally lazy:
- `filter()`
- `map()`
- `sorted()`
- `distinct()`
- `limit()`
- `skip()`

### Terminal operations
Trigger execution:
- `toList()`
- `collect()`
- `forEach()`
- `count()`
- `reduce()`
- `findFirst()`
- `anyMatch()`

### Important concepts
- **Lazy evaluation** — intermediate operations do not execute until a terminal operation is invoked.
- **Short-circuiting** — operations such as `findFirst()` can stop processing once the answer is found.
- Streams generally do not modify the source collection.
- A stream generally cannot be reused after a terminal operation.

### Common operations

#### `filter()`
Selects elements:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .toList();
```

#### `map()`
Transforms elements:

```java
names.stream()
     .map(String::toUpperCase)
     .toList();
```

#### `reduce()`
Combines multiple values into one:

```java
int sum = numbers.stream()
        .reduce(0, (a, b) -> a + b);
```

#### Parallel stream
```java
list.parallelStream();
```

Can process work across multiple threads, but should only be used when parallel processing actually benefits the workload.

---

# 7. Java Lambda

A lambda expression is a concise implementation of the single abstract method of a **functional interface**.

Syntax:

```java
(parameters) -> expression
```

Example:

```java
Calculator c = (a, b) -> a + b;
```

Functional interface:

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

### Common functional interfaces

| Interface | Input | Output | Common use |
|---|---|---|---|
| `Predicate<T>` | T | boolean | `filter()` |
| `Function<T,R>` | T | R | `map()` |
| `Consumer<T>` | T | void | `forEach()` |
| `Supplier<T>` | none | T | Generate/provide values |

### Method reference

Lambda:

```java
names.forEach(name -> System.out.println(name));
```

Method reference:

```java
names.forEach(System.out::println);
```

---

# 8. Java Annotations

An annotation is metadata attached to code elements such as classes, methods, fields, and parameters.

Syntax:

```java
@Override
```

### Common Java annotations
- `@Override`
- `@Deprecated`
- `@SuppressWarnings`

### Common Spring annotations
- `@SpringBootApplication`
- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`
- `@Configuration`
- `@Bean`
- `@Autowired`

### REST annotations
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PathVariable`
- `@RequestParam`
- `@RequestBody`

### JPA annotations
- `@Entity`
- `@Id`
- `@GeneratedValue`
- `@Column`
- `@Table`
- `@OneToOne`
- `@OneToMany`
- `@ManyToOne`
- `@ManyToMany`

---

# 9. Custom Annotation

Create one using `@interface`:

```java
public @interface MyAnnotation {
}
```

Add values:

```java
public @interface MyAnnotation {
    String name();
    int version() default 1;
}
```

Use it:

```java
@MyAnnotation(name = "Employee", version = 2)
class Employee {
}
```

### `@Target`
Controls where the annotation can be used:

```java
@Target(ElementType.TYPE)
```

### `@Retention`
Controls how long it is retained:

```java
@Retention(RetentionPolicy.RUNTIME)
```

Retention policies:
- `SOURCE` — source code only
- `CLASS` — stored in `.class`
- `RUNTIME` — available through reflection

Example:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    String name();
}
```

Read it with reflection:

```java
MyAnnotation annotation =
        Employee.class.getAnnotation(MyAnnotation.class);

System.out.println(annotation.name());
```

---

# 10. `application.properties` in Spring Boot

Location:

```text
src/main/resources/application.properties
```

Used for externalized configuration.

### Server port

```properties
server.port=8081
```

### Database

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

### JPA

```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### Custom properties

```properties
app.name=Order Service
app.version=1.0
```

Read with:

```java
@Value("${app.name}")
private String appName;
```

For grouped configuration, prefer `@ConfigurationProperties`.

### Profiles

```text
application.properties
application-dev.properties
application-prod.properties
```

Activate a profile:

```properties
spring.profiles.active=dev
```

---

# 11. Spring vs Spring Boot

### Spring Framework
Provides the core framework/infrastructure for Java applications:
- IoC
- Dependency Injection
- Spring MVC
- AOP
- Transaction management
- Data access support

### Spring Boot
Built on Spring and simplifies application development with:
- Auto-configuration
- Starter dependencies
- Embedded servers
- Externalized configuration
- Convention over configuration

### `@SpringBootApplication`

Combines:

```text
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

Typical application:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Simple interview answer

> Spring provides the framework and infrastructure; Spring Boot simplifies Spring application development through auto-configuration, starters, embedded servers, and convention over configuration.

---

# 12. Kafka Workflow

Kafka is a **distributed event-streaming platform**.

Basic flow:

```text
Producer
   ↓
Topic
   ↓
Partition
   ↓
Consumer Group
   ↓
Consumer
```

## Producer

Sends records to Kafka:

```java
ProducerRecord<String, String> record =
        new ProducerRecord<>(
                "orders",
                "order-101",
                "Order Created"
        );

producer.send(record);
```

## Topic

A logical category for events:

```text
orders
  ├── Partition 0
  ├── Partition 1
  └── Partition 2
```

## Partition

Partitions provide:
- Parallelism
- Scalability
- Higher throughput

Ordering is guaranteed **within a partition**, not across the entire topic.

Records with the same key are normally routed to the same partition.

## Broker

A Kafka broker is a Kafka server. A Kafka cluster contains multiple brokers.

```text
Kafka Cluster
 ├── Broker 1
 ├── Broker 2
 └── Broker 3
```

Partitions are distributed across brokers.

## Replication

Kafka can maintain replicas of partitions:

```text
Partition 0
 ├── Broker 1 → Leader
 ├── Broker 2 → Replica
 └── Broker 3 → Replica
```

Replication provides fault tolerance.

## Consumer

Consumers read records from Kafka using `poll()`:

```java
KafkaConsumer<String, String> consumer =
        new KafkaConsumer<>(properties);

consumer.subscribe(List.of("orders"));

while (true) {
    ConsumerRecords<String, String> records =
            consumer.poll(Duration.ofMillis(1000));

    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record.value());
    }
}
```

## Consumer Group

Consumers in the same group share the work.

```text
3 partitions
     ↓
Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

Within a consumer group, a partition is assigned to at most one consumer at a time.

If there are 3 partitions and 5 consumers, at most 3 consumers can actively consume partitions at that time.

## Multiple Consumer Groups

Different groups can independently consume the same topic:

```text
orders topic
   ├── Payment Group
   ├── Inventory Group
   └── Analytics Group
```

This allows the same event to drive multiple independent services.

## Offset

An offset represents a record's position within a partition.

```text
Offset 0 → Order 101
Offset 1 → Order 102
Offset 2 → Order 103
```

Offsets are tracked per consumer group and partition.

## Retention and Replay

Kafka does not normally delete a record simply because a consumer has read it.

Records remain according to the topic's retention configuration. A consumer can reset its offset and reprocess retained records.

---

# 13. Kafka vs SQS

| SQS | Kafka |
|---|---|
| Managed AWS message queue | Distributed event-streaming platform |
| Queue-oriented | Topic/partition-oriented |
| Commonly used for asynchronous tasks | Commonly used for event streaming |
| Message is typically deleted after successful processing | Records remain according to retention |
| Replay is limited compared with Kafka | Replay/reprocessing is a core capability |
| FIFO queues support ordering | Ordering within a partition |
| Fully managed AWS service | Kafka can be self-managed or managed |

### Simple mental model

```text
SQS:
Producer → Queue → Consumer

Kafka:
Producer → Topic → Partition → Consumer Group → Consumer
```

**SQS:** "Give this task to a consumer."

**Kafka:** "Store this event stream so multiple consumers can process it independently."

---

# Quick Interview Revision

## OOP
```text
Encapsulation → Data hiding/control
Inheritance   → Parent-child reuse
Polymorphism  → Multiple forms
Abstraction   → Hide implementation details
```

## Java String
```text
String → Immutable
String Pool → Reuse/interning
GC → Reclaims unreachable objects
```

## Collections
```text
Collection → Interface
Collections → Utility class
```

## Stream
```text
Source
  ↓
Intermediate operations
  ↓
Terminal operation
  ↓
Result
```

## Lambda
```text
Functional Interface
        ↓
      Lambda
        ↓
 Stream operations
```

## Custom Annotation
```text
@interface → create
@Target     → where usable
@Retention  → how long retained
```

## Spring Boot
```text
Spring
  ↓
Spring Boot
  ↓
Auto-configuration
+ Starters
+ Embedded Server
+ Externalized Configuration
```

## Kafka
```text
Producer
  ↓
Topic
  ↓
Partition
  ↓
Consumer Group
  ↓
Consumer
  ↓
Offset
```
