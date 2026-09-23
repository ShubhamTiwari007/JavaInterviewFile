# Java / Spring Boot + React — Interview Quick Revision

## 1. Controller Layer
- Entry point for HTTP requests in Spring Boot.
- Flow: **Client → Controller → Service → Repository → DB**
- Common annotations: `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestParam`, `@RequestBody`.
- Keep business logic in the service layer.

## 2. Containerization / Docker
- Packages an application and its dependencies into a portable container.
- **Image** = packaged application; **Container** = running instance.
- Common Dockerfile instructions: `FROM`, `WORKDIR`, `COPY`, `EXPOSE`, `ENTRYPOINT`.
- Containers are lighter and faster to start than VMs.

## 3. EC2, ECS, Lambda
- **EC2:** Virtual server; you manage the OS and application environment.
- **ECS:** AWS service for running Docker containers; can use EC2 or Fargate.
- **Lambda:** Serverless, event-driven function execution.
- Quick distinction: **EC2 = server, ECS = containers, Lambda = functions.**

## 4. Autowiring
- Spring automatically injects dependencies.
- Prefer constructor injection.
- Use `@Qualifier` when multiple beans of the same type exist.

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

## 5. Spring Boot Profiles
- Used for environment-specific configuration.
- Typical profiles: `dev`, `test`, `qa`, `prod`.
- Files: `application-dev.yml`, `application-prod.yml`.
- Activate with `spring.profiles.active=dev`.
- `@Profile("dev")` can conditionally create beans.

## 6. Constructor Injection vs Field Injection
Constructor injection is preferred because it:
- Makes dependencies explicit.
- Supports `final` fields and immutability.
- Improves unit testing.
- Ensures required dependencies exist during object construction.
- Makes circular dependencies easier to identify.

## 7. Design Patterns
Important Java/Spring patterns:
- **Singleton:** One instance; Spring beans are singleton-scoped by default.
- **Factory:** Encapsulates object creation.
- **Strategy:** Allows interchangeable algorithms/behaviors.
- **Observer:** Notifies dependent objects of changes/events.
- **Builder:** Helps construct complex objects.

## 8. Java Stream API
- Functional-style processing of collections.
- Common intermediate operations: `filter`, `map`, `sorted`, `distinct`, `limit`.
- Common terminal operations: `toList`, `collect`, `forEach`, `reduce`, `count`, `findFirst`.
- Streams are generally lazy until a terminal operation executes.

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .map(String::toUpperCase)
    .toList();
```

## 9. Multithreading / Parallel Processing
- Use `ExecutorService` for managed thread pools.
- `CompletableFuture` is useful for asynchronous workflows.
- `parallelStream()` can parallelize collection processing.
- Consider thread safety, race conditions, deadlocks, shared state, and thread-pool sizing.
- Don't assume `parallelStream()` is always faster.

## 10. Spring Security
Handles:
- Authentication — **Who are you?**
- Authorization — **What can you access?**
- JWT, OAuth2, roles, password encoding, CSRF, etc.

Typical REST flow:
**React → JWT → Spring Security → Controller**

## 11. Advanced Microservices
Key concepts:
- Service discovery
- API Gateway
- Circuit breaker / Resilience4j
- Distributed tracing / OpenTelemetry
- Centralized configuration
- REST / gRPC
- Kafka / RabbitMQ / SQS
- Saga pattern
- CQRS
- Idempotency
- Eventual consistency
- Retry / timeout
- Rate limiting
- Bulkhead pattern
- Observability
- Database-per-service

## 12. Kafka
Distributed event streaming platform.

Basic flow:
**Producer → Topic → Consumer**

Key concepts:
- Topic
- Partition
- Offset
- Producer
- Consumer
- Consumer group
- Broker
- Replication
- Retention
- Acknowledgments

Important:
- Ordering is guaranteed within a partition.
- Within a consumer group, a partition is consumed by only one consumer at a time.

## 13. Redux
State management for React applications.

Flow:
**Component → dispatch → Action → Reducer → Store → Component**

- Prefer **Redux Toolkit (RTK)** for modern Redux.
- Useful for shared/complex application state.
- `useState` may be sufficient for simple local state.

## 14. HOC — Higher Order Component
- Function that takes a component and returns an enhanced component.

```javascript
function withAuth(Component) {
    return function AuthenticatedComponent(props) {
        return isLoggedIn()
            ? <Component {...props} />
            : <Login />;
    };
}
```

- Common uses: authentication, authorization, logging.
- Many modern HOC use cases can be handled with custom hooks.

## 15. Alternative to `componentWillUnmount`
For functional components, use `useEffect` cleanup:

```javascript
useEffect(() => {
    const subscription = subscribe();

    return () => {
        subscription.unsubscribe();
    };
}, []);
```

- Class component: `componentWillUnmount()`
- Functional component: cleanup function returned by `useEffect`.

## 16. Enable Button When Form Fields Are Entered
Use controlled inputs and derive validity:

```javascript
const isValid =
    form.name.trim() !== "" &&
    form.email.trim() !== "";

<button disabled={!isValid}>Submit</button>
```

For production, also validate formats, lengths, required fields, and server-side constraints.

## 17. Webpack
- Module bundler.
- Processes JavaScript and other assets into optimized bundles.
- Key concepts: **entry, output, loaders, plugins, code splitting, tree shaking, minification**.
- Modern React applications may use Vite instead.

## 18. React Transpiling to JavaScript
JSX must be transformed into JavaScript that can execute in the browser.

Conceptually:
**JSX + modern JS → Babel/JSX transform → JavaScript**

Distinction:
- **Transpiling:** Converts source syntax.
- **Bundling:** Combines modules/assets.
- **Minification:** Reduces file size.

## 19. Testing in React
Common tools:
- **React Testing Library**
- **Jest**
- **Vitest**
- **Cypress**
- **Playwright**

Testing levels:
- **Unit:** Small isolated logic.
- **Component/Integration:** Components and interactions.
- **E2E:** Complete user flows.

Best practice:
> Test what the user sees and does rather than implementation details.

---

## Rapid-Fire Interview Answers

| Topic | One-line answer |
|---|---|
| Controller | Handles HTTP requests and delegates business logic |
| Docker | Packages application + dependencies into containers |
| EC2 | AWS virtual server |
| ECS | AWS container orchestration service |
| Lambda | Serverless event-driven compute |
| Autowiring | Spring dependency injection |
| Profiles | Environment-specific configuration |
| Constructor Injection | Explicit, testable, immutable dependencies |
| Stream API | Functional collection processing |
| Multithreading | Concurrent execution using multiple threads |
| Spring Security | Authentication and authorization |
| Microservices | Distributed services with resilience and observability |
| Kafka | Distributed event streaming platform |
| Redux | Predictable shared state management |
| HOC | Function that enhances a component |
| Unmount cleanup | `useEffect` cleanup function |
| Form button | Derive `disabled` from form validity |
| Webpack | JavaScript/module bundler |
| JSX transpilation | JSX transformed into JavaScript |
| React Testing | Test behavior from the user's perspective |
