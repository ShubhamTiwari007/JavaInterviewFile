# Java Design Patterns & Microservices Design Patterns — Interview Guide
## For 5–6 Years Experienced Java / Spring Boot Developers

> This guide combines the design-pattern discussion from the conversation and extends it with **microservices design patterns**, interview questions, practical Java/Spring examples, comparisons, and interview-oriented trade-offs.

---

# 1. Design Patterns — Interview-Level Summary

| Pattern | Type | Core Problem | Java/Spring Example | What interviewer expects at 5–6 YOE |
|---|---|---|---|---|
| **Singleton** | Creational | Need exactly one shared instance | Spring singleton-scoped bean | Thread safety, serialization, reflection, class loaders |
| **Factory Method** | Creational | Object creation should be delegated | `NotificationFactory` | Avoiding `new` everywhere, Open/Closed Principle |
| **Abstract Factory** | Creational | Create related families of objects | Cross-platform UI components | Difference from Factory Method |
| **Builder** | Creational | Construct complex objects cleanly | DTO/request builders | Immutability, optional parameters |
| **Prototype** | Creational | Create objects by copying | Cloning/config templates | Deep vs shallow copy |
| **Adapter** | Structural | Make incompatible interfaces work together | Legacy API adapter | Composition over inheritance |
| **Decorator** | Structural | Add behavior dynamically | Java I/O streams | Difference from inheritance/proxy |
| **Facade** | Structural | Hide subsystem complexity | Service/API facade | Simplifying clients without hiding business rules |
| **Proxy** | Structural | Control access to an object | Spring AOP, transactions | JDK vs CGLIB proxies |
| **Composite** | Structural | Treat individual and group objects uniformly | File/folder hierarchy | Recursive structures |
| **Bridge** | Structural | Separate abstraction from implementation | Notification + delivery mechanism | Avoid class explosion |
| **Flyweight** | Structural | Share objects to reduce memory usage | String pool, cached objects | Intrinsic vs extrinsic state |
| **Strategy** | Behavioral | Select algorithm/behavior dynamically | Payment/discount strategies | Composition + Open/Closed Principle |
| **Observer** | Behavioral | Notify multiple dependents | Spring events | Event-driven architecture |
| **Chain of Responsibility** | Behavioral | Pass requests through handlers | Servlet filters/Spring Security | Ordering and termination |
| **Template Method** | Behavioral | Fixed algorithm with customizable steps | Framework lifecycle | Inheritance vs composition |
| **Command** | Behavioral | Encapsulate a request/action | Job queues, undo | Decoupling sender/receiver |
| **State** | Behavioral | Behavior changes according to state | Order/workflow | Avoiding large `if/else` blocks |
| **Iterator** | Behavioral | Traverse collections without exposing internals | Java `Iterator` | Encapsulation |
| **Mediator** | Behavioral | Reduce direct communication between objects | Chat room/workflow | Reduce many-to-many dependencies |
| **Memento** | Behavioral | Save/restore object state | Undo functionality | Encapsulation of state |
| **Visitor** | Behavioral | Add operations without modifying classes | AST processing | Double dispatch |
| **Interpreter** | Behavioral | Interpret grammar/expressions | Expression engines | Usually less common in enterprise Java |

---

# 2. Categories of Design Patterns

| Category | Description | Patterns |
|---|---|---|
| **Creational** | Object creation mechanisms | Singleton, Factory Method, Abstract Factory, Builder, Prototype |
| **Structural** | Composition of classes/objects | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| **Behavioral** | Communication and responsibilities | Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor, Interpreter |

---

# 3. Singleton Pattern

Singleton guarantees one logical instance and a global access point.

## Thread-safe implementation with double-checked locking

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

### Why is `volatile` required?

`volatile` provides visibility and ordering guarantees for the instance reference. It prevents unsafe publication/reordering issues associated with double-checked locking.

## Enum Singleton

```java
public enum Singleton {
    INSTANCE;

    public void execute() {
        System.out.println("Executing...");
    }
}
```

### Common interview questions

| Question | Expected answer |
|---|---|
| Can reflection break Singleton? | Reflection can potentially access a private constructor unless additional protection is used. Enum Singleton has stronger protection. |
| Can serialization break Singleton? | A Serializable Singleton can create another instance unless `readResolve()` is used. |
| What about cloning? | Prevent cloning or return the same instance. |
| Is Singleton always good? | No. It introduces global state and can make testing/dependency management harder. |
| Is a Spring singleton the same as a Java Singleton? | No. Spring singleton scope means one bean instance per Spring container, not necessarily one JVM-wide instance. |

---

# 4. Factory Pattern

Factory centralizes object creation.

```java
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class NotificationFactory {

    public static Notification create(String type) {

        return switch (type.toUpperCase()) {
            case "EMAIL" -> new EmailNotification();
            case "SMS" -> new SmsNotification();
            default -> throw new IllegalArgumentException(
                    "Unsupported notification type"
            );
        };
    }
}
```

### Interview question: Why Factory instead of `new`?

A strong answer:

> The factory centralizes object-creation logic, hides concrete implementations from clients, and allows construction logic to evolve without spreading object-creation decisions throughout the application.

### Important trade-off

A giant `switch` factory can itself become difficult to maintain. For a highly extensible system, a registry or dependency-injection-based factory can be preferable.

---

# 5. Factory Method vs Abstract Factory

| Feature | Factory Method | Abstract Factory |
|---|---|---|
| Main purpose | Create a product | Create families of related products |
| Complexity | Lower | Higher |
| Example | `NotificationFactory` | `WindowsUIFactory`, `MacUIFactory` |
| Focus | One product hierarchy | Multiple related product hierarchies |

Interview answer:

> Factory Method generally deals with creating a product through a method that allows a concrete implementation to determine the product, while Abstract Factory provides an interface for creating a family of related objects.

---

# 6. Builder Pattern

Builder is useful for complex objects with many optional fields.

```java
User user = User.builder()
        .firstName("John")
        .lastName("Doe")
        .age(30)
        .email("john@example.com")
        .city("Delhi")
        .active(true)
        .build();
```

A manual implementation typically uses:

```java
public class User {

    private final String firstName;
    private final String lastName;
    private final Integer age;

    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {

        private String firstName;
        private String lastName;
        private Integer age;

        public Builder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }

        public Builder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }

        public Builder age(Integer age) {
            this.age = age;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

### Interview questions

**Why Builder instead of constructor?**

- Better readability.
- Handles optional parameters.
- Avoids telescoping constructors.
- Works naturally with immutable objects.
- Allows validation in `build()`.

**Is Lombok `@Builder` the Builder pattern?**

The annotation is an implementation mechanism that generates Builder-pattern code.

---

# 7. Adapter Pattern

Adapter makes incompatible interfaces work together.

```java
interface PaymentProcessor {
    void pay(double amount);
}

class LegacyPaymentGateway {
    void makePayment(double amount) {
        // legacy implementation
    }
}

class PaymentAdapter implements PaymentProcessor {

    private final LegacyPaymentGateway gateway;

    public PaymentAdapter(LegacyPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

Typical architecture:

```text
Application
    |
PaymentProcessor
    |
PaymentAdapter
    |
Third-party / Legacy API
```

---

# 8. Decorator vs Proxy

| Decorator | Proxy |
|---|---|
| Primarily adds behavior | Primarily controls access |
| Often used for feature composition | Used for security, lazy loading, remoting, AOP |
| Multiple decorators can be chained | Usually represents/control access to another object |
| Java I/O is a classic example | Spring AOP is a common example |

Java I/O example:

```text
InputStream
   |
BufferedInputStream
   |
GZIPInputStream
```

Each wrapper adds behavior.

---

# 9. Facade Pattern

Facade provides a simplified interface over a complex subsystem.

```java
class OrderFacade {

    public void placeOrder(Order order) {
        inventory.reserve(order);
        payment.charge(order);
        shipping.createShipment(order);
        notification.send(order);
    }
}
```

Client:

```java
orderFacade.placeOrder(order);
```

### Facade vs Service

A service can contain business logic. A facade primarily provides a simplified entry point over multiple subsystem operations.

---

# 10. Strategy Pattern

Strategy is one of the most important patterns for experienced Java developers.

Instead of:

```java
if (paymentType.equals("CARD")) {
    // card
} else if (paymentType.equals("UPI")) {
    // UPI
} else if (paymentType.equals("PAYPAL")) {
    // PayPal
}
```

Use an abstraction:

```java
interface PaymentStrategy {
    void pay(BigDecimal amount);
}
```

Implementations:

```java
class CardPaymentStrategy implements PaymentStrategy {

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("Card payment");
    }
}

class UpiPaymentStrategy implements PaymentStrategy {

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("UPI payment");
    }
}
```

Spring-friendly registry:

```java
@Service
class PaymentService {

    private final Map<String, PaymentStrategy> strategies;

    PaymentService(List<PaymentStrategy> strategies) {
        this.strategies = strategies.stream()
                .collect(Collectors.toMap(
                        PaymentStrategy::getType,
                        Function.identity()
                ));
    }

    public void pay(String type, BigDecimal amount) {
        PaymentStrategy strategy = strategies.get(type);

        if (strategy == null) {
            throw new IllegalArgumentException(
                    "Unsupported payment type"
            );
        }

        strategy.pay(amount);
    }
}
```

### Strategy vs Factory

> Factory determines **which object to create**. Strategy determines **which behavior/algorithm to execute**.

They can also be combined:

```text
Factory
   |
select Strategy
   |
Strategy executes behavior
```

---

# 11. Observer Pattern

Useful for decoupled event notification.

```text
Order Created
     |
 ┌───┼─────────────┬────────────┐
Email SMS       Analytics      Audit
```

Spring example:

```java
public record OrderCreatedEvent(Long orderId) {}
```

Publishing:

```java
applicationEventPublisher.publishEvent(
    new OrderCreatedEvent(orderId)
);
```

Listener:

```java
@EventListener
public void handle(OrderCreatedEvent event) {
    // notification
}
```

### Important Spring question

Are Spring events automatically asynchronous?

> No. Standard Spring application event handling is synchronous by default unless asynchronous handling is configured.

---

# 12. Chain of Responsibility

A request moves through a chain of handlers.

```text
Request
  |
Authentication
  |
Authorization
  |
Logging
  |
Validation
  |
Controller
```

Common Java/Spring examples:

- Servlet filters
- Spring Security filter chains
- Validation pipelines
- Processing pipelines

Each handler can process, reject, or pass the request onward.

---

# 13. Template Method

Template Method defines the algorithm skeleton while subclasses customize steps.

```java
abstract class PaymentProcessor {

    public final void execute() {
        validate();
        authenticate();
        process();
        audit();
    }

    protected abstract void process();

    protected void validate() {
        // common
    }

    protected void authenticate() {
        // common
    }

    protected void audit() {
        // common
    }
}
```

Key distinction:

- Template Method → inheritance
- Strategy → composition

---

# 14. State Pattern

State is useful when behavior changes according to object state.

```text
CREATED
   |
PAID
   |
SHIPPED
   |
DELIVERED
```

Instead of large conditional logic, behavior can be moved to state-specific classes:

```text
Order
 |
OrderState
 ├── CreatedState
 ├── PaidState
 ├── ShippedState
 └── DeliveredState
```

### Strategy vs State

| Strategy | State |
|---|---|
| Client chooses algorithm | Behavior changes because object state changes |
| Usually external choice | Usually internal lifecycle |
| Composition | Composition |
| Payment strategy | Order state |

---

# 15. Proxy Pattern and Spring

Spring uses proxies extensively for features such as:

- `@Transactional`
- `@Cacheable`
- `@Async`
- Spring AOP

Architecture:

```text
Client
  |
Spring Proxy
  |
Actual Bean
```

### JDK Dynamic Proxy vs CGLIB

| JDK Dynamic Proxy | CGLIB-style proxy |
|---|---|
| Interface-based | Subclass-based |
| Implements interfaces | Proxies concrete classes |
| Standard Java proxy mechanism | Bytecode/subclass approach |

### Important `@Transactional` interview question

Why can self-invocation cause `@Transactional` not to behave as expected?

```java
this.someTransactionalMethod();
```

The internal call normally does not pass through the Spring proxy.

Conceptually:

```text
External caller
      |
Spring Proxy
      |
@Transactional method
```

Self-invocation can bypass that proxy boundary.

---

# 16. SOLID and Design Patterns

| SOLID Principle | Commonly related patterns |
|---|---|
| SRP | Facade, Command |
| OCP | Strategy, Factory, Decorator |
| LSP | Polymorphic designs, Template Method |
| ISP | Adapter |
| DIP | Strategy, Factory, Dependency Injection |

Important interview point:

> A design pattern does not automatically guarantee SOLID compliance. It is a tool for addressing a recurring design problem.

---

# 17. Common Pattern Comparisons

## Factory vs Builder vs Prototype

| Pattern | Question it answers |
|---|---|
| Factory | Which object should I create? |
| Builder | How should I construct this complex object? |
| Prototype | Can I create this object by copying an existing object? |

## Adapter vs Facade vs Decorator vs Proxy

| Pattern | Core question |
|---|---|
| Adapter | How can incompatible interfaces work together? |
| Facade | How can I simplify access to a complex subsystem? |
| Decorator | How can I dynamically add behavior? |
| Proxy | How can I control access to the object? |

---

# 18. Microservices Design Patterns

Microservices introduce another family of patterns. These are not simply replacements for GoF object-oriented patterns; they address **distributed-system architecture**.

## Core Microservices Pattern Table

| Microservice Pattern | Problem Solved | Typical Example | Key Interview Topic |
|---|---|---|---|
| **API Gateway** | Single entry point for clients | Spring Cloud Gateway | Routing, auth, rate limiting |
| **BFF (Backend for Frontend)** | Different clients need different APIs | Mobile BFF / Web BFF | Client-specific aggregation |
| **Service Discovery** | Services need to locate instances dynamically | Eureka / Consul / Kubernetes DNS | Dynamic service locations |
| **Circuit Breaker** | Prevent cascading failures | Resilience4j | Closed/Open/Half-Open |
| **Retry** | Recover from transient failures | Resilience4j | Backoff, jitter, idempotency |
| **Timeout** | Prevent requests waiting indefinitely | HTTP client timeout | Latency/failure isolation |
| **Bulkhead** | Isolate resources/failures | Thread/connection pools | Prevent resource exhaustion |
| **Rate Limiting** | Control request volume | Gateway/token bucket | Protect downstream systems |
| **Load Balancing** | Distribute traffic | Kubernetes/service LB | Client/server-side balancing |
| **Saga** | Manage distributed transactions | Order → Payment → Inventory | Choreography vs orchestration |
| **Outbox** | Reliable DB + event publication | Order DB + Outbox table | Atomicity |
| **CQRS** | Separate read/write models | Order command/query services | Eventual consistency |
| **Event Sourcing** | Store state changes as events | Account/order event stream | Rebuild current state |
| **Strangler Fig** | Incrementally migrate monolith | Old + new services | Migration strategy |
| **Anti-Corruption Layer** | Protect new domain from legacy model | Legacy adapter layer | Domain isolation |
| **Sidecar** | Add infrastructure behavior beside service | Proxy/agent container | Service mesh |
| **Service Mesh** | Infrastructure-level service communication | Istio/Envoy | mTLS, telemetry, routing |
| **Centralized Configuration** | Manage configuration consistently | Config Server/K8s ConfigMaps | Externalized config |
| **Distributed Tracing** | Trace calls across services | OpenTelemetry | Correlation/trace IDs |
| **Health Check** | Determine service health | Actuator/Kubernetes probes | Liveness/readiness |
| **Cache-Aside** | Reduce database load | Redis + DB | Cache consistency |
| **Database-per-Service** | Preserve service ownership | Order DB, Payment DB | Loose coupling |
| **Shared Database** | Share data across services | Common RDBMS | Strong coupling/trade-offs |
| **Aggregator** | Combine multiple service responses | Product details + pricing + inventory | Latency/failure handling |
| **Event-Driven Architecture** | Decouple services | Kafka/RabbitMQ | Async communication |

---

# 19. API Gateway Pattern

Architecture:

```text
                 Clients
                    |
               API Gateway
          _________|_________
         |         |         |
      Order     Payment    User
     Service    Service   Service
```

The gateway can handle:

- routing
- authentication
- authorization
- rate limiting
- request aggregation
- observability
- TLS termination

### Interview question

Why not let clients directly call every microservice?

Possible problems:

- many service endpoints exposed externally;
- client becomes coupled to internal topology;
- authentication and cross-cutting concerns are duplicated;
- mobile/web clients may require different aggregation.

But an API Gateway can also become a bottleneck or a single operational failure domain if poorly designed.

---

# 20. BFF — Backend for Frontend

Instead of one generic gateway API:

```text
Mobile
   |
Mobile BFF
   |
Services

Web
 |
Web BFF
 |
Services
```

Useful when different clients have substantially different data requirements.

### Interview question

BFF vs API Gateway?

> API Gateway is a general entry point for routing and cross-cutting concerns. BFF is specifically shaped around the needs of a particular client experience.

They can coexist.

---

# 21. Service Discovery

Microservice instances can change dynamically.

Instead of hard-coding:

```text
http://10.10.10.23:8080
```

use service discovery:

```text
Order Service
     |
Service Discovery
     |
Order instance A
Order instance B
Order instance C
```

Modern Kubernetes deployments often rely on Kubernetes Services/DNS rather than a separate application-level discovery mechanism.

### Interview question

Client-side vs server-side discovery?

**Client-side:**

```text
Client → Registry → Selected Service
```

**Server-side:**

```text
Client → Load Balancer → Service
```

---

# 22. Circuit Breaker Pattern

Problem:

```text
Service A
   |
Service B
   |
Service C
```

If C becomes slow:

```text
A → B → C
       timeout
       timeout
       timeout
```

Threads/resources can become exhausted and the failure can cascade.

Circuit breaker introduces states:

```text
        failure threshold
CLOSED -----------------> OPEN
   ^                         |
   |                         | timeout/wait
   |                         v
   +--------------------- HALF-OPEN
            success
```

### States

| State | Behavior |
|---|---|
| Closed | Requests flow normally |
| Open | Calls fail fast |
| Half-Open | Limited calls test recovery |

### Interview question

Circuit breaker vs retry?

> Retry attempts the operation again. Circuit breaker prevents repeatedly calling an unhealthy dependency.

They can be combined carefully.

---

# 23. Retry Pattern

Useful for transient failures.

```text
Request
  |
Attempt 1 → failure
  |
wait
  |
Attempt 2 → failure
  |
wait
  |
Attempt 3 → success
```

Prefer exponential backoff:

```text
100ms
200ms
400ms
800ms
```

Add jitter in distributed systems to avoid synchronized retry storms.

### Critical interview point

Retries require **idempotency** consideration.

For example:

```text
POST /payment
```

If the server processed the payment but the response was lost, blindly retrying could charge twice.

Use mechanisms such as idempotency keys where appropriate.

---

# 24. Timeout Pattern

Never allow remote calls to wait indefinitely.

Example:

```text
Service A
   |
   | timeout = 2 seconds
   v
Service B
```

If B doesn't respond within the timeout, A should stop waiting and apply a defined fallback/error strategy.

### Interview question

Why are timeouts essential?

Because in distributed systems, waiting threads/connections are resources. Without bounded waiting, slow dependencies can exhaust the caller's resources and contribute to cascading failure.

---

# 25. Bulkhead Pattern

Named after ship compartments.

If one dependency consumes all resources:

```text
Without Bulkhead

Service
  |
  +-- Payment → all threads
  |
  +-- Inventory
  |
  +-- User
```

Payment failure can starve the entire service.

With isolation:

```text
Service
 |
 +-- Payment Pool
 |
 +-- Inventory Pool
 |
 +-- User Pool
```

Failure in one pool is contained.

### Interview answer

> Bulkhead limits the blast radius of a dependency failure by isolating resources such as thread pools, connection pools, or concurrency limits.

---

# 26. Saga Pattern

One of the most important microservices interview topics.

Traditional transaction:

```text
Order
Payment
Inventory
```

In a monolith, one database transaction might cover everything.

In microservices, each service owns its database.

Saga breaks the business transaction into local transactions.

```text
Create Order
    |
Reserve Inventory
    |
Charge Payment
    |
Confirm Order
```

If payment fails:

```text
Payment Failed
     |
Release Inventory
     |
Cancel Order
```

These are compensating transactions.

## Saga choreography

Services communicate through events:

```text
Order Service
     |
 OrderCreated
     v
Inventory Service
     |
InventoryReserved
     v
Payment Service
```

## Saga orchestration

A central orchestrator coordinates:

```text
             Saga Orchestrator
              /       |       \
             v        v        v
          Order   Inventory  Payment
```

### Choreography vs orchestration

| Choreography | Orchestration |
|---|---|
| Event-driven | Central coordinator |
| Less central control | Clear workflow |
| Can become hard to visualize at scale | Orchestrator can become complex |
| Services react to events | Orchestrator tells participants what to do |

### Key interview point

Saga does **not** provide traditional ACID transaction semantics across independent databases. It provides business-level consistency using local transactions and compensating actions.

---

# 27. Transactional Outbox Pattern

A classic distributed consistency problem:

```text
DB transaction
      +
publish Kafka event
```

What if DB commit succeeds but Kafka publishing fails?

Or Kafka succeeds but DB transaction rolls back?

Transactional Outbox addresses this.

```text
Service
  |
  +------ DB Transaction ------+
  |                            |
Business Data              Outbox Event
                               |
                               v
                         Outbox Publisher
                               |
                               v
                             Kafka
```

The business row and outbox row are written in the same local DB transaction.

A publisher later reads the outbox and publishes the event.

### Interview question

Does Outbox guarantee exactly-once processing?

Not automatically.

Consumers still need idempotency and appropriate delivery/processing semantics.

---

# 28. CQRS

Command Query Responsibility Segregation.

Instead of one model serving both reads and writes:

```text
Commands → Write Model
Queries  → Read Model
```

Architecture:

```text
                 API
               /     \
          Commands   Queries
             |          |
        Write Model  Read Model
             |          |
           DB        Read Store
```

Useful when read and write requirements differ significantly.

### Interview warning

Don't say:

> "CQRS means two databases."

It does not inherently require two databases. It separates command and query responsibilities; separate stores are a common implementation.

---

# 29. Event Sourcing

Instead of storing only current state:

```text
Account balance = 500
```

store state-changing events:

```text
AccountCreated
MoneyDeposited(1000)
MoneyWithdrawn(300)
MoneyWithdrawn(200)
```

Current state can be reconstructed by replaying events.

### Event Sourcing vs Event-Driven Architecture

They are not the same.

**Event-driven architecture:**

> Services communicate/react through events.

**Event sourcing:**

> Events are the authoritative representation of state changes.

You can use event-driven architecture without event sourcing.

---

# 30. Strangler Fig Pattern

Used to migrate a monolith incrementally.

```text
                 Client
                    |
              Routing Layer
               /          \
        New Service      Legacy
             |             |
        New functionality  Old functionality
```

Over time:

```text
Legacy functionality
       ↓
replaced incrementally
       ↓
new services
```

Useful because replacing an entire monolith at once is risky.

### Interview question

Why not rewrite the monolith completely?

A rewrite has significant migration, compatibility, operational, and unknown-behavior risks. Incremental migration allows functionality to move in controlled slices.

---

# 31. Anti-Corruption Layer

Useful when integrating with a legacy system.

```text
New Domain
    |
Anti-Corruption Layer
    |
Legacy Domain Model
```

The ACL translates between models so the legacy concepts do not leak into the new domain.

This is particularly relevant in Domain-Driven Design.

---

# 32. Sidecar Pattern

A supporting process/container runs alongside the application.

```text
Pod
+----------------------+
| Application          |
|                      |
| Sidecar Proxy        |
+----------------------+
```

The sidecar may provide:

- telemetry
- proxying
- security
- configuration
- service-to-service communication

A service mesh commonly uses sidecar proxies, although modern service-mesh architectures may also use other deployment models.

---

# 33. Service Mesh

A service mesh moves communication infrastructure concerns away from application code.

```text
Service A
   |
Proxy
   |
Network
   |
Proxy
   |
Service B
```

Common concerns:

- mTLS
- traffic management
- retries
- observability
- service identity

Examples include technologies such as Istio and Envoy.

### Interview question

Why use a service mesh instead of implementing retries/security in Java?

> A mesh can standardize infrastructure-level communication concerns across many services, reducing duplicated application code. However, business-level retry or compensation logic still belongs in the application domain.

---

# 34. Cache-Aside Pattern

Very common in microservices.

```text
Application
    |
    v
Check Cache
   / \
 hit  miss
 |      |
return  DB
        |
        v
      Cache
        |
      return
```

### Advantages

- reduces database load;
- improves read latency.

### Challenges

- stale data;
- cache invalidation;
- cache stampede;
- consistency;
- eviction policies.

---

# 35. Database-per-Service Pattern

A core microservices principle:

```text
Order Service    → Order DB
Payment Service  → Payment DB
User Service     → User DB
```

Benefits:

- independent ownership;
- independent schema evolution;
- reduced coupling;
- service autonomy.

Trade-off:

> Cross-service joins become distributed operations and may require APIs, events, CQRS, or data replication.

---

# 36. Shared Database Pattern

Multiple services access the same database.

```text
Order Service ----\
Payment Service ---+--> Shared DB
User Service -----/
```

This can be easy initially but creates:

- schema coupling;
- deployment coupling;
- ownership ambiguity;
- harder independent evolution.

It can still be pragmatic during migrations or in legacy environments, but it reduces service autonomy.

---

# 37. Aggregator Pattern

One service combines results from multiple services.

```text
              Aggregator
              /   |   \
             /    |    \
        Product Price Inventory
```

Example:

```text
GET /product-page/100

{
  "product": {...},
  "price": {...},
  "inventory": {...}
}
```

### Interview concerns

- downstream latency;
- partial failures;
- parallel calls;
- timeouts;
- caching;
- response consistency.

---

# 38. Distributed Tracing

A request can travel through:

```text
API Gateway
   ↓
Order Service
   ↓
Payment Service
   ↓
Fraud Service
```

Logs from each service must be correlated.

Typical concepts:

- Trace ID
- Span ID
- Context propagation

OpenTelemetry is commonly used for standardized observability instrumentation.

---

# 39. Health Checks

Two important concepts in containerized deployments:

### Liveness

> Is the application process healthy enough to remain running?

### Readiness

> Is the application ready to receive traffic?

A service can be alive but not ready because it is still initializing or temporarily unable to serve traffic.

Spring Boot Actuator is commonly used to expose health information.

---

# 40. API Gateway vs Load Balancer vs Service Mesh

| Component | Main responsibility |
|---|---|
| API Gateway | External API entry point, routing, auth, rate limiting, aggregation |
| Load Balancer | Distribute traffic among instances |
| Service Mesh | Service-to-service communication infrastructure |

They can all coexist.

---

# 41. Retry + Circuit Breaker + Timeout + Bulkhead

These patterns are often used together.

```text
Client
  |
Timeout
  |
Bulkhead
  |
Circuit Breaker
  |
Retry
  |
Remote Service
```

The exact ordering depends on the technology and desired semantics.

### Important interview point

Don't blindly add all four.

For example:

- retries can increase load;
- circuit breakers can fail fast;
- timeouts bound latency;
- bulkheads isolate resources.

The design must consider failure modes and idempotency.

---

# 42. Microservices Interview Scenario

### Question

> Design an order-processing system using microservices.

A reasonable high-level architecture:

```text
                     Client
                        |
                  API Gateway
                        |
                  Order Service
                        |
                +-------+--------+
                |                |
          Order DB           Outbox
                                   |
                                   v
                                Kafka
                              /   |   \
                             /    |    \
                            v     v     v
                     Inventory Payment Notification
                        |        |
                     DB        DB
```

Possible flow:

```text
1. Client creates order
2. Order Service stores order + outbox event
3. Outbox publisher sends OrderCreated
4. Inventory reserves stock
5. Payment authorizes/charges
6. Services publish resulting events
7. Order state progresses
8. Failures trigger compensation through Saga
```

### Questions the interviewer can ask next

**What if Payment fails?**

Use a Saga compensation such as releasing reserved inventory and transitioning the order to a failed/cancelled state.

**What if Kafka publishing fails?**

Transactional Outbox prevents losing the event between the DB transaction and event publication.

**What if Payment receives the same event twice?**

Make the consumer idempotent using an event ID/business key/idempotency record.

**What if Inventory is down?**

Use timeout/circuit breaker and define whether the order remains pending, fails, or is retried asynchronously.

**What if Payment is slow?**

Bound the synchronous call with a timeout. Depending on business requirements, move the workflow to an asynchronous state.

---

# 43. Design Pattern Interview Questions — 5–6 YOE

## Core questions

1. What is a design pattern?
2. What are the three major GoF categories?
3. Explain Singleton and its thread-safety.
4. How can Singleton be broken?
5. Factory vs Abstract Factory?
6. Factory vs Builder?
7. Strategy vs State?
8. Strategy vs Factory?
9. Adapter vs Facade?
10. Decorator vs Proxy?
11. Composition vs inheritance?
12. How do patterns relate to SOLID?
13. How does Spring use Proxy?
14. How does `@Transactional` relate to proxies?
15. Why does self-invocation affect Spring AOP?
16. Where does Spring use Chain of Responsibility?
17. When should you NOT use a design pattern?
18. How would you replace a large `if/else` or `switch`?
19. Design a payment system with multiple providers.
20. Design a notification system.

---

# 44. Microservices Interview Questions — 5–6 YOE

## Architecture

1. What are the benefits and disadvantages of microservices?
2. Monolith vs modular monolith vs microservices?
3. How do you decide service boundaries?
4. What is bounded context?
5. What makes a service independently deployable?
6. Why should each service own its data?
7. When would you avoid microservices?

## Communication

8. REST vs messaging?
9. Synchronous vs asynchronous communication?
10. When would you use Kafka/RabbitMQ?
11. What is eventual consistency?
12. How do you handle duplicate messages?
13. What is idempotency?
14. How do you handle message ordering?
15. How do you handle poison messages?

## Resilience

16. Explain Circuit Breaker.
17. Circuit Breaker vs Retry?
18. Why are timeouts important?
19. Explain Bulkhead.
20. What is a retry storm?
21. Why use exponential backoff and jitter?
22. How do you prevent cascading failures?

## Transactions

23. Why can't we simply use `@Transactional` across microservices?
24. Explain Saga.
25. Choreography vs orchestration?
26. What is a compensating transaction?
27. Explain Transactional Outbox.
28. Outbox vs 2PC?
29. How do you achieve idempotent processing?

## Data

30. Database-per-service?
31. Why avoid shared database?
32. When could shared DB be pragmatic?
33. CQRS?
34. Event sourcing?
35. CQRS vs event sourcing?

## Infrastructure

36. API Gateway?
37. BFF?
38. Service discovery?
39. Service mesh?
40. Sidecar?
41. Distributed tracing?
42. Health checks?
43. Liveness vs readiness?
44. Rate limiting?
45. Load balancing?

## Migration

46. How would you migrate a monolith?
47. Explain Strangler Fig.
48. What is an Anti-Corruption Layer?
49. How do you identify service boundaries during migration?
50. How do you prevent a distributed monolith?

---

# 45. High-Value Comparison Table

| Topic | Option A | Option B | Key distinction |
|---|---|---|---|
| Object creation | Factory | Builder | Which object vs how to construct |
| Behavior | Strategy | State | Selected algorithm vs lifecycle state |
| Composition | Adapter | Facade | Compatibility vs simplification |
| Added behavior | Decorator | Proxy | Enhancement vs access control |
| Communication | Observer | Mediator | Notification vs centralized interaction |
| Workflow | Chain of Responsibility | Template Method | Handler pipeline vs algorithm skeleton |
| Distributed transaction | Saga | 2PC | Compensation/eventual consistency vs coordinated atomic commit |
| Reliable event publishing | Outbox | Direct publish | Atomic local DB write + eventual publication |
| Service communication | REST | Messaging | Synchronous request/response vs asynchronous events |
| Entry point | API Gateway | BFF | General gateway vs client-specific backend |
| Failure handling | Retry | Circuit Breaker | Reattempt vs fail-fast protection |
| Failure isolation | Bulkhead | Circuit Breaker | Resource isolation vs dependency call prevention |
| State/query design | CRUD | CQRS | Single model vs separated command/query models |
| Event architecture | Event-driven | Event sourcing | Events for communication vs events as source of state |
| Migration | Strangler Fig | Big-bang rewrite | Incremental migration vs replacement |

---

# 46. What a 5–6 YOE Answer Should Sound Like

Avoid:

> "Strategy pattern is used to select an algorithm at runtime."

Better:

> "I'd use Strategy when I have multiple independently evolving implementations of the same behavior. For example, payment processing could have Card, UPI and Wallet strategies. In Spring I'd expose a common interface, inject implementations, and maintain a registry keyed by payment type. This avoids a growing conditional block and makes adding a strategy localized. However, if there are only two fixed cases and trivial logic, I would not introduce the abstraction just for the pattern."

This demonstrates:

```text
Pattern knowledge
      +
Java
      +
Spring
      +
SOLID
      +
Maintainability
      +
Trade-offs
      +
Testing
      +
Operational thinking
```

---

# 47. Practical Priority List for a 5–6 YOE Java/Spring Boot Interview

| Priority | Pattern / Topic | Why |
|---:|---|---|
| 1 | **Strategy** | Common real-world extensibility problem |
| 2 | **Factory** | Object creation and extensibility |
| 3 | **Singleton** | Thread safety and Java fundamentals |
| 4 | **Builder** | Common Java/Spring usage |
| 5 | **Proxy** | Critical for Spring AOP and transactions |
| 6 | **Observer / Events** | Event-driven systems |
| 7 | **Chain of Responsibility** | Spring Security/filter pipeline |
| 8 | **Circuit Breaker** | Distributed-system resilience |
| 9 | **Saga** | Distributed transactions |
| 10 | **Transactional Outbox** | Reliable event publication |
| 11 | **API Gateway / BFF** | Microservice edge architecture |
| 12 | **Retry / Timeout / Bulkhead** | Resilience |
| 13 | **CQRS** | Read/write scaling and architecture |
| 14 | **Database-per-Service** | Data ownership |
| 15 | **Distributed Tracing** | Production troubleshooting |
| 16 | **Strangler Fig** | Monolith migration |
| 17 | **Adapter** | External/legacy integrations |
| 18 | **Decorator** | Behavior composition |
| 19 | **Facade** | Subsystem simplification |
| 20 | **State / Template Method / Command** | Common design discussions |

---

# 48. Final Interview Preparation Framework

For every important pattern, prepare these seven things:

```text
1. What problem does it solve?
          ↓
2. What are its participants?
          ↓
3. Implement it in Java
          ↓
4. Show a Spring Boot example
          ↓
5. Explain a real production use case
          ↓
6. Explain alternatives and trade-offs
          ↓
7. Explain testing / failure / scalability implications
```

For microservices patterns, extend that to:

```text
Business problem
      ↓
Service boundary
      ↓
Communication model
      ↓
Data ownership
      ↓
Consistency model
      ↓
Failure handling
      ↓
Observability
      ↓
Scalability
      ↓
Operational trade-offs
```

The most important principle for a 5–6 year developer is:

> **Do not use a pattern because you know the pattern. Use it because you can clearly identify the design problem it solves and explain the trade-offs it introduces.**



---

# 49. Design Pattern Interview Questions — Questions + Answers

## Q1. What is a design pattern?

**Answer:**  
A design pattern is a reusable design approach for a recurring software-design problem. It is not a library, framework, or copy-paste code solution. It describes a structure, participating objects, responsibilities, and trade-offs.

---

## Q2. What are the three major GoF categories?

**Answer:**

- **Creational:** object creation — Singleton, Factory Method, Abstract Factory, Builder, Prototype.
- **Structural:** object/class composition — Adapter, Decorator, Facade, Proxy, Composite, Bridge, Flyweight.
- **Behavioral:** communication and responsibility — Strategy, Observer, State, Command, Template Method, Chain of Responsibility, etc.

---

## Q3. Explain Singleton and its thread-safety.

**Answer:**  
Singleton restricts a class to one logical instance within its intended scope. Thread-safe implementations include eager initialization, static-holder idiom, enum Singleton, and correctly implemented double-checked locking with `volatile`.

For example:

```java
private static volatile Singleton instance;

public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

`volatile` is important because it provides visibility and ordering guarantees for the shared reference.

---

## Q4. How can Singleton be broken?

**Answer:**  
A traditional Singleton can potentially be broken through:

- reflection;
- serialization/deserialization;
- cloning;
- multiple class loaders.

Common defenses include:

- enum Singleton;
- `readResolve()` for serialization;
- preventing cloning;
- guarding reflective construction where appropriate.

Also clarify the intended scope: a Spring singleton is one instance per Spring container, not necessarily globally across every JVM/container.

---

## Q5. Factory vs Abstract Factory?

**Answer:**  
A Factory Method typically focuses on creating a product from one product hierarchy. Abstract Factory creates a family of related products.

Example:

```text
Factory Method:
NotificationFactory → Email/SMS/Push

Abstract Factory:
UIFactory
   ├── WindowsFactory → WindowsButton + WindowsCheckbox
   └── MacFactory     → MacButton + MacCheckbox
```

---

## Q6. Factory vs Builder?

**Answer:**  

- **Factory:** decides which object/product to create.
- **Builder:** controls how a complex object is assembled.

For example, a factory can decide whether to create `CreditCardPayment` or `UpiPayment`, while a builder can construct a complex `OrderRequest` containing many optional fields.

---

## Q7. Strategy vs State?

**Answer:**  
Both often use composition and can have very similar class structures.

- **Strategy:** selects an algorithm or behavior, usually based on an external choice.
- **State:** changes behavior because the object's internal state changes.

Example:

```text
Strategy → CardPayment / UpiPayment
State    → CREATED / PAID / SHIPPED / DELIVERED
```

---

## Q8. Strategy vs Factory?

**Answer:**  
Factory and Strategy solve different problems.

- **Factory:** object creation.
- **Strategy:** behavior selection.

They can be combined:

```text
Factory
   ↓
creates/selects Strategy
   ↓
Strategy executes behavior
```

---

## Q9. Adapter vs Facade?

**Answer:**

- **Adapter:** changes an interface so incompatible components can work together.
- **Facade:** provides a simpler interface over a complex subsystem.

Example:

```text
Adapter:
Our PaymentProcessor → LegacyPaymentGateway

Facade:
OrderFacade → Inventory + Payment + Shipping + Notification
```

---

## Q10. Decorator vs Proxy?

**Answer:**

- **Decorator:** primarily adds or composes behavior.
- **Proxy:** primarily controls access to another object.

Java I/O wrappers are classic decorators. Spring AOP and transaction interception are common proxy examples.

---

## Q11. Composition vs inheritance?

**Answer:**  

Inheritance represents an **is-a** relationship:

```text
ElectricCar is a Car
```

Composition represents a **has-a** relationship:

```text
Car has an Engine
```

Composition is often preferred when behavior needs to be changed independently because it reduces rigid inheritance coupling and allows implementations to be substituted at runtime.

---

## Q12. How do patterns relate to SOLID?

**Answer:**  
Patterns can help implement SOLID principles, but they do not automatically guarantee them.

Examples:

- Strategy → Open/Closed Principle and Dependency Inversion.
- Adapter → Interface Segregation/Dependency Inversion in suitable designs.
- Facade → separation of responsibilities and reduced coupling.
- Factory → Dependency Inversion by hiding concrete creation.
- Decorator → Open/Closed Principle through composition.

The important point is to explain the **design problem**, not simply associate a pattern with a principle.

---

## Q13. How does Spring use Proxy?

**Answer:**  
Spring can create proxy objects around beans to intercept method calls and apply cross-cutting behavior.

Examples include:

- transactions;
- caching;
- asynchronous execution;
- Spring AOP.

Conceptually:

```text
Client
  ↓
Spring Proxy
  ↓
Target Bean
```

The proxy can execute additional behavior before/after the target method.

---

## Q14. How does `@Transactional` relate to proxies?

**Answer:**  
Spring commonly applies transaction behavior through a proxy. An external caller invokes the proxy, which starts/joins a transaction before invoking the target method.

Therefore, proxy boundaries matter.

A common issue is self-invocation:

```java
this.doSomethingTransactional();
```

That call normally bypasses the proxy, so the expected proxy-based interception may not occur.

---

## Q15. Why does self-invocation affect Spring AOP?

**Answer:**  
Because the call occurs directly on `this`, rather than passing through the Spring-managed proxy.

```text
External caller
      ↓
Spring proxy
      ↓
Target method
```

versus:

```text
Target method
      ↓
this.otherMethod()
      ↓
Target directly
```

For proxy-based AOP, the second path can bypass interception.

---

## Q16. Where does Spring use Chain of Responsibility?

**Answer:**  
A strong example is the Spring Security filter chain.

Conceptually:

```text
Request
 ↓
Authentication Filter
 ↓
Authorization Filter
 ↓
Other Security Filters
 ↓
Application
```

Each filter can inspect/process the request and either continue or stop the chain.

---

## Q17. When should you NOT use a design pattern?

**Answer:**  
Do not introduce a pattern merely to demonstrate pattern knowledge.

If a simple `if`, `switch`, or direct class is:

- easy to understand;
- unlikely to change;
- small;
- adequately testable;

then adding Strategy/Factory/etc. may create unnecessary abstraction.

The correct interview answer is:

> "I would first identify the variability and expected change. I would introduce the pattern when it reduces meaningful coupling or complexity."

---

## Q18. How would you replace a large `if/else` or `switch`?

**Answer:**  
First determine whether the conditions represent genuinely different behaviors.

If they do, possible approaches include:

- Strategy;
- State;
- Command;
- polymorphism;
- a registry/map of handlers.

For example:

```java
Map<String, PaymentStrategy> strategies;
```

can replace a growing payment-type switch.

However, a small fixed switch can remain the simpler design.

---

## Q19. Design a payment system with multiple providers.

**Answer:**  
Use a common abstraction:

```java
interface PaymentStrategy {
    void pay(BigDecimal amount);
}
```

Implement:

```text
CardPaymentStrategy
UpiPaymentStrategy
WalletPaymentStrategy
```

Use Spring dependency injection and a registry:

```text
PaymentService
     |
Map<PaymentType, PaymentStrategy>
```

Important production concerns:

- idempotency;
- payment timeout;
- provider failures;
- retries;
- duplicate callbacks;
- reconciliation;
- auditability.

---

## Q20. Design a notification system.

**Answer:**  
Use Strategy for channel-specific behavior:

```text
NotificationService
       |
NotificationStrategy
  /      |       \
Email    SMS     Push
```

If sending is asynchronous, add a message broker:

```text
API
 ↓
Notification Service
 ↓
Kafka/RabbitMQ
 ↓
Email/SMS/Push Consumers
```

Add retries, dead-letter handling, idempotency, rate limiting, and provider fallback where required.

---

# 50. Microservices Interview Questions — Questions + Answers

## Q1. What are the benefits and disadvantages of microservices?

**Answer:**

### Benefits

- independent deployment;
- independent scaling;
- team/service ownership;
- technology flexibility where justified;
- fault isolation when designed correctly;
- smaller bounded contexts.

### Costs

- distributed-system complexity;
- network failures;
- eventual consistency;
- observability complexity;
- deployment/platform overhead;
- distributed transactions;
- data synchronization.

A strong answer should explicitly say that microservices are not automatically better than a modular monolith.

---

## Q2. Monolith vs modular monolith vs microservices?

**Answer:**

| Architecture | Main characteristic |
|---|---|
| Monolith | One deployable application, often highly coupled |
| Modular monolith | One deployable unit with strong internal module boundaries |
| Microservices | Multiple independently deployable services |

A modular monolith can be a useful intermediate architecture when independent deployment/scaling is not yet required.

---

## Q3. How do you decide service boundaries?

**Answer:**  
Start with business capabilities and bounded contexts rather than technical layers.

Look for:

- cohesive business responsibility;
- independent data ownership;
- transaction boundaries;
- change patterns;
- team ownership;
- scaling requirements;
- independent deployment needs.

Avoid splitting a system into tiny CRUD services merely because individual tables exist.

---

## Q4. What is a bounded context?

**Answer:**  
A bounded context is a boundary within which a particular domain model and terminology have a defined meaning.

For example, "Customer" may mean different things in:

```text
Sales
Billing
Shipping
Support
```

A bounded context helps prevent unrelated domain models from being forced into one universal model.

---

## Q5. What makes a service independently deployable?

**Answer:**  
A service should ideally have:

- clear API/event contracts;
- owned code;
- controlled data ownership;
- minimal runtime coupling;
- backward-compatible interfaces where needed;
- independent build/deployment;
- independent operational health.

A service that cannot deploy without coordinating deployment with five other services is moving toward a distributed monolith.

---

## Q6. Why should each service own its data?

**Answer:**  
Data ownership reduces coupling and allows services to evolve their schemas independently.

```text
Order Service → Order DB
Payment Service → Payment DB
Inventory Service → Inventory DB
```

Cross-service data access should generally happen through APIs/events rather than direct database joins.

---

## Q7. When would you avoid microservices?

**Answer:**  
Avoid them when the organizational and operational complexity outweighs the benefits.

Examples:

- small team;
- simple domain;
- low scale;
- no independent deployment requirement;
- weak DevOps/observability capability;
- domain boundaries are not understood.

A well-structured monolith can be a better architecture for many systems.

---

## Q8. REST vs messaging?

**Answer:**

### REST/synchronous

Useful when the caller needs an immediate response.

```text
Order → Payment
```

### Messaging/asynchronous

Useful when work can happen independently and eventual consistency is acceptable.

```text
OrderCreated → Broker → Consumers
```

Messaging improves decoupling but introduces complexity around ordering, duplication, retries, consumer failures, and eventual consistency.

---

## Q9. Synchronous vs asynchronous communication?

**Answer:**

| Synchronous | Asynchronous |
|---|---|
| Immediate response | Response/work can happen later |
| Simpler request flow | Better decoupling |
| Caller waits | Caller can continue |
| Failure propagates directly | Failure can be isolated through messaging |
| More latency coupling | More operational complexity |

---

## Q10. When would you use Kafka/RabbitMQ?

**Answer:**  
Use a broker when you need asynchronous communication, buffering, decoupling, fan-out, event-driven processing, or durable work queues.

The exact choice depends on requirements such as:

- event streaming vs work queue;
- ordering;
- retention;
- replay;
- consumer model;
- operational ecosystem.

---

## Q11. What is eventual consistency?

**Answer:**  
After a successful update in one service, other services may temporarily have stale data, but the system is designed to converge toward a consistent state.

Example:

```text
Order Created
   ↓
Inventory event processing
   ↓
Payment event processing
```

The order, inventory, and payment views may not update at exactly the same instant.

---

## Q12. How do you handle duplicate messages?

**Answer:**  
Consumers should be designed to be idempotent.

Possible techniques:

- unique event/message IDs;
- idempotency tables;
- unique database constraints;
- checking processed-message records;
- state-transition validation.

Example:

```text
eventId = 12345

ProcessedEvents
---------------
12345
```

If event `12345` arrives again, the consumer can avoid applying the business operation twice.

---

## Q13. What is idempotency?

**Answer:**  
An operation is idempotent if repeating the same logical operation produces the same intended business result as executing it once.

For payments, an idempotency key can prevent duplicate charging when a client retries after a timeout.

---

## Q14. How do you handle message ordering?

**Answer:**  
First determine whether ordering is actually required.

If required, techniques can include:

- partitioning messages by business key;
- using ordered queues;
- sequence numbers;
- version checks.

For example, all events for one `orderId` can be routed to the same Kafka partition to preserve partition-level ordering.

---

## Q15. How do you handle poison messages?

**Answer:**  
A poison message repeatedly fails processing.

Typical approach:

```text
Consumer
   ↓
Retry
   ↓
Retry limit
   ↓
Dead Letter Queue
```

Then investigate/reprocess the message after correcting the underlying issue.

---

## Q16. Explain Circuit Breaker.

**Answer:**  
Circuit Breaker prevents repeated calls to an unhealthy dependency.

States:

```text
CLOSED → normal calls
OPEN → fail fast
HALF-OPEN → test recovery
```

It helps prevent cascading failures.

---

## Q17. Circuit Breaker vs Retry?

**Answer:**

- **Retry:** "Try the operation again."
- **Circuit Breaker:** "Stop calling this unhealthy dependency for a period."

Retries can increase load during an outage, so they should be bounded and combined with backoff. Circuit breakers can prevent repeated pressure on an unhealthy dependency.

---

## Q18. Why are timeouts important?

**Answer:**  
A remote call without a timeout can wait indefinitely and consume:

- threads;
- connections;
- memory;
- request slots.

Eventually the caller can become unhealthy too.

Timeouts establish a bounded failure behavior.

---

## Q19. Explain Bulkhead.

**Answer:**  
Bulkhead isolates resources so that failure in one dependency does not consume all resources of the service.

Example:

```text
Service
 ├── Payment connection/thread pool
 ├── Inventory connection/thread pool
 └── User connection/thread pool
```

If Payment becomes slow, it should not necessarily consume all capacity needed by Inventory.

---

## Q20. What is a retry storm?

**Answer:**  
A retry storm occurs when many clients simultaneously retry failed requests, increasing traffic against an already unhealthy dependency.

Mitigations:

- exponential backoff;
- jitter;
- retry limits;
- circuit breakers;
- rate limits;
- load shedding.

---

## Q21. Why use exponential backoff and jitter?

**Answer:**  
Exponential backoff increases the delay between retries:

```text
100ms
200ms
400ms
800ms
```

Jitter adds randomness so many clients do not retry at exactly the same moment.

This reduces synchronized retry spikes.

---

## Q22. How do you prevent cascading failures?

**Answer:**  
Use a combination of:

- timeouts;
- circuit breakers;
- bounded retries;
- exponential backoff + jitter;
- bulkheads;
- rate limiting;
- load shedding;
- asynchronous processing;
- graceful degradation;
- good observability.

Do not automatically add every mechanism; choose them according to the failure mode.

---

## Q23. Why can't we simply use `@Transactional` across microservices?

**Answer:**  
Spring's normal `@Transactional` transaction boundary is local to a transaction manager/resource context. A distributed business transaction spanning independent databases cannot generally be made into one simple local ACID transaction.

Possible approaches include:

- Saga;
- transactional outbox;
- carefully designed asynchronous workflows;
- distributed transaction protocols in special cases.

---

## Q24. Explain Saga.

**Answer:**  
Saga breaks a distributed business transaction into a sequence of local transactions.

Example:

```text
Create Order
    ↓
Reserve Inventory
    ↓
Charge Payment
    ↓
Confirm Order
```

If payment fails:

```text
Payment failure
      ↓
Release inventory
      ↓
Cancel order
```

The rollback is implemented through business-level compensating actions rather than one global database rollback.

---

## Q25. Choreography vs orchestration?

**Answer:**

### Choreography

Services react to events:

```text
OrderCreated
   ↓
Inventory
   ↓
InventoryReserved
   ↓
Payment
```

Advantages:

- decentralized;
- naturally event-driven.

Risks:

- workflow can become difficult to understand as the number of services grows.

### Orchestration

A central coordinator controls the workflow:

```text
Saga Orchestrator
 /      |       \
Order Inventory Payment
```

Advantages:

- explicit workflow;
- centralized coordination.

Risk:

- orchestrator can become overly complex or contain too much business logic.

---

## Q26. What is a compensating transaction?

**Answer:**  
A compensating transaction semantically reverses or offsets a previously completed business action.

Example:

```text
ReserveInventory()
```

compensation:

```text
ReleaseInventory()
```

It is not necessarily a database rollback; it is a business operation.

---

## Q27. Explain Transactional Outbox.

**Answer:**  
The service writes business data and an event record to the same local database transaction.

```text
DB transaction
 ├── Order row
 └── Outbox row
```

A separate publisher reads the outbox and publishes the event.

This avoids the dangerous gap:

```text
DB committed
Kafka publish failed
```

because the event intent is durably stored.

---

## Q28. Outbox vs 2PC?

**Answer:**

**Outbox:**

- local DB transaction;
- asynchronous event publication;
- eventual consistency;
- usually simpler operational model.

**2PC:**

- coordinates a distributed transaction;
- stronger atomicity semantics;
- higher coordination and operational complexity;
- often undesirable for loosely coupled microservices.

The choice depends on business consistency requirements and infrastructure.

---

## Q29. How do you achieve idempotent processing?

**Answer:**  
Use a stable business/event identifier and record processing state.

Example:

```text
PaymentEvent(eventId=ABC123)
        ↓
Check processed_events
        ↓
Not found → process + record ABC123
Found     → ignore/reconcile duplicate
```

Also use database uniqueness constraints where appropriate.

---

## Q30. Database-per-service?

**Answer:**  
Each service owns its data store/schema:

```text
Order → Order DB
Payment → Payment DB
Inventory → Inventory DB
```

Benefits:

- independent schema evolution;
- clear ownership;
- reduced coupling.

Cost:

- no simple cross-service SQL join;
- distributed queries become more complex;
- consistency often becomes eventual.

---

## Q31. Why avoid a shared database?

**Answer:**  
A shared DB creates coupling through:

- tables;
- schemas;
- database permissions;
- joins;
- deployment dependencies;
- implicit data contracts.

Two services may be independently deployed at the application layer but still tightly coupled through the database.

---

## Q32. When could a shared DB be pragmatic?

**Answer:**  
During incremental migration, legacy modernization, or when independent data ownership is not yet practical.

However, if used, establish clear ownership boundaries and migration plans rather than allowing every service to modify every table.

---

## Q33. What is CQRS?

**Answer:**  
CQRS separates command/write responsibilities from query/read responsibilities.

```text
Commands → Write Model
Queries  → Read Model
```

It can be useful when read and write workloads, models, scaling needs, or consistency requirements differ substantially.

CQRS does not inherently require separate databases.

---

## Q34. What is Event Sourcing?

**Answer:**  
Event Sourcing stores state-changing events as the authoritative record.

Instead of:

```text
Balance = 500
```

store:

```text
AccountCreated
Deposited(1000)
Withdrawn(300)
Withdrawn(200)
```

Current state can be reconstructed by replaying events.

Trade-offs include event-schema evolution, storage, replay cost, debugging complexity, and eventual consistency considerations.

---

## Q35. CQRS vs Event Sourcing?

**Answer:**

- **CQRS:** separates command and query responsibilities.
- **Event Sourcing:** stores state changes as events.

They can be used independently.

```text
CQRS without Event Sourcing → possible
Event Sourcing without CQRS → possible
CQRS + Event Sourcing       → common combination
```

---

## Q36. What is API Gateway?

**Answer:**  
An API Gateway is an entry point for external clients and can provide:

- routing;
- authentication/authorization;
- rate limiting;
- request transformation;
- aggregation;
- observability;
- TLS termination.

It should not become a giant business-logic monolith.

---

## Q37. What is BFF?

**Answer:**  
Backend for Frontend provides a backend tailored to a specific client type.

```text
Mobile → Mobile BFF → Services
Web    → Web BFF    → Services
```

It is useful when mobile, web, and other clients have substantially different data and interaction requirements.

---

## Q38. What is service discovery?

**Answer:**  
Service discovery allows services to locate available instances dynamically instead of hard-coding host/port information.

```text
Order Service
      ↓
Discovery / Service DNS
      ↓
Order Instance A/B/C
```

In Kubernetes, Service objects and cluster DNS commonly provide service discovery.

---

## Q39. What is a service mesh?

**Answer:**  
A service mesh provides infrastructure-level service-to-service communication capabilities such as:

- mTLS;
- traffic routing;
- telemetry;
- service identity;
- resilience features.

It can reduce duplicated networking code in applications.

---

## Q40. What is a sidecar?

**Answer:**  
A sidecar is a supporting process/container deployed alongside an application.

```text
Pod
 ├── Application
 └── Sidecar Proxy
```

The sidecar can provide proxying, telemetry, security, or other infrastructure capabilities.

---

## Q41. What is distributed tracing?

**Answer:**  
Distributed tracing tracks one logical request across multiple services.

Example:

```text
Trace ID: 123

Gateway
  └── Order Service
        └── Payment Service
              └── Fraud Service
```

A trace contains spans representing operations in individual components.

OpenTelemetry is a common standard/tooling ecosystem for instrumentation and context propagation.

---

## Q42. What are health checks?

**Answer:**  
Health checks expose whether a service is operational and/or ready for traffic.

Two common concepts:

- **Liveness:** should the process/container be restarted?
- **Readiness:** should the service receive traffic?

A service can be alive but not ready.

---

## Q43. Liveness vs readiness?

**Answer:**

**Liveness:**

> "Is the process healthy enough to keep running?"

**Readiness:**

> "Can this instance currently serve requests?"

For example, a newly started service may be alive but not ready until its initialization is complete.

---

## Q44. What is rate limiting?

**Answer:**  
Rate limiting restricts request volume over a defined period.

Example:

```text
Client → 100 requests/minute allowed
```

Common algorithms include:

- token bucket;
- leaky bucket;
- fixed window;
- sliding window.

Use cases:

- API protection;
- abuse prevention;
- downstream protection;
- fair resource allocation.

---

## Q45. What is load balancing?

**Answer:**  
Load balancing distributes requests among multiple service instances.

```text
              Load Balancer
             /      |      \
        Instance A B       C
```

Goals include:

- utilization;
- availability;
- horizontal scaling.

---

## Q46. How would you migrate a monolith?

**Answer:**  
Use incremental extraction rather than immediately rewriting everything.

A practical sequence:

```text
1. Understand domain boundaries
2. Identify a cohesive capability
3. Introduce an anti-corruption boundary if needed
4. Extract functionality
5. Route traffic selectively
6. Move data ownership carefully
7. Observe and stabilize
8. Repeat
```

The Strangler Fig pattern is a common migration approach.

---

## Q47. Explain Strangler Fig.

**Answer:**  
The new system gradually takes over functionality from the old system.

```text
Client
  |
Routing
 /     \
New     Legacy
service system
```

As functionality migrates, the legacy surface shrinks until it can be retired.

---

## Q48. What is an Anti-Corruption Layer?

**Answer:**  
An Anti-Corruption Layer protects a new domain model from a legacy system's model.

```text
New Domain
    |
ACL / Translator
    |
Legacy System
```

It translates:

- data structures;
- terminology;
- commands;
- responses.

This prevents legacy concepts from spreading throughout the new service.

---

## Q49. How do you identify service boundaries during migration?

**Answer:**  
Look at:

- business capabilities;
- bounded contexts;
- ownership;
- data boundaries;
- transaction boundaries;
- change frequency;
- scaling requirements;
- dependencies.

Avoid extracting services solely based on Java packages or database tables.

---

## Q50. How do you prevent a distributed monolith?

**Answer:**  
A distributed monolith occurs when services are technically separate but operationally and logically tightly coupled.

Prevent it by:

- defining clear ownership;
- reducing synchronous call chains;
- using asynchronous communication where appropriate;
- maintaining backward-compatible contracts;
- avoiding shared databases;
- allowing independent deployment;
- avoiding mandatory simultaneous releases;
- measuring service dependency graphs;
- keeping domain boundaries meaningful.

---

# 51. Scenario-Based Interview Questions + Model Answers

## Scenario 1: Payment service is down. What happens to Order Service?

**Answer:**  
First define the business requirement.

If payment must be synchronous:

```text
Order → Payment
          |
       timeout
          |
   circuit breaker
```

The order should move to an appropriate state such as `PAYMENT_PENDING` or failure, depending on the business workflow.

If payment can be asynchronous:

```text
OrderCreated
     ↓
Broker
     ↓
Payment Service
```

The order can remain pending while payment processing continues.

Do not blindly retry payment because payment operations require idempotency.

---

## Scenario 2: Kafka event is delivered twice. What do you do?

**Answer:**  
Assume duplicate delivery is possible.

Use:

```text
eventId
  ↓
processed-events check
  ↓
process only once
```

Also enforce business-level uniqueness where possible.

The key principle is:

> Consumers should be idempotent.

---

## Scenario 3: Database commit succeeds but Kafka publish fails.

**Answer:**  
Use Transactional Outbox.

```text
Single DB transaction:
    Order update
    Outbox event

Outbox Publisher
       ↓
     Kafka
```

The event remains durable even if the initial Kafka call fails.

---

## Scenario 4: Three downstream services are required for an API response.

**Answer:**  
Consider an Aggregator/BFF.

Call independent services in parallel where possible:

```text
             Aggregator
            /    |     \
         A       B       C
```

Use:

- independent timeouts;
- partial failure handling;
- circuit breakers where appropriate;
- response caching where appropriate.

Parallelization can reduce total latency, but the aggregator now owns additional failure-handling complexity.

---

## Scenario 5: One downstream dependency is consuming all threads.

**Answer:**  
Use a Bulkhead/concurrency isolation strategy.

```text
Service
 ├── Payment capacity
 ├── Inventory capacity
 └── User capacity
```

Also investigate:

- timeouts;
- circuit breaker;
- connection pool limits;
- queueing;
- backpressure.

---

## Scenario 6: The same customer requests payment twice because of a network timeout.

**Answer:**  
Use an idempotency key.

```text
POST /payments
Idempotency-Key: CUSTOMER-ORDER-123
```

The service stores the result associated with that key.

If the request is repeated, return the previously recorded result instead of creating another payment operation.

---

## Scenario 7: Inventory is reserved, but payment fails.

**Answer:**  
This is a Saga scenario.

```text
Reserve Inventory
       ↓
Payment fails
       ↓
Release Inventory
       ↓
Cancel/mark Order
```

The release is a compensating business transaction.

---

## Scenario 8: The web UI needs product, pricing, reviews, and inventory data.

**Answer:**  
An Aggregator or BFF can provide a client-oriented endpoint.

```text
Web
 |
Web BFF
 |--- Product
 |--- Pricing
 |--- Reviews
 └--- Inventory
```

This prevents the browser from knowing every internal service endpoint and lets the backend optimize aggregation.

---

## Scenario 9: A legacy customer system has a completely different data model.

**Answer:**  
Use an Anti-Corruption Layer/Adapter.

```text
New Customer Domain
       |
Translation Layer
       |
Legacy Customer API
```

Keep legacy concepts out of the new domain model.

---

## Scenario 10: You have 30 microservices and debugging a request is difficult.

**Answer:**  
Implement distributed tracing and structured logging.

Propagate:

```text
Trace ID
Span ID
Correlation context
```

Use OpenTelemetry-compatible instrumentation and a tracing backend.

Also ensure logs contain useful identifiers such as:

```text
traceId
spanId
service
operation
businessId
timestamp
```

---

# 52. Final Interview Checklist

Before a 5–6 YOE Java/Spring Boot interview, be able to answer each pattern using this structure:

```text
1. Definition
2. Problem it solves
3. When to use it
4. When NOT to use it
5. Java implementation
6. Spring Boot implementation
7. Real production example
8. Alternative approaches
9. Trade-offs
10. Testing implications
11. Failure/scalability implications
```

For microservices:

```text
1. Business problem
2. Service boundary
3. Data ownership
4. Communication style
5. Consistency model
6. Failure handling
7. Idempotency
8. Observability
9. Scaling
10. Operational trade-offs
```

## The key interview principle

> **Do not use a pattern because you know the pattern. Use it because you can identify the design problem it solves and explain the trade-offs it introduces.**

For a 5–6 year Java developer, interviewers generally get more value from hearing *why* you selected Strategy, Saga, Outbox, Circuit Breaker, or CQRS than from hearing a memorized definition.

