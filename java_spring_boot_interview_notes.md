# Java & Spring Boot Interview Notes

A practical interview revision guide covering every topic discussed,
with concise definitions, real-world context, and implementation
examples.

> **Scope:** The conversation covered Java, Java concurrency, Java
> Streams, and Spring Boot/Spring Cloud. No React topics were discussed,
> so this guide focuses on those technologies.

------------------------------------------------------------------------

## 1. Java 25 Features

**Java 25 is an LTS release**, released in September 2025. Important
features include:

-   **Compact source files and instance `main` methods:** Reduce
    boilerplate for simple programs and learning examples.
-   **Module import declarations:** Import packages exported by a module
    using a module import declaration.
-   **Flexible constructor bodies:** Permit certain statements before an
    explicit `super(...)` or `this(...)` invocation, while preserving
    constructor-safety restrictions.
-   **Primitive types in patterns, `instanceof`, and `switch`:** Extends
    pattern matching to primitive types.
-   **Scoped Values (final):** Provide a structured way to share
    immutable data across methods and threads.
-   **Structured Concurrency (preview in Java 25):** Treats related
    concurrent tasks as a single unit of work.
-   **Key Derivation Function (KDF) API:** Adds an API for deriving
    cryptographic keys.
-   **Generational Shenandoah:** Adds generational garbage collection to
    Shenandoah.
-   **Ahead-of-Time (AOT) improvements:** Includes features intended to
    improve startup and warm-up.

**Interview summary:** Java 25 focuses on reducing boilerplate,
improving concurrency and cryptographic APIs, and improving runtime
performance. Be clear that preview features may require preview flags
and are not equivalent to final features.

------------------------------------------------------------------------

## 2. Functional Interface

A **functional interface** has exactly one abstract method (SAM: Single
Abstract Method). It can have multiple `default` and `static` methods.
The `@FunctionalInterface` annotation is optional but recommended
because the compiler validates the contract.

``` java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);

    default void printInfo() {
        System.out.println("Calculator");
    }

    static Calculator create() {
        return (a, b) -> a + b;
    }
}

Calculator calculator = (a, b) -> a + b;
System.out.println(calculator.add(10, 20)); // 30
calculator.printInfo();
```

### Why are default and static methods allowed?

-   **Abstract method:** Must be implemented by a class or supplied by a
    lambda.
-   **Default method:** Has an implementation and is inherited by
    implementing classes; it may be overridden.
-   **Static method:** Belongs to the interface and is called using the
    interface name.
-   Default and static methods do not count toward the
    one-abstract-method requirement. Methods matching public `Object`
    methods also do not make the interface non-functional.

### Before Java 8 vs Java 8

Before Java 8, a common way to pass behavior was an anonymous inner
class:

``` java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Processing order");
    }
};
```

Java 8 lambda equivalent:

``` java
Runnable task = () -> System.out.println("Processing order");
```

**Real-world use:** Pass behavior to sorting, event handlers, background
tasks, and stream operations without creating a separate class for every
small action.

### Common built-in functional interfaces

  -----------------------------------------------------------------------
  Interface               Main method             Typical use
  ----------------------- ----------------------- -----------------------
  `Predicate<T>`          `boolean test(T)`       Check a condition

  `Function<T, R>`        `R apply(T)`            Transform a value

  `Consumer<T>`           `void accept(T)`        Perform an action

  `Supplier<T>`           `T get()`               Provide a value

  `Runnable`              `void run()`            Task with no result

  `Callable<T>`           `T call()`              Task that returns a
                                                  result and may throw
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. Java Thread Lifecycle

Java defines six states in `Thread.State`:

1.  **NEW** --- Created but not started.
2.  **RUNNABLE** --- Eligible to run or currently running; Java does not
    separate ready and running into distinct states.
3.  **BLOCKED** --- Waiting to acquire a monitor lock to enter/re-enter
    a `synchronized` block or method.
4.  **WAITING** --- Waiting indefinitely for another thread's action,
    such as through `join()` or `wait()`.
5.  **TIMED_WAITING** --- Waiting for a bounded time, such as with
    `sleep()` or timed `join()`.
6.  **TERMINATED** --- Execution has finished.

``` java
Thread thread = new Thread(() -> System.out.println("Working"));

System.out.println(thread.getState()); // NEW
thread.start();
// It may be RUNNABLE while executing.
thread.join();
System.out.println(thread.getState()); // TERMINATED
```

### Can a terminated thread be restarted?

**No.** Calling `start()` a second time on the same `Thread` object
throws `IllegalThreadStateException`. Create a new `Thread` object, or
preferably submit a task to an executor when appropriate.

**Real-world use:** Knowing the states helps diagnose threads waiting
for locks, blocked work, and tasks that have completed.

------------------------------------------------------------------------

## 4. `map()` vs `flatMap()` in Streams

-   **`map()`** transforms each input element into one output element.
-   **`flatMap()`** transforms each input into a stream/sequence and
    flattens the nested results into one stream.

``` java
List<String> names = List.of("john", "alex");

List<String> upper = names.stream()
        .map(String::toUpperCase)
        .toList();
// [JOHN, ALEX]

List<List<Integer>> nested = List.of(
        List.of(1, 2),
        List.of(3, 4)
);

List<Integer> flattened = nested.stream()
        .flatMap(List::stream)
        .toList();
// [1, 2, 3, 4]
```

**Real-world use:** `map()` can convert an `Order` to an order number.
`flatMap()` can combine all line items from a list of orders into one
list.

------------------------------------------------------------------------

## 5. Stream vs Collection

  -----------------------------------------------------------------------
  Collection                          Stream
  ----------------------------------- -----------------------------------
  Stores/manages data                 Processes data through a pipeline

  Can usually be traversed repeatedly A stream is consumed once

  Supports data-management            Does not itself store the source
  operations, depending on            data
  implementation                      

  Operations act on the collection    Intermediate operations are lazy
  directly                            until a terminal operation runs

  Examples: `List`, `Set`, `Map`      Created with methods such as
                                      `stream()`
  -----------------------------------------------------------------------

``` java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

List<Integer> evens = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toList();

System.out.println(evens); // [2, 4]
```

**Real-world use:** Keep employees in a collection, then use a stream to
filter employees by department or calculate totals. Stream operations
generally do not mutate the source collection unless the code explicitly
performs side effects.

------------------------------------------------------------------------

## 6. `Optional`

`Optional<T>` represents a value that may be present or absent. It is
useful for expressing a potentially missing return value and reducing
manual null checks.

### Common methods

-   `Optional.of(value)` --- Requires a non-null value; throws
    `NullPointerException` if passed `null`.
-   `Optional.ofNullable(value)` --- Converts a nullable value into a
    present or empty `Optional`.
-   `Optional.empty()` --- Creates an empty optional.
-   `isPresent()` / `isEmpty()` --- Checks presence/absence.
-   `ifPresent(action)` --- Runs an action when a value is present.
-   `map(mapper)` --- Transforms a present value.
-   `orElse(defaultValue)` --- Returns the value or the supplied
    default.
-   `orElseGet(supplier)` --- Computes a default lazily if empty.
-   `orElseThrow()` --- Returns the value or throws if empty.

``` java
Optional<String> a = Optional.of("Java");
Optional<String> b = Optional.ofNullable(null); // Optional.empty()

System.out.println(b.orElse("Default")); // Default
```

### Why use `of()` if `ofNullable()` is safer?

Use `of()` when `null` violates an invariant and should fail
immediately. Use `ofNullable()` when absence is expected and should be
handled. Neither method magically validates business data; choose based
on the contract.

**Real-world examples:** - `Optional.ofNullable(user.getNickname())`
when the nickname may be missing. - `Optional.of(requiredConfig)` when a
missing configuration value should be treated as a
programming/configuration error.

As a convention, `Optional` is most commonly used for return types.
Avoid automatically wrapping every field or method parameter in
`Optional`.

### Convert nested null checks to `Optional`

Suppose `User` has `getAddress()` and `Address` has `getCity()`:

``` java
public String getCity(User user) {
    return Optional.ofNullable(user)
            .map(User::getAddress)
            .map(Address::getCity)
            .orElse("Default Value");
}
```

Step-by-step: 1. `ofNullable(user)` becomes empty if `user` is null. 2.
The first `map` gets the address if the user exists. 3. The second `map`
gets the city if the address exists. 4. If any mapped result is null,
the optional becomes empty and later maps are skipped. 5. `orElse`
returns `"Default Value"` if no city is present.

If the city is `"Delhi"`, the method returns `"Delhi"`. If the user,
address, or city is null, it returns `"Default Value"`.

------------------------------------------------------------------------

## 7. Maximum Salary by Employee Grade Using Streams

Assume `Employee` has `getGrade()` returning `String` and `getSalary()`
returning `Integer`.

### A. Highest-paid employee per grade

``` java
Map<String, Optional<Employee>> highestPaidEmployeeByGrade =
        employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getGrade,
                        Collectors.maxBy(
                                Comparator.comparing(Employee::getSalary)
                        )
                ));
```

How it works: - `groupingBy(Employee::getGrade)` creates one group per
grade. - `Comparator.comparing(Employee::getSalary)` compares employees
by salary. - `maxBy(...)` selects the employee with the greatest salary
in each group. - The result uses `Optional<Employee>` because `maxBy`
returns an optional result.

### B. Maximum salary per grade as `Map<String, Integer>`

A direct solution is:

``` java
Map<String, Integer> maxSalaryByGrade =
        employees.stream()
                .collect(Collectors.toMap(
                        Employee::getGrade,
                        Employee::getSalary,
                        Integer::max
                ));
```

The merge function `Integer::max` resolves duplicate grade keys by
retaining the higher salary. This is concise and returns the required
grade-to-maximum-salary map.

Alternatively, use grouping and downstream reduction:

``` java
Map<String, Integer> maxSalaryByGrade =
        employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getGrade,
                        Collectors.reducing(
                                Integer.MIN_VALUE,
                                Employee::getSalary,
                                Integer::max
                        )
                ));
```

For ordinary non-negative salaries, `0` could be used as an identity
value, but `Integer.MIN_VALUE` is a more general identity for a max
reduction over integers. If the list is empty, no grade keys are
created.

**Real-world use:** Find the maximum salary within each job level or pay
band for a compensation review. Decide whether the requirement is to
return the employee record or only the numeric salary.

------------------------------------------------------------------------

## 8. `ExecutorService` for Multithreading

`ExecutorService` manages task execution and can reuse a pool of worker
threads. This avoids creating a new thread manually for every task.

``` java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

ExecutorService executor = Executors.newFixedThreadPool(3);

for (int i = 1; i <= 5; i++) {
    int taskId = i;
    executor.submit(() ->
            System.out.println("Task " + taskId + " on "
                    + Thread.currentThread().getName())
    );
}

executor.shutdown();
```

Key methods: - `execute(Runnable)` --- Runs a task without returning a
`Future`. - `submit(...)` --- Submits a task and returns a `Future`. -
`shutdown()` --- Rejects new tasks while allowing submitted tasks to
finish. - `shutdownNow()` --- Attempts to stop active tasks and returns
tasks that never started; it is best-effort, not a guarantee of
immediate termination.

**Real-world use:** Process independent report-generation or
image-resizing jobs using a bounded thread pool. Choose pool size based
on whether tasks are CPU-bound or I/O-bound and monitor queueing and
resource usage.

------------------------------------------------------------------------

## 9. `CompletableFuture`

`CompletableFuture` supports asynchronous computations, result
transformations, task composition, combining independent tasks, and
exception handling.

``` java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> "Hello")
                .thenApply(result -> result + " Java");

System.out.println(future.join()); // Hello Java
```

Common methods: - `supplyAsync()` --- Runs a task asynchronously and
returns a result. - `runAsync()` --- Runs a task asynchronously without
a result. - `thenApply()` --- Transforms a result synchronously in the
completion pipeline. - `thenCompose()` --- Chains a function that itself
returns a future; avoids nested futures. - `thenCombine()` --- Combines
results from two independent futures. - `thenAccept()` --- Consumes a
result without producing a new value. - `exceptionally()` --- Recovers
from an exception by supplying a replacement result. - `join()` ---
Waits for completion and throws an unchecked completion exception on
failure. - `get()` --- Waits for completion but uses checked exceptions.

By default, many async methods without an explicit executor use
`ForkJoinPool.commonPool()`. For blocking I/O or production workloads,
consider supplying a properly sized custom executor.

### Real-world example: fetch independent data concurrently

``` java
ExecutorService ioPool = Executors.newFixedThreadPool(10);

CompletableFuture<User> userFuture =
        CompletableFuture.supplyAsync(
                () -> userService.getUser(42L), ioPool);

CompletableFuture<List<Order>> ordersFuture =
        CompletableFuture.supplyAsync(
                () -> orderService.getOrders(42L), ioPool);

CompletableFuture<UserDashboard> dashboardFuture =
        userFuture.thenCombine(
                ordersFuture,
                UserDashboard::new
        );

UserDashboard dashboard = dashboardFuture.join();
ioPool.shutdown();
```

This example assumes the service methods and `UserDashboard` constructor
exist. In production, add timeouts, exception handling, and
lifecycle-managed executors.

------------------------------------------------------------------------

## 10. `ExecutorService` vs `CompletableFuture`

  -----------------------------------------------------------------------
  `ExecutorService`                   `CompletableFuture`
  ----------------------------------- -----------------------------------
  Manages thread/task execution and   Represents a result that may
  pool lifecycle                      complete later

  Submits work with `submit()` or     Builds asynchronous pipelines with
  `execute()`                         `supplyAsync()` and continuation
                                      methods

  `Future.get()` retrieves a result   Supports transformations,
  and blocks                          composition, and combining results

  Owns shutdown/lifecycle concerns    Can use an executor, including an
  for its executor                    `ExecutorService`

  Useful for controlling worker       Useful for expressing dependent or
  resources                           independent async workflows
  -----------------------------------------------------------------------

**Interview summary:** `ExecutorService` manages execution resources;
`CompletableFuture` models and composes asynchronous results. They
complement each other rather than replace each other.

------------------------------------------------------------------------

## 11. Spring Boot Version

Spring Boot **4.0.x** is the latest major generation as of this guide's
date (October 2026); Spring Boot **3.5.x** remains a relevant line for
many applications. Check the official release documentation for the
latest patch version and support status before choosing a production
version.

-   Spring Boot 3.x requires Java 17 or later.
-   Spring Boot 4.x requires Java 17 or later.
-   Choose a Spring Boot version that is compatible with your Java
    version, Spring Cloud release train, and dependencies.

**Interview tip:** State both the major/minor version used in your
project and the Java version. For production systems, use a supported
release line and a compatible Spring Cloud BOM.

------------------------------------------------------------------------

## 12. Calling Another REST API in Spring Boot

Common options: - **`RestClient`** --- Synchronous, fluent HTTP client;
available since Spring Framework 6.1 / Spring Boot 3.2. -
**`WebClient`** --- Supports reactive, non-blocking HTTP
communication. - **`RestTemplate`** --- Older synchronous client;
maintenance mode. - **OpenFeign** --- Declarative HTTP client, commonly
used with Spring Cloud.

### `RestClient` example

``` java
RestClient restClient = RestClient.create();

String response = restClient.get()
        .uri("https://api.example.com/users")
        .retrieve()
        .body(String.class);
```

**Real-world use:** A checkout service may call an inventory service to
check stock. In a real application, configure timeouts, authentication,
error handling, observability, and resilience.

------------------------------------------------------------------------

## 13. OpenFeign Client

OpenFeign lets you describe a remote HTTP API as a Java interface.
Spring Cloud creates the implementation.

### 1. Add the dependency

Use the Spring Cloud BOM compatible with your Spring Boot version, then
include:

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

Do not choose an arbitrary Spring Cloud version; use the compatibility
table for your Spring Boot release.

### 2. Enable Feign scanning

``` java
@SpringBootApplication
@EnableFeignClients
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 3. Declare the client

``` java
@FeignClient(name = "userClient", url = "${services.user.url}")
public interface UserClient {

    @GetMapping("/users/{id}")
    User getUser(@PathVariable("id") Long id);
}
```

Example configuration:

``` properties
services.user.url=https://api.example.com
```

### 4. Inject and call it

``` java
@Service
public class UserService {
    private final UserClient userClient;

    public UserService(UserClient userClient) {
        this.userClient = userClient;
    }

    public User getUser(Long id) {
        return userClient.getUser(id);
    }
}
```

**Real-world use:** An order service can call a customer service through
a typed interface instead of manually building HTTP requests. Configure
timeouts, error decoding, retries (carefully), and tracing as
appropriate.

------------------------------------------------------------------------

## 14. Feign Headers

### A. Header specific to one call

Use `@RequestHeader` when the value is part of that call's inputs:

``` java
@FeignClient(name = "userClient", url = "${services.user.url}")
public interface UserClient {

    @GetMapping("/users/{id}")
    User getUser(
            @PathVariable("id") Long id,
            @RequestHeader("Authorization") String authorization
    );
}
```

Call it with:

``` java
User user = userClient.getUser(1L, "Bearer " + token);
```

### B. Common header added without a method parameter

Use a `RequestInterceptor`:

``` java
@Configuration
public class FeignConfig {

    @Bean
    public RequestInterceptor authInterceptor() {
        return template ->
                template.header("X-Client-Name", "order-service");
    }
}
```

Attach the configuration:

``` java
@FeignClient(
    name = "userClient",
    url = "${services.user.url}",
    configuration = FeignConfig.class
)
public interface UserClient {
    @GetMapping("/users")
    List<User> getUsers();
}
```

### How does the header get added if it is not a method parameter?

1.  The application calls `getUsers()`.
2.  The generated Feign client builds a request template.
3.  Feign runs registered request interceptors.
4.  The interceptor adds the header to the outgoing request.
5.  Feign executes the HTTP request.

**Important:** Do not hardcode real secrets in source code. Load tokens
and API keys from a secure secret store or managed configuration. A
client-specific Feign configuration should be scoped correctly so that
its interceptor does not unintentionally affect every Feign client.

------------------------------------------------------------------------

## 15. Spring Boot Actuator

Actuator adds operational endpoints for health, metrics, application
information, and other diagnostics.

### Add dependency

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Common endpoints

  -----------------------------------------------------------------------
  Endpoint                            Purpose
  ----------------------------------- -----------------------------------
  `/actuator/health`                  Health status

  `/actuator/info`                    Application information

  `/actuator/metrics`                 Application and JVM metrics

  `/actuator/env`                     Environment properties; sensitive

  `/actuator/beans`                   Registered Spring beans

  `/actuator/prometheus`              Prometheus-format metrics when the
                                      relevant registry dependency is
                                      configured
  -----------------------------------------------------------------------

Most endpoints are not all exposed over HTTP by default. Endpoint
availability also depends on whether the endpoint is enabled and whether
the required dependencies/configuration are present.

### Expose selected endpoints

``` properties
management.endpoints.web.exposure.include=health,info,metrics
```

Then check:

`http://localhost:8080/actuator/health`

**Real-world use:** A container orchestrator can query the health
endpoint, while monitoring software collects metrics to detect rising
memory usage, request latency, or errors.

### Default port

By default, Actuator endpoints use the **same port as the application**,
normally `8080` if no other port is configured.

### Run Actuator on a separate port

``` properties
server.port=8080
management.server.port=8081
```

The application listens on port `8080`, while the management server
listens on `8081`. For example:

`http://localhost:8081/actuator/health`

A separate management port is useful for separating operational traffic
from application traffic, but network access and authentication still
need to be configured securely.

### Enable or disable endpoints

``` properties
# Enable the health endpoint
management.endpoint.health.enabled=true

# Disable the env endpoint
management.endpoint.env.enabled=false

# Expose selected endpoints over HTTP
management.endpoints.web.exposure.include=health,info,metrics

# Exclude an endpoint from HTTP exposure
management.endpoints.web.exposure.exclude=env,beans
```

These properties have different purposes:

-   `management.endpoint.<id>.enabled` controls whether the endpoint is
    enabled.
-   `management.endpoints.web.exposure.include` controls which enabled
    endpoints are exposed over HTTP.
-   `management.endpoints.web.exposure.exclude` removes endpoints from
    HTTP exposure.

Setting `enabled=false` is not the same as merely hiding an endpoint
from HTTP. Do not expose all endpoints (`*`) casually, especially `env`,
`beans`, or other diagnostic endpoints. Secure management endpoints
using Spring Security and/or network controls.

------------------------------------------------------------------------

## 16. Quick Interview Revision

-   **Functional interface:** Exactly one abstract method; supports
    lambdas.
-   **Default method:** An implemented interface method; does not count
    toward the SAM limit.
-   **Thread states:** `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`,
    `TIMED_WAITING`, `TERMINATED`.
-   **Restart terminated thread?** No; create a new thread or resubmit a
    task.
-   **`map()` vs `flatMap()`:** Transform each element vs transform and
    flatten nested sequences.
-   **Collection vs Stream:** Stores data vs processes data.
-   **`Optional.of()` vs `ofNullable()`:** Rejects null vs converts null
    to empty.
-   **`ExecutorService`:** Manages task execution and thread-pool
    lifecycle.
-   **`CompletableFuture`:** Composes asynchronous results.
-   **`RestClient` vs Feign:** Imperative HTTP client API vs declarative
    interface-based client.
-   **Feign headers:** `@RequestHeader` for per-call values;
    `RequestInterceptor` for common request headers.
-   **Actuator:** Production monitoring and management endpoints.
-   **Actuator port:** Same as application by default; set
    `management.server.port` for a separate port.
-   **Actuator endpoint control:** Endpoint enablement and HTTP exposure
    are separate settings.

## Practical checklist for production code

1.  Select a supported Java and Spring Boot version and compatible
    Spring Cloud release.
2.  Set connection/read timeouts for outbound HTTP calls.
3.  Handle non-2xx responses and exceptions explicitly.
4.  Avoid hardcoded secrets and avoid logging authorization headers.
5.  Use bounded executors for workloads that need explicit resource
    control.
6.  Add timeouts and exception recovery to asynchronous pipelines.
7.  Expose only necessary Actuator endpoints and protect sensitive
    management data.
8.  Write tests for null inputs, empty collections, duplicate grades,
    API failures, and concurrency edge cases.
