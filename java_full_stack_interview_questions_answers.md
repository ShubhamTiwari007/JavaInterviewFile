# Java Full Stack Developer Interview Questions & Answers

## Tailored to the supplied JD and Shubham Tiwari's resume

**Target role:** Java Full Stack Developer (5+ years)\
**Core stack:** React, Java 8, Spring Boot, Microservices, JUnit/TDD,
SQL, AWS, Claude\
**Preparation note:** This guide consolidates the questions and answers
from the interview-preparation response. Resume-specific answers are
grounded in the supplied resume. Technologies not explicitly
demonstrated in the resume are marked as topics to prepare rather than
claimed as hands-on experience.

------------------------------------------------------------------------

## 1. Resume alignment and preparation priorities

  -----------------------------------------------------------------------
  JD requirement          Resume alignment        Priority
  ----------------------- ----------------------- -----------------------
  Java 8                  Direct experience       Very high

  Spring Boot             Direct experience       Very high

  Microservices           Direct experience       Very high

  React                   Direct experience       Very high

  JUnit / Mockito / TDD   JUnit listed; Mockito   Very high
                          not listed              

  SQL / PostgreSQL / DB2  SQL experience;         Very high
                          DB2/PostgreSQL not      
                          established             

  AWS                     S3, Lambda, SQS,        Very high
                          Secrets Manager         

  ECS, EC2, IAM, SNS, API Not explicitly          High
  Gateway                 established             

  DynamoDB, CloudWatch,   Not explicitly          High
  SSM, CloudFormation     established             

  Claude                  Claude Code and         Very high
                          Anthropic API           
                          experience              

  Kafka / event-driven    Direct experience       Very high
  architecture                                    

  Docker / Kubernetes     Direct experience       High

  Jenkins / CI/CD         Jenkins listed          High

  JavaScript              Direct experience       High

  Agile / JIRA / Sonar /  Not explicitly          Medium
  Splunk / Kong           established             
  -----------------------------------------------------------------------

### Resume-specific preparation

The strongest project discussion areas are:

-   Infosys: Spring Boot microservices, React, external API
    integrations, Java/SQL reporting automation, Claude Code, Anthropic
    API, Kubernetes-based scheduled jobs.
-   GlobalLogic: React, Keycloak authentication, React Flow, APIs,
    Apache Kafka, ECharts.
-   Capgemini: Java 8 standards, SQL optimization, EhCache, API
    performance improvements.

Do not claim implementation details, AWS services, databases, or tools
that you have not personally used. Be ready to distinguish direct
experience from conceptual knowledge.

------------------------------------------------------------------------

# 2. Resume-based and project deep-dive questions

### Q1. Tell me about yourself.

**Answer:**

"I am a Java Full Stack Developer with over five years of experience
building enterprise applications using Java, Spring Boot, React, and
cloud technologies.

Currently, I am working at Infosys as a Senior Associate Consultant,
where I develop full-stack applications using Spring Boot microservices
and React. My responsibilities include integrating external APIs,
automating business processes using Java and SQL, and improving
application reliability.

One of my recent contributions involved using Claude Code and GitHub
Copilot for code generation, unit test generation, and debugging. I also
worked on integrating Anthropic's API into a Kubernetes-based job
execution workflow to analyze repeated failures and assist with
remediation.

Previously, at GlobalLogic, I worked on React-based workflow
applications, Keycloak authentication, API integrations, and Apache
Kafka. At Capgemini, I focused on Java 8 upgrades, SQL optimization, API
performance improvements, and caching using EhCache.

My strongest areas are Java, Spring Boot, Microservices, React, SQL, and
building reliable cloud-native applications. I am now looking for an
opportunity where I can contribute to scalable distributed systems and
modern full-stack architectures."

### Q2. Explain your current project architecture at Infosys.

**Answer:**

"My current application follows a full-stack architecture:

-   Frontend: React handles user interactions, data presentation, and
    API communication.
-   Backend: Spring Boot microservices expose REST APIs and implement
    business logic.
-   Database: SQL-based persistence is used for business data.
-   Integrations: External services such as Google Ads API and Google
    Merchant Center are integrated to automate product-feed management.
-   Background processing: Scheduled jobs automate reporting and other
    backend activities.
-   Cloud and infrastructure: We use the infrastructure and services
    applicable to the project; I would describe only the components I
    have personally worked with.

The application manages digital content and product information and
publishes it to external marketplaces. We maintain separation between
frontend responsibilities, backend business logic, integrations, and
asynchronous processing."

### Q3. Explain the self-healing cron job you implemented using Claude.

**Answer:**

"We had a requirement to improve the reliability of scheduled jobs
running inside Kubernetes.

I worked on a solution that uses Anthropic's API to introduce
AI-assisted failure analysis.

The workflow is:

1.  A scheduled job executes inside a Kubernetes pod.
2.  If execution fails, the system attempts to re-trigger the job within
    the same pod.
3.  If the failure persists, the failure details and relevant context
    are sent to Claude through Anthropic's API.
4.  Claude analyzes the potential cause of the failure.
5.  The system attempts to generate a potential code fix.
6.  A pull request is raised for developer review.

The key objective was to reduce manual investigation and accelerate
issue resolution.

An important design principle is that AI-generated changes should not
automatically be trusted in production. The generated fix must be
validated, tested, reviewed, and approved before deployment."

**Likely follow-ups and answer points:**

-   **How did you send information to Claude?** Explain the actual
    Anthropic API integration, request format, authentication, and
    response handling you implemented.
-   **How did you construct the prompt?** Include the error, relevant
    logs, code context, constraints, and expected output format.
-   **How did you prevent sensitive information from being sent?**
    Redact secrets and personal or confidential data; apply
    data-minimization controls.
-   **What happens if Claude returns an incorrect fix?** Validate it
    with tests and code review; do not deploy unverified changes.
-   **How do you validate generated code?** Compile, run
    unit/integration tests, run static analysis, and require review.
-   **How do you prevent infinite retry loops?** Use bounded retries,
    backoff, attempt limits, and a terminal failure state.
-   **How do you handle API timeouts?** Use timeouts, bounded retries,
    and graceful failure handling.
-   **How do you maintain auditability?** Record job IDs, attempts,
    model request metadata, generated suggestions, test outcomes, and PR
    references without logging secrets.
-   **How do you ensure a generated PR cannot directly affect
    production?** Require protected branches, reviews, CI checks, and
    deployment approvals.

Be precise about what your implementation actually automated. Do not
claim automatic production deployment if it only raises a pull request.

### Q4. What is Claude Code, and how have you used it?

**Answer:**

"Claude Code is an AI-assisted coding tool that can understand a
codebase, help modify source files, generate tests, investigate errors,
and assist with development workflows.

I have used it for understanding existing code, generating JUnit test
cases, identifying potential bugs, suggesting code improvements,
debugging complex issues, and accelerating repetitive development tasks.

I treat its output as a development aid rather than authoritative code.
I review generated changes, execute tests, and validate behavior before
accepting them."

### Q5. What is the difference between Claude Code and Anthropic API?

  -----------------------------------------------------------------------
  Claude Code                         Anthropic API
  ----------------------------------- -----------------------------------
  Developer-facing coding tool        Programmatic API

  Helps developers work with          Allows applications to interact
  codebases                           with Claude models

  Used for coding and debugging       Used to build AI-powered
  assistance                          application features

  Works within a developer workflow   Integrated into backend services
  -----------------------------------------------------------------------

### Q6. Explain your Kafka experience at GlobalLogic.

**Answer:**

"Apache Kafka is a distributed event-streaming platform. It enables
services to communicate asynchronously through events.

In an event-driven architecture, a producer publishes an event to a
Kafka topic, and consumers subscribe to that topic and process the
event.

The advantage is that producers and consumers do not need to communicate
synchronously. This improves scalability and reduces tight coupling."

Prepare your exact role: whether you developed producers, consumers,
topic configuration, serialization, offset management, or error
handling.

### Q7. How did you improve API performance at Capgemini?

**Answer:**

"I worked on improving API response times through caching and SQL
optimization.

We used EhCache to cache frequently accessed data and reduce unnecessary
database calls.

I also analyzed SQL queries and optimized them by minimizing unnecessary
joins and retrieving only the required data.

The objective was to reduce database execution time and improve overall
API responsiveness."

**How would you measure the improvement?** Compare execution plans,
query execution time, database load, API latency, throughput, and cache
hit ratio before and after the change.

### Q8. Explain how you implemented authentication using Keycloak.

**Answer:**

"Keycloak is an identity and access management solution that supports
authentication and authorization.

In our application, we integrated Keycloak to manage user authentication
and role-based access.

Users authenticate through Keycloak, and the application uses the issued
tokens to establish identity and authorize access to protected
resources.

I also worked on customizing the login experience through Keycloak
themes, including custom login pages and email templates."

Prepare OAuth 2.0, OpenID Connect, JWT, access tokens, refresh tokens,
and role-based access control.

------------------------------------------------------------------------

# 3. Core Java 8 and advanced Java

### Q9. What are the major features introduced in Java 8?

**Answer:**

1.  Lambda expressions.
2.  Functional interfaces.
3.  Stream API.
4.  Method references.
5.  Default and static methods in interfaces.
6.  Optional class.
7.  New Date and Time API.
8.  CompletableFuture improvements.
9.  Collectors.

Java 8 introduced functional programming capabilities that make
collection processing and asynchronous programming more expressive.

### Q10. What is a functional interface?

**Answer:**

A functional interface contains exactly one abstract method. It can also
contain multiple default and static methods.

``` java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator calculator = (a, b) -> a + b;
System.out.println(calculator.add(10, 20));
```

Common built-in functional interfaces:

  Interface    Main method   Purpose
  ------------ ------------- --------------------
  Predicate    test()        Returns boolean
  Function     apply()       Transforms input
  Consumer     accept()      Consumes input
  Supplier     get()         Supplies a value
  BiFunction   apply()       Accepts two inputs

### Q11. What is a lambda expression?

**Answer:**

A lambda expression is an anonymous function that can be passed as an
argument or used to implement a functional interface.

Syntax:

``` java
(parameters) -> expression
```

Example:

``` java
List<String> names = Arrays.asList("Shubham", "Amit", "Rahul");
names.forEach(name -> System.out.println(name));
```

Benefits include less boilerplate code, functional programming support,
and easier collection processing.

### Q12. What is the Stream API?

**Answer:**

The Stream API processes collections of data in a declarative manner.

``` java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

List<Integer> result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());

// [4, 16, 36]
```

Important operations:

-   `filter()` -- filters elements.
-   `map()` -- transforms elements.
-   `flatMap()` -- flattens nested structures.
-   `sorted()` -- sorts elements.
-   `distinct()` -- removes duplicates.
-   `reduce()` -- combines elements.
-   `collect()` -- collects results.
-   `forEach()` -- performs an action.

### Q13. Difference between map() and flatMap()?

**Answer:**

`map()` transforms each element into another element. `flatMap()`
transforms each element into a stream and flattens the resulting
streams.

``` java
List<List<Integer>> numbers = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);

List<Integer> result = numbers.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());

// [1, 2, 3, 4]
```

### Q14. Difference between intermediate and terminal operations?

**Answer:**

Intermediate operations return another Stream and are generally lazy.
Examples: `filter()`, `map()`, `sorted()`.

Terminal operations trigger stream processing and produce a result or
side effect. Examples: `collect()`, `count()`, `forEach()`, `reduce()`.

### Q15. What is Optional in Java 8?

**Answer:**

Optional is a container object that may or may not contain a non-null
value. It makes the absence of a value explicit and can reduce
accidental NullPointerException occurrences.

``` java
Optional<String> name = Optional.ofNullable(getName());
String result = name.orElse("Unknown");
```

Important methods: `of()`, `ofNullable()`, `empty()`, `isPresent()`,
`ifPresent()`, `orElse()`, `orElseGet()`, `orElseThrow()`, `map()`, and
`flatMap()`.

**orElse() vs orElseGet():** `orElse()` evaluates its fallback argument
eagerly; `orElseGet()` evaluates its supplier only when the Optional is
empty.

### Q16. Difference between HashMap and ConcurrentHashMap?

  HashMap                           ConcurrentHashMap
  --------------------------------- -------------------------------
  Not thread-safe                   Supports concurrent access
  Allows one null key               Does not allow null keys
  Allows null values                Does not allow null values
  No concurrency guarantees         Provides concurrency controls
  Suitable for non-concurrent use   Suitable for concurrent use

### Q17. How does HashMap work internally?

**Answer:**

HashMap stores key-value pairs in an internal array of buckets.

1.  It calculates the hash code of the key.
2.  It uses the hash to determine the bucket index.
3.  It stores the key-value pair in the bucket.
4.  If multiple keys map to the same bucket, a collision occurs.
5.  Collisions are handled using linked nodes and, under suitable
    conditions, balanced trees.

Average lookup complexity is O(1), assuming a reasonably distributed
hash function.

### Q18. Difference between == and equals()?

**Answer:**

`==` compares primitive values or object references. `equals()` compares
logical equality when appropriately overridden.

``` java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

### Q19. Explain the contract between equals() and hashCode().

**Answer:**

If two objects are equal according to `equals()`, they must return the
same hash code. Two objects with the same hash code do not necessarily
have to be equal. This contract is essential for HashMap and HashSet.

### Q20. Difference between ArrayList and LinkedList?

  -----------------------------------------------------------------------
  ArrayList                           LinkedList
  ----------------------------------- -----------------------------------
  Dynamic array                       Doubly linked list

  Fast random access                  Slow random access

  Insertions in the middle may        Insertions after locating the node
  require shifting                    are efficient

  Usually faster iteration            Higher pointer overhead

  Lower memory overhead               Higher memory overhead
  -----------------------------------------------------------------------

### Q21. Difference between checked and unchecked exceptions?

**Answer:**

Checked exceptions are checked by the compiler and must be caught or
declared. Examples: IOException and SQLException.

Unchecked exceptions occur at runtime and extend RuntimeException.
Examples: NullPointerException and IllegalArgumentException.

### Q22. Difference between final, finally, and finalize?

-   `final`: Restricts reassignment, overriding, or inheritance
    depending on where it is used.
-   `finally`: A block commonly used for cleanup after try/catch.
-   `finalize()`: An obsolete finalization mechanism; deprecated and
    should not be used for resource management.

### Q23. What is multithreading?

**Answer:**

Multithreading allows multiple threads to execute concurrently within a
process. It can improve resource utilization and throughput for suitable
workloads.

Java supports threads through Thread, Runnable, Callable,
ExecutorService, and other concurrency utilities.

### Q24. Difference between synchronized and volatile?

**Answer:**

`synchronized` provides mutual exclusion and memory visibility
guarantees when used correctly. `volatile` guarantees visibility and
ordering for reads and writes of a variable, but does not make compound
operations such as `count++` atomic.

### Q25. What is CompletableFuture?

**Answer:**

CompletableFuture is used for asynchronous and non-blocking task
composition.

``` java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> "Hello")
        .thenApply(value -> value + " Java");

System.out.println(future.join());
```

Important methods include `supplyAsync()`, `runAsync()`, `thenApply()`,
`thenCompose()`, `thenCombine()`, `exceptionally()`, and `allOf()`.

### Q26. Difference between String, StringBuilder, and StringBuffer?

  -----------------------------------------------------------------------
  String                  StringBuilder           StringBuffer
  ----------------------- ----------------------- -----------------------
  Immutable               Mutable                 Mutable

  Concatenation creates   Efficient modifications Efficient modifications
  new objects                                     

  Safe to share due to    Not synchronized        Synchronized
  immutability                                    
  -----------------------------------------------------------------------

### Q27. Explain JVM, JRE, and JDK.

-   JVM: Executes Java bytecode.
-   JRE: JVM plus runtime libraries required to execute Java
    applications.
-   JDK: Development kit containing compiler and development tools along
    with runtime components.

### Q28. How does garbage collection work in Java?

**Answer:**

Garbage collection automatically reclaims heap memory occupied by
objects that are no longer reachable.

Topics to prepare: Young Generation, Old Generation, minor and major
collection terminology, G1 Garbage Collector, garbage collection pauses,
and memory leaks caused by unintended object references.

------------------------------------------------------------------------

# 4. Spring Boot and Spring Framework

### Q29. What is Spring Boot, and why do we use it?

**Answer:**

Spring Boot simplifies Spring application development through:

-   Auto-configuration.
-   Starter dependencies.
-   Embedded servers.
-   Production-ready monitoring features through Actuator.
-   Externalized configuration.
-   Simplified application setup.

It reduces configuration overhead and accelerates development.

### Q30. Difference between Spring and Spring Boot?

  Spring Framework                              Spring Boot
  --------------------------------------------- -------------------------------------
  Core application framework                    Simplifies Spring application setup
  More explicit configuration may be required   Auto-configuration
  Server setup can be external                  Embedded server support
  Dependency selection is more manual           Starter dependencies

### Q31. What is dependency injection?

**Answer:**

Dependency Injection is a design pattern in which an object's
dependencies are supplied externally rather than created directly by
that object.

``` java
@Service
public class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring manages the object lifecycle and injects the repository
dependency. Constructor injection is generally preferred because it
supports immutability and easier unit testing.

### Q32. Difference between @Component, @Service, @Repository, and @Controller?

-   `@Component`: Generic Spring-managed component.
-   `@Service`: Service/business logic layer.
-   `@Repository`: Persistence layer and exception translation.
-   `@Controller`: MVC controller that typically returns views.
-   `@RestController`: REST controller returning response bodies.

### Q33. What is Spring Bean lifecycle?

**Answer:**

A simplified lifecycle is:

1.  Bean instantiation.
2.  Dependency population.
3.  Aware callbacks, where applicable.
4.  Bean post-processors before initialization.
5.  Initialization callbacks.
6.  Bean post-processors after initialization.
7.  Bean available for use.
8.  Destruction callbacks when the context closes, for applicable
    scopes.

### Q34. What is Spring Boot auto-configuration?

**Answer:**

Auto-configuration automatically configures beans based on classpath
dependencies, existing beans, and configuration conditions. For example,
when relevant web dependencies are present, Spring Boot can configure
web application infrastructure automatically.

### Q35. Difference between @Bean and @Component?

**Answer:**

`@Component` marks a class for component scanning. `@Bean` is applied to
a method, generally within a configuration class, to explicitly register
the returned object as a Spring bean.

### Q36. Explain @Transactional.

**Answer:**

`@Transactional` manages transaction boundaries.

-   A transaction is started or joined, depending on propagation.
-   Database operations execute within that transaction.
-   The transaction commits if execution completes successfully.
-   It rolls back according to configured rollback rules.

By default, Spring rolls back on unchecked exceptions and Errors, not
all checked exceptions.

Follow-ups: transaction propagation, isolation, `rollbackFor`,
`readOnly`, and why self-invocation may bypass proxy-based transaction
behavior.

### Q37. What are transaction propagation types?

  Type            Meaning
  --------------- -----------------------------------------------------
  REQUIRED        Joins existing transaction or creates one
  REQUIRES_NEW    Suspends existing transaction and creates a new one
  SUPPORTS        Uses an existing transaction if available
  NOT_SUPPORTED   Executes without a transaction
  MANDATORY       Requires an existing transaction
  NEVER           Fails if a transaction exists
  NESTED          Uses a nested transaction where supported

### Q38. How do you handle exceptions globally in Spring Boot?

**Answer:**

Use `@RestControllerAdvice` and `@ExceptionHandler`.

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(
            ResourceNotFoundException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
    }
}
```

This centralizes exception handling and maintains consistent API error
responses.

### Q39. What is Spring Boot Actuator?

**Answer:**

Spring Boot Actuator provides operational endpoints for monitoring and
managing applications.

Common endpoints include `/actuator/health`, `/actuator/info`,
`/actuator/metrics`, and `/actuator/prometheus` when configured.
Sensitive endpoints should be protected.

### Q40. How do you secure REST APIs in Spring Boot?

**Answer:**

Use Spring Security, OAuth 2.0, OpenID Connect, JWT validation,
role-based access control, HTTPS, input validation, appropriate CORS
configuration, rate limiting at an API gateway, and secure secret
management.

------------------------------------------------------------------------

# 5. Microservices and distributed systems

### Q41. What are microservices?

**Answer:**

Microservices architecture divides an application into small,
independently deployable services. Each service focuses on a specific
business capability and communicates with other services through APIs or
messaging systems.

Advantages: independent deployment, independent scaling, technology
flexibility, modularity, and fault isolation.

Challenges: distributed transactions, network failures, service
discovery, observability, data consistency, and deployment complexity.

### Q42. Monolithic architecture versus microservices?

  -----------------------------------------------------------------------
  Monolithic                          Microservices
  ----------------------------------- -----------------------------------
  Single deployable application       Multiple independently deployable
                                      services

  Often shared codebase               Separate service codebases or
                                      modules

  Scaling often application-wide      Individual services can scale
                                      independently

  Simpler initial deployment          More complex operations

  Internal communication              Network-based communication
  -----------------------------------------------------------------------

### Q43. How do microservices communicate?

**Answer:**

Synchronous communication includes REST and gRPC. Asynchronous
communication includes Apache Kafka, RabbitMQ, AWS SQS, and AWS SNS.

Synchronous communication is useful when an immediate response is
required. Asynchronous communication is useful for decoupling and
background processing.

### Q44. What is API Gateway?

**Answer:**

API Gateway acts as a single entry point for client requests.
Responsibilities can include request routing, authentication,
authorization, rate limiting, SSL termination, request transformation,
and logging.

### Q45. What is service discovery?

**Answer:**

Service discovery allows services to dynamically locate other service
instances. It is useful where instances are frequently created,
terminated, or scaled.

### Q46. What is the Circuit Breaker pattern?

**Answer:**

Circuit Breaker prevents repeated calls to a failing downstream service.

1.  Closed: Requests flow normally.
2.  Open: Requests are rejected or fail fast.
3.  Half-open: A limited number of requests are allowed to test
    recovery.

This helps prevent cascading failures.

### Q47. What is the Saga pattern?

**Answer:**

Saga manages distributed transactions through a sequence of local
transactions. If one step fails, compensating transactions can reverse
the effects of earlier steps.

-   Choreography: Services react to events.
-   Orchestration: A central coordinator manages the workflow.

### Q48. What is eventual consistency?

**Answer:**

Eventual consistency means distributed replicas or services may
temporarily contain different data, but are expected to converge if
updates stop and replication succeeds.

### Q49. What is idempotency?

**Answer:**

An operation is idempotent if executing it multiple times has the same
intended effect as executing it once. A payment API can use an
idempotency key to prevent duplicate charges when clients retry
requests.

### Q50. How do you handle failures between microservices?

**Answer:**

Use timeouts, bounded retry policies, exponential backoff, circuit
breakers, bulkheads, fallback mechanisms, dead-letter queues, idempotent
consumers, distributed tracing, monitoring, and alerting.

Retries should be limited and should not amplify an overloaded
downstream service.

### Q51. What is the difference between REST and SOAP?

**Answer:**

REST is an architectural style generally using HTTP resources and
representations. SOAP is a messaging protocol based on XML with formal
message structures and related standards.

REST is generally simpler for modern web APIs, while SOAP remains
relevant in some enterprise integrations.

### Q52. What are micro frontends?

**Answer:**

Micro frontends divide a frontend application into independently
developed and managed UI modules. Teams can own different functional
areas.

Integration approaches include Module Federation, Web Components,
build-time integration, and runtime integration.

Challenges include shared dependencies, routing, styling consistency,
and cross-module communication.

------------------------------------------------------------------------

# 6. React and JavaScript

### Q53. What is React?

**Answer:**

React is a JavaScript library used to build component-based user
interfaces. Important features include component-based architecture,
Virtual DOM, JSX, props and state, hooks, declarative rendering, and
reusable UI components.

### Q54. Difference between props and state?

  -----------------------------------------------------------------------
  Props                               State
  ----------------------------------- -----------------------------------
  Passed from parent to child         Managed within a component

  Read-only                           Updated using state setters

  Used to configure components        Used for dynamic data

  Changes when parent provides new    Changes through component logic
  values                              
  -----------------------------------------------------------------------

### Q55. What are React hooks?

**Answer:**

Hooks allow functional components to use React features such as state,
lifecycle-related effects, and context.

Important hooks: `useState`, `useEffect`, `useContext`, `useMemo`,
`useCallback`, `useRef`, and `useReducer`.

### Q56. Explain useState and useEffect.

**Answer:**

`useState` manages component state.

``` jsx
const [count, setCount] = useState(0);
```

`useEffect` performs side effects such as API calls, subscriptions, and
synchronization with external systems.

``` jsx
useEffect(() => {
    fetchUsers();
}, []);
```

An empty dependency array means the effect runs after the initial mount,
subject to React Strict Mode development behavior.

### Q57. What is the Virtual DOM?

**Answer:**

The Virtual DOM is an in-memory representation of the UI. When state
changes, React calculates required updates by comparing the previous and
new UI representations and commits necessary changes to the actual DOM.

### Q58. Difference between useMemo and useCallback?

**Answer:**

-   `useMemo` memoizes a calculated value.
-   `useCallback` memoizes a function reference.

``` jsx
const filteredUsers = useMemo(() => {
    return users.filter(user => user.active);
}, [users]);

const handleClick = useCallback(() => {
    console.log("Clicked");
}, []);
```

Use them when there is a meaningful performance benefit, not
automatically everywhere.

### Q59. What is React reconciliation?

**Answer:**

Reconciliation is the process React uses to determine what changes are
required when the component tree changes. React compares elements and
uses keys to identify items in lists.

### Q60. Why are keys important in React?

**Answer:**

Keys help React identify individual elements across renders.

``` jsx
users.map(user => (
    <User key={user.id} user={user} />
))
```

Using array indexes as keys can cause issues when elements are inserted,
removed, or reordered.

### Q61. What is a controlled versus uncontrolled component?

**Answer:**

A controlled component gets its value from React state. An uncontrolled
component stores its value in the DOM and is generally accessed using
refs. Controlled components provide better control over validation and
form behavior.

### Q62. What is prop drilling?

**Answer:**

Prop drilling means passing props through multiple component layers even
when intermediate components do not need those values. Possible
solutions include Context API, Redux, Zustand, and component
composition.

### Q63. What is Redux?

**Answer:**

Redux is a state management library that centralizes application state.
Core concepts include Store, Actions, Reducers, Dispatch, and Selectors.
Redux Toolkit is the recommended modern approach for most Redux
applications.

### Q64. How do you optimize React performance?

**Answer:**

-   Use React.memo where appropriate.
-   Use useMemo and useCallback where useful.
-   Avoid unnecessary state updates.
-   Use lazy loading and code splitting.
-   Virtualize large lists.
-   Avoid unnecessary component re-renders.
-   Optimize API calls.
-   Use appropriate keys.

### Q65. How do you call a Spring Boot API from React?

**Answer:**

Use Fetch API or Axios.

``` javascript
import axios from "axios";

async function fetchUsers() {
    try {
        const response = await axios.get("/api/users");
        return response.data;
    } catch (error) {
        console.error(error);
        throw error;
    }
}
```

A production application should also handle authentication, loading
states, errors, request cancellation, and API timeouts.

### Q66. What is CORS?

**Answer:**

Cross-Origin Resource Sharing is a browser security mechanism that
controls requests made from one origin to another. Spring Boot can
configure CORS globally or for individual controllers. CORS is not a
substitute for authentication or authorization.

### Q67. Difference between var, let, and const?

  -----------------------------------------------------------------------
  var                     let                     const
  ----------------------- ----------------------- -----------------------
  Function scoped         Block scoped            Block scoped

  Can be redeclared       Cannot be redeclared in Cannot be redeclared in
                          same scope              same scope

  Can be reassigned       Can be reassigned       Cannot be reassigned

  Hoisted and initialized Hoisted but in temporal Hoisted but in temporal
  as undefined            dead zone               dead zone
  -----------------------------------------------------------------------

### Q68. Difference between == and === in JavaScript?

**Answer:**

`==` performs equality comparison with type coercion. `===` checks
equality without type coercion.

``` javascript
console.log(5 == "5");  // true
console.log(5 === "5"); // false
```

### Q69. What is a Promise?

**Answer:**

A Promise represents the eventual completion or failure of an
asynchronous operation. It has three states: pending, fulfilled, and
rejected.

``` javascript
fetch("/api/users")
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));
```

### Q70. What is async/await?

**Answer:**

Async/await provides cleaner syntax for working with Promises.

``` javascript
async function getUsers() {
    try {
        const response = await fetch("/api/users");
        const data = await response.json();
        return data;
    } catch (error) {
        console.error(error);
    }
}
```

### Q71. Difference between Promise.all() and Promise.allSettled()?

**Answer:**

`Promise.all()` rejects when one of its input promises rejects.
`Promise.allSettled()` waits for all input promises to settle and
returns individual outcomes.

### Q72. What is event bubbling?

**Answer:**

Event bubbling means that an event triggered on a child element
propagates upward through its parent elements. Event delegation uses
this behavior to handle events at a common ancestor rather than
attaching handlers to every child.

------------------------------------------------------------------------

# 7. SQL, PostgreSQL, DB2, and database concepts

### Q73. Difference between DELETE, TRUNCATE, and DROP?

  -----------------------------------------------------------------------
  DELETE                  TRUNCATE                DROP
  ----------------------- ----------------------- -----------------------
  Removes selected rows   Removes all rows        Removes the database
  or all rows                                     object

  Supports WHERE          No WHERE                Removes table structure

  DML                     Generally classified as DDL
                          DDL                     

  Row-level deletion      Typically bulk          Object removal
  behavior                operation               
  -----------------------------------------------------------------------

Transaction and rollback behavior can vary by database engine.

### Q74. What are primary key and foreign key?

**Answer:**

A primary key uniquely identifies a record in a table. A foreign key
establishes a relationship with a candidate or primary key in another
table and helps maintain referential integrity.

### Q75. Difference between INNER JOIN and LEFT JOIN?

**Answer:**

-   INNER JOIN returns matching records from both tables.
-   LEFT JOIN returns all records from the left table and matching
    records from the right table. Nonmatching right-side values are
    NULL.

``` sql
SELECT e.name, d.department_name
FROM employee e
LEFT JOIN department d
    ON e.department_id = d.id;
```

### Q76. What are SQL indexes?

**Answer:**

An index is a data structure that helps the database locate rows more
efficiently.

Advantages: faster reads and improved lookup/join performance in
suitable cases.

Disadvantages: additional storage, slower inserts/updates/deletes due to
index maintenance, and possible overhead from poorly selected indexes.

### Q77. What is normalization?

**Answer:**

Normalization organizes relational data to reduce redundancy and improve
data integrity.

-   1NF: Atomic values and no repeating groups.
-   2NF: 1NF plus no partial dependency on a candidate key.
-   3NF: 2NF plus no inappropriate transitive dependency on candidate
    keys.
-   BCNF: Every determinant is a candidate key.

### Q78. What is denormalization?

**Answer:**

Denormalization deliberately introduces redundancy to improve read
performance or simplify queries. It can increase storage usage and make
data consistency more difficult.

### Q79. Difference between WHERE and HAVING?

**Answer:**

`WHERE` filters rows before grouping. `HAVING` filters groups after
aggregation.

``` sql
SELECT department_id, COUNT(*)
FROM employee
WHERE status = 'ACTIVE'
GROUP BY department_id
HAVING COUNT(*) > 5;
```

### Q80. Find the second-highest salary.

**Answer -- subquery:**

``` sql
SELECT MAX(salary)
FROM employee
WHERE salary < (
    SELECT MAX(salary)
    FROM employee
);
```

**Answer -- DENSE_RANK:**

``` sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employee
) t
WHERE rnk = 2;
```

The second approach handles duplicate salaries correctly.

### Q81. Find duplicate records in a table.

**Answer:**

``` sql
SELECT email, COUNT(*)
FROM employee
GROUP BY email
HAVING COUNT(*) > 1;
```

### Q82. Find the highest salary in each department.

**Answer:**

``` sql
SELECT department_id, MAX(salary)
FROM employee
GROUP BY department_id;
```

To return employee details as well:

``` sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (
               PARTITION BY department_id
               ORDER BY salary DESC
           ) AS rnk
    FROM employee e
) t
WHERE rnk = 1;
```

### Q83. Difference between UNION and UNION ALL?

**Answer:**

UNION removes duplicate rows. UNION ALL retains duplicate rows and is
generally faster because it does not perform duplicate elimination.

### Q84. What are ACID properties?

**Answer:**

-   Atomicity: Transaction is all-or-nothing.
-   Consistency: Transaction preserves defined integrity constraints.
-   Isolation: Concurrent transactions are controlled according to
    isolation rules.
-   Durability: Committed changes survive failures according to database
    guarantees.

### Q85. What are transaction isolation levels?

  Isolation level    General behavior
  ------------------ -------------------------------
  Read Uncommitted   May allow dirty reads
  Read Committed     Prevents dirty reads
  Repeatable Read    Prevents non-repeatable reads
  Serializable       Strongest standard isolation

Exact behavior and supported levels can differ between database systems.

### Q86. What is a deadlock?

**Answer:**

A deadlock occurs when two or more transactions wait indefinitely for
resources held by each other.

Solutions include consistent lock acquisition order, short transactions,
appropriate indexing, deadlock detection, and retry handling.

### Q87. What is the difference between SQL and NoSQL?

  -----------------------------------------------------------------------
  SQL                                 NoSQL
  ----------------------------------- -----------------------------------
  Relational model                    Multiple data models

  Structured schema                   Often flexible schema

  Tables and relationships            Documents, key-value, graph, etc.

  SQL queries                         Database-specific APIs/query
                                      languages

  Strong relational capabilities      Often suited to specialized
                                      scalability and access patterns
  -----------------------------------------------------------------------

### Q88. PostgreSQL versus DB2 -- what should you know?

**PostgreSQL:**

-   Open-source relational database.
-   Supports SQL, JSON/JSONB, indexing, and advanced query capabilities.
-   Supports MVCC.
-   Has extensive extensions.

**IBM Db2:**

-   Enterprise relational database platform.
-   Common in large enterprise and legacy environments.
-   Supports SQL, transactions, indexing, stored procedures, and
    enterprise workload management.

Prepare PostgreSQL data types, JSONB, sequences and identity columns,
EXPLAIN and EXPLAIN ANALYZE, B-tree and other index types, Db2 schema
and SQL conventions, pagination syntax differences, window functions,
isolation, and locking behavior.

### Q89. How do you optimize a slow SQL query?

**Answer:**

1.  Identify the slow query.
2.  Examine the execution plan.
3.  Check indexes.
4.  Avoid unnecessary SELECT \*.
5.  Reduce unnecessary joins.
6.  Filter records early where appropriate.
7.  Review subqueries and correlated queries.
8.  Check table statistics.
9.  Review pagination and sorting.
10. Measure the result after optimization.

This connects directly with the SQL optimization work described in your
Capgemini experience.

------------------------------------------------------------------------

# 8. AWS interview questions

### Q90. What is IAM?

**Answer:**

AWS Identity and Access Management controls who can access AWS resources
and what actions they can perform.

Main concepts: users, groups, roles, policies, permissions, and least
privilege.

An IAM role provides temporary credentials and is commonly used by AWS
services and applications.

### Q91. What is EC2?

**Answer:**

Amazon EC2 provides resizable virtual compute instances in AWS. You can
configure operating system, instance type, CPU and memory, storage,
networking, and security groups.

Typical use cases include hosting applications, backend services, and
custom workloads.

### Q92. What is ECS?

**Answer:**

Amazon Elastic Container Service is a managed container orchestration
service. It runs and manages containerized applications.

Important concepts: cluster, task definition, task, service, container
image, launch type, Fargate, and EC2 capacity.

### Q93. Difference between ECS and EKS?

**Answer:**

ECS is AWS's container orchestration service. EKS is AWS's managed
Kubernetes service.

ECS is typically simpler when using AWS-native container orchestration.
EKS is suitable when Kubernetes compatibility and its ecosystem are
required.

### Q94. What is AWS Lambda?

**Answer:**

AWS Lambda is a serverless compute service that executes code in
response to events, such as API requests, S3 events, SQS messages,
scheduled triggers, and EventBridge events. You do not manage the
underlying servers.

### Q95. What is S3?

**Answer:**

Amazon S3 is an object storage service. It stores data in buckets as
objects.

Use cases include document storage, images, backups, logs, static
website assets, and data lakes.

Important concepts: bucket policies, IAM policies, versioning, lifecycle
rules, encryption, and presigned URLs.

### Q96. Difference between SQS and SNS?

  -----------------------------------------------------------------------
  SQS                                 SNS
  ----------------------------------- -----------------------------------
  Message queue                       Publish-subscribe messaging

  Consumers poll messages             Pushes notifications to subscribers

  Message remains in queue until      Delivers messages to subscribers
  processed and deleted               

  Useful for decoupled processing     Useful for broadcasting events

  Supports visibility timeout         Supports subscription delivery and
                                      filtering
  -----------------------------------------------------------------------

Example: An order service publishes an order-created event to SNS. SNS
can distribute it to multiple SQS queues, where different microservices
process their respective tasks.

### Q97. What is a Dead Letter Queue?

**Answer:**

A Dead Letter Queue stores messages that cannot be successfully
processed after configured delivery or receive attempts. It helps with
debugging, failure isolation, message inspection, and controlled
reprocessing.

### Q98. What is API Gateway?

**Answer:**

Amazon API Gateway allows developers to create, publish, secure, and
monitor APIs. It can integrate with AWS Lambda, HTTP backends, and other
AWS services. It supports authentication, throttling, monitoring, and
API management capabilities.

### Q99. What is DynamoDB?

**Answer:**

Amazon DynamoDB is a fully managed NoSQL database. It supports key-value
and document data models.

Important concepts: partition key, sort key, global secondary index,
local secondary index, on-demand capacity, provisioned capacity, TTL,
Streams, and conditional writes.

### Q100. Difference between RDS and DynamoDB?

  -----------------------------------------------------------------------
  RDS                                 DynamoDB
  ----------------------------------- -----------------------------------
  Relational database                 NoSQL database

  SQL-based                           Key-value/document model

  Supports relational joins           No traditional relational joins

  Structured schema                   Flexible item attributes

  Suitable for relational data        Suitable for known high-scale
                                      access patterns
  -----------------------------------------------------------------------

### Q101. What is CloudWatch?

**Answer:**

Amazon CloudWatch provides monitoring and observability for AWS
resources and applications. It provides metrics, logs, alarms,
dashboards, and event-related capabilities.

Example: Trigger an alarm when Lambda errors or API latency exceed a
defined threshold.

### Q102. What is CloudFormation?

**Answer:**

AWS CloudFormation is an Infrastructure as Code service. It allows you
to define AWS infrastructure in templates and deploy it consistently.

Benefits: repeatable environments, version-controlled infrastructure,
automated resource provisioning, change management, and reduced manual
configuration.

### Q103. What is AWS Systems Manager (SSM)?

**Answer:**

AWS Systems Manager helps centrally manage and operate AWS resources.

Common capabilities include Parameter Store, Session Manager, Run
Command, Automation, and Patch Manager.

Parameter Store can store configuration values and secrets with
appropriate security controls.

### Q104. How would you deploy a Spring Boot microservice on AWS?

**Answer:**

1.  Develop the Spring Boot application.
2.  Create a Docker image.
3.  Push the image to Amazon ECR.
4.  Define an ECS task definition.
5.  Deploy the service on ECS using Fargate or EC2.
6.  Configure networking and security groups.
7.  Expose the service through an Application Load Balancer or API
    Gateway, as appropriate.
8.  Store configuration securely.
9.  Configure CloudWatch logs and monitoring.
10. Set up CI/CD for repeatable deployments.

### Q105. How would you build a serverless application?

**Answer:**

A typical architecture is:

React application → Amazon API Gateway → AWS Lambda → DynamoDB and/or
S3.

CloudWatch provides monitoring and logging across the architecture. This
avoids managing application servers and supports event-driven scaling.

### Q106. Difference between SNS, SQS, and EventBridge?

**Answer:**

-   SNS: Pub/sub messaging and fan-out.
-   SQS: Durable message queue for asynchronous processing.
-   EventBridge: Event bus for routing events based on rules between
    producers and consumers.

### Q107. Difference between ECS Fargate and ECS EC2 launch types?

**Answer:**

Fargate: AWS manages the underlying compute infrastructure.

EC2: You manage the EC2 instances that provide compute capacity for
containers.

Fargate reduces infrastructure management. EC2 gives more control over
the underlying hosts.

### Q108. How do you secure an AWS application?

**Answer:**

-   Use least-privilege IAM roles.
-   Avoid hardcoded credentials.
-   Use Secrets Manager or SSM Parameter Store.
-   Encrypt data at rest and in transit.
-   Configure security groups.
-   Use private subnets where appropriate.
-   Enable monitoring and audit logging.
-   Protect APIs using authentication and throttling.
-   Rotate credentials where applicable.

------------------------------------------------------------------------

# 9. JUnit, Mockito, and Test Driven Development

### Q109. What is JUnit?

**Answer:**

JUnit is a Java testing framework used to write and execute unit tests.
It helps validate application behavior and catch regressions.

JUnit 5 includes JUnit Platform, JUnit Jupiter, and JUnit Vintage.

### Q110. Difference between JUnit 4 and JUnit 5?

  JUnit 4                  JUnit 5
  ------------------------ ------------------------------------
  `@Test` from org.junit   `@Test` from org.junit.jupiter.api
  `@Before`                `@BeforeEach`
  `@After`                 `@AfterEach`
  `@BeforeClass`           `@BeforeAll`
  `@AfterClass`            `@AfterAll`
  Runners                  Extensions model

### Q111. What is Mockito?

**Answer:**

Mockito is a Java mocking framework used to create test doubles for
dependencies. It helps isolate a class under test from databases,
external services, and other dependencies.

### Q112. Difference between @Mock and @InjectMocks?

**Answer:**

-   `@Mock`: Creates a mock dependency.
-   `@InjectMocks`: Creates the class under test and injects available
    mocks into it.

### Q113. Difference between mock() and spy()?

**Answer:**

A mock is a test double whose behavior is configured. A spy wraps a real
object and can call real methods unless behavior is stubbed.

Spies should be used carefully because partial mocking can make tests
harder to understand.

### Q114. Explain when(), thenReturn(), and verify().

``` java
when(userRepository.findById(1L))
    .thenReturn(Optional.of(user));

verify(userRepository).findById(1L);
```

-   `when()` defines behavior.
-   `thenReturn()` specifies the returned value.
-   `verify()` checks whether a mock interaction occurred.

### Q115. How do you test a Spring Boot REST controller?

**Answer:**

Use MockMvc with `@WebMvcTest` to test controller behavior without
starting the full application context.

``` java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUsers() throws Exception {
        when(userService.getUsers()).thenReturn(List.of());

        mockMvc.perform(get("/users"))
            .andExpect(status().isOk());
    }
}
```

The example uses an empty list to illustrate test structure; actual
tests should assert meaningful response data. In newer Spring Boot
versions, check the supported Mockito bean-override annotations for the
version in use.

### Q116. What is Test Driven Development?

**Answer:**

TDD is a development methodology where tests are written before
implementation. It follows the Red-Green-Refactor cycle:

1.  Red: Write a failing test.
2.  Green: Write minimum implementation to pass.
3.  Refactor: Improve the implementation while maintaining passing
    tests.

Benefits: better test coverage, clearer requirements, reduced
regressions, better modularity, and faster feedback.

### Q117. Difference between unit testing and integration testing?

  -----------------------------------------------------------------------
  Unit testing                        Integration testing
  ----------------------------------- -----------------------------------
  Tests a small unit in isolation     Tests interactions between
                                      components

  Usually mocks dependencies          Often uses real infrastructure or
                                      test containers

  Faster                              Usually slower

  Detects logic errors                Detects integration issues
  -----------------------------------------------------------------------

### Q118. How do you test exception scenarios?

**Answer:**

JUnit 5 supports `assertThrows()`.

``` java
@Test
void shouldThrowException() {
    assertThrows(
        IllegalArgumentException.class,
        () -> service.findUser(-1L)
    );
}
```

### Q119. What is code coverage?

**Answer:**

Code coverage measures which parts of source code are executed during
tests.

Common types: line coverage, branch coverage, method coverage, and
instruction coverage.

High coverage does not automatically mean high-quality tests. Assertions
must verify meaningful behavior.

### Q120. What is SonarQube?

**Answer:**

SonarQube performs static code analysis to identify bugs,
vulnerabilities, code smells, duplicated code, and maintainability
issues.

It can be integrated into a CI/CD pipeline to enforce quality gates.

------------------------------------------------------------------------

# 10. Kafka and event-driven architecture

### Q121. What is Apache Kafka?

**Answer:**

Kafka is a distributed event-streaming platform that supports
high-throughput, fault-tolerant event processing.

Main components: producer, consumer, topic, partition, broker, consumer
group, and offset.

### Q122. What is a Kafka topic?

**Answer:**

A topic is a named logical stream of records. Topics are divided into
partitions to support scalability and parallelism.

### Q123. What is a Kafka partition?

**Answer:**

A partition is an ordered sequence of records within a topic. Kafka
guarantees ordering within a partition, not globally across all
partitions of a topic.

### Q124. What is a consumer group?

**Answer:**

A consumer group allows multiple consumers to share the processing
workload of a topic. Within a consumer group, a partition is assigned to
at most one consumer at a time under normal assignment rules.

### Q125. What is an offset?

**Answer:**

An offset is the position of a record within a Kafka partition.
Consumers track offsets to know which records have been processed or
acknowledged.

### Q126. Difference between at-most-once, at-least-once, and exactly-once processing?

**Answer:**

-   At-most-once: Messages may be lost but are not processed more than
    once.
-   At-least-once: Messages may be processed more than once.
-   Exactly-once: Processing effects are coordinated to provide
    exactly-once semantics within supported boundaries.

Exactly-once delivery and exactly-once business effects are not
automatically the same thing.

### Q127. What is a Kafka rebalance?

**Answer:**

A rebalance occurs when Kafka redistributes partitions among consumers
in a group. It may happen when consumers join, leave, fail, or when
topic partition assignments change.

### Q128. Kafka versus AWS SQS?

  -----------------------------------------------------------------------
  Kafka                               SQS
  ----------------------------------- -----------------------------------
  Distributed event streaming         Managed message queue

  Retains events based on retention   Messages removed after successful
  configuration                       processing and deletion

  Consumer offsets                    Visibility timeout and message
                                      deletion

  Partition-based ordering            FIFO queues offer ordered
                                      processing within supported message
                                      groups

  Suitable for event streaming and    Suitable for managed asynchronous
  replay                              task processing
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 11. Docker, Kubernetes, Jenkins, and DevOps

### Q129. What is Docker?

**Answer:**

Docker is a containerization platform that packages an application with
its dependencies into a container image.

Benefits: consistent environments, portability, faster deployment,
isolation, and efficient resource utilization.

### Q130. Difference between Docker image and container?

**Answer:**

An image is a read-only template used to create containers. A container
is a running instance of an image.

### Q131. What is a Dockerfile?

**Answer:**

A Dockerfile defines the instructions required to build a Docker image.

``` dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

For a Java 8 project, select a Java 8-compatible base image and runtime.
The example above uses Java 17, not Java 8.

### Q132. What is Docker Compose?

**Answer:**

Docker Compose defines and runs multi-container applications using a
YAML configuration file. It is useful for local development environments
involving multiple services, databases, and message brokers.

### Q133. What is Kubernetes?

**Answer:**

Kubernetes is a container orchestration platform used to deploy, scale,
and manage containerized applications.

Important concepts: Pod, Deployment, Service, ConfigMap, Secret,
Namespace, Ingress, ReplicaSet, Persistent Volume, and Horizontal Pod
Autoscaler.

### Q134. Difference between Deployment and StatefulSet?

**Answer:**

Deployment is generally used for stateless workloads. StatefulSet is
used for workloads that need stable network identities, persistent
storage associations, and ordered deployment or scaling behavior.

### Q135. Difference between readiness and liveness probes?

**Answer:**

-   Readiness probe: Determines whether a container is ready to receive
    traffic.
-   Liveness probe: Determines whether a container needs restarting.
-   Startup probe: Can be used for applications with slow
    initialization.

### Q136. What is Jenkins?

**Answer:**

Jenkins is an automation server used to build CI/CD pipelines. It can
automate source-code checkout, compilation, unit testing, static
analysis, Docker image creation, image publishing, and deployment.

### Q137. Difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?

**Answer:**

-   CI: Frequently integrate code and validate it through automated
    builds and tests.
-   Continuous Delivery: Keep changes deployable, with production
    release generally requiring approval.
-   Continuous Deployment: Automatically release changes to production
    after passing required checks.

### Q138. What is a Jenkinsfile?

**Answer:**

A Jenkinsfile defines the CI/CD pipeline as code.

``` groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Quality Check') {
            steps {
                echo 'Run quality analysis'
            }
        }
    }
}
```

### Q139. Difference between Git merge and rebase?

**Answer:**

Merge combines branches and generally creates a merge commit when
required. Rebase moves or replays commits onto another base commit,
creating a more linear history.

Rebase should be used carefully on shared branches because it rewrites
commit history.

### Q140. Difference between GitHub and Bitbucket?

**Answer:**

Both are Git-based source-code hosting platforms. They provide
repository management, branch management, pull requests, code reviews,
access control, and CI/CD integrations.

GitHub provides GitHub Actions, while Bitbucket integrates with
Bitbucket Pipelines.

------------------------------------------------------------------------

# 12. Claude, AI, and AI-assisted development

### Q141. What is an LLM?

**Answer:**

A Large Language Model is a machine learning model trained on large
amounts of text and other data to understand and generate language.

It can perform text generation, summarization, code generation, question
answering, reasoning assistance, and information extraction.

### Q142. How does an application integrate with Claude?

**Answer:**

A backend application can communicate with Anthropic's API using HTTP
requests.

1.  Receive a user request or system event.
2.  Prepare the prompt and contextual information.
3.  Authenticate using an API key stored securely.
4.  Send a request to the Anthropic Messages API.
5.  Receive the model response.
6.  Validate and process the response.
7.  Return the result or trigger an approved downstream action.

### Q143. What is prompt engineering?

**Answer:**

Prompt engineering is the practice of designing instructions and
contextual inputs to obtain useful and consistent model outputs.

Good prompts generally include clear instructions, relevant context,
output format, constraints, examples where useful, and expected
error-handling behavior.

### Q144. What are hallucinations in AI?

**Answer:**

Hallucinations occur when an AI model generates information that sounds
plausible but is incorrect or unsupported.

Ways to reduce their impact: provide relevant context, validate
generated output, use retrieval or authoritative data sources, require
structured outputs, use automated tests, and apply human review for
critical decisions.

### Q145. How do you securely integrate an LLM into an enterprise application?

**Answer:**

-   Keep API keys in secure secret stores.
-   Avoid sending sensitive information unnecessarily.
-   Mask personal and confidential data.
-   Validate model responses.
-   Restrict tool permissions.
-   Apply rate limiting.
-   Monitor token usage and costs.
-   Maintain audit logs.
-   Require human approval for sensitive actions.

### Q146. What is RAG?

**Answer:**

Retrieval-Augmented Generation combines information retrieval with
language-model generation.

1.  User submits a query.
2.  The system searches relevant documents or data.
3.  Relevant context is retrieved.
4.  The context is passed to the LLM.
5.  The LLM generates an answer grounded in the retrieved information.

RAG helps reduce unsupported responses and provides access to external
knowledge.

### Q147. Difference between fine-tuning and RAG?

  -----------------------------------------------------------------------
  Fine-tuning                         RAG
  ----------------------------------- -----------------------------------
  Changes model behavior through      Supplies external context at
  additional training                 inference time

  Useful for specialized behavior and Useful for current or
  patterns                            organization-specific knowledge

  Requires training data and process  Requires retrieval infrastructure

  Updating knowledge may require      Knowledge can be updated in the
  another training process            retrieval source
  -----------------------------------------------------------------------

### Q148. How would you use Claude to analyze a failed Java application?

**Answer:**

1.  Capture the exception stack trace.
2.  Collect relevant logs and execution metadata.
3.  Remove sensitive information.
4.  Identify the relevant code context.
5.  Send a structured debugging request to Claude.
6.  Ask for probable root causes and suggested fixes.
7.  Validate suggestions using unit and integration tests.
8.  Create a pull request for developer review.

Do not allow the LLM to independently deploy unverified fixes to
production.

### Q149. How would you prevent prompt injection?

**Answer:**

Prompt injection occurs when malicious input attempts to override system
instructions or manipulate model behavior.

Defenses include treating user-provided content as untrusted, separating
instructions from retrieved content, validating tool calls, restricting
permissions, using allowlists, avoiding exposing secrets to the model,
requiring approval for sensitive actions, and validating outputs before
execution.

### Q150. What is an AI agent?

**Answer:**

An AI agent is a system that uses an AI model to interpret a goal,
decide on actions, and interact with tools or external systems to
complete tasks.

An agent may use an LLM, tools, memory, planning, execution loops, and
validation. It should have clearly defined permissions and execution
boundaries.

------------------------------------------------------------------------

# 13. Agile, JIRA, and behavioral questions

### Q151. What is Agile?

**Answer:**

Agile is an iterative software development approach focused on
continuous delivery, collaboration, adaptability, and customer feedback.

### Q152. What is a Scrum sprint?

**Answer:**

A sprint is a fixed-length development iteration during which a team
completes a defined set of work.

Common Scrum events: sprint planning, Daily Scrum, sprint review, and
sprint retrospective.

### Q153. What is a user story?

**Answer:**

A user story describes a feature from the perspective of the user.

Common format: "As a \[user\], I want \[functionality\], so that
\[benefit\]."

### Q154. Difference between story points and hours?

**Answer:**

Story points estimate relative effort and complexity. Hours estimate
actual time. Story points are commonly used for sprint planning and
velocity tracking.

### Q155. What is a pull request?

**Answer:**

A pull request proposes changes to a codebase for review and eventual
integration. It enables code review, automated testing, quality checks,
discussion, approval, and controlled merging.

### Q156. How do you handle a production issue?

**Suggested answer:**

"My approach is to first understand the impact and severity of the
issue.

I check logs, monitoring dashboards, error messages, and recent
deployments to identify the likely cause.

I then isolate the failing component, reproduce the problem where
possible, and implement a fix.

Before deployment, I validate the fix with relevant tests. After
deployment, I monitor the application to ensure that the issue has been
resolved and no regressions have been introduced.

I also document the root cause and preventive actions."

### Q157. What do you do when you disagree with a team member?

**Suggested answer:**

"I first understand their perspective and compare both approaches
against technical requirements, maintainability, performance, and
business impact.

If necessary, I use a small proof of concept or technical evidence to
evaluate the alternatives.

My objective is to reach the best technical decision rather than prove
that my approach is correct."

### Q158. How do you prioritize multiple tasks?

**Suggested answer:**

"I consider business impact, deadlines, dependencies, severity, and
effort.

I coordinate priorities with the product owner or team lead and
communicate risks early if multiple critical tasks compete for the same
resources."

### Q159. What is your biggest technical achievement?

**Suggested answer based on the resume:**

"One of my notable contributions was implementing an AI-assisted
failure-analysis workflow for scheduled jobs running in Kubernetes.

It combined my backend development experience with Anthropic's API and
automated development workflows.

The objective was to reduce manual debugging effort and accelerate issue
resolution while keeping developers involved in reviewing generated
fixes."

------------------------------------------------------------------------

# 14. Practical coding questions

For a five-year Java developer, theoretical questions alone are
insufficient. Expect live coding, shared-screen exercises, or take-home
assignments.

    No. Coding problem                        Key concept
  ----- ------------------------------------- -------------------------
      1 Reverse a string                      String manipulation
      2 Check palindrome                      Two pointers
      3 Find duplicate elements in an array   HashSet
      4 Find the second-highest number        Streams
      5 Count character frequency             HashMap
      6 Find first non-repeating character    LinkedHashMap
      7 Sort employees by salary              Comparator
      8 Group employees by department         Collectors.groupingBy
      9 Find maximum salary per department    Streams
     10 Remove duplicates from a list         distinct / Set
     11 Merge two sorted arrays               Two pointers
     12 Find missing number                   Mathematics / Set
     13 Fibonacci series                      Iteration / Recursion
     14 Check anagram strings                 Frequency map
     15 Find common elements in two lists     Set
     16 Implement LRU Cache                   LinkedHashMap
     17 Implement producer-consumer           BlockingQueue
     18 Find top K frequent elements          HashMap / PriorityQueue
     19 Implement custom immutable class      OOP
     20 Implement thread-safe Singleton       Concurrency

## Coding example 1: Find the second-highest number using Java 8

``` java
List<Integer> numbers = Arrays.asList(10, 20, 30, 30, 40);

Integer secondHighest = numbers.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElseThrow();

System.out.println(secondHighest); // 30
```

## Coding example 2: Count character frequency

``` java
String input = "programming";

Map<Character, Long> result = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));

System.out.println(result);
```

## Coding example 3: Find duplicate elements

``` java
List<Integer> numbers = Arrays.asList(1, 2, 3, 2, 4, 3, 5);

Set<Integer> seen = new HashSet<>();

Set<Integer> duplicates = numbers.stream()
    .filter(n -> !seen.add(n))
    .collect(Collectors.toSet());

System.out.println(duplicates);
```

## Coding example 4: Sort employees by salary

``` java
employees.stream()
    .sorted(
        Comparator.comparing(Employee::getSalary).reversed()
    )
    .forEach(System.out::println);
```

## Coding example 5: Group employees by department

``` java
Map<String, List<Employee>> result = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

------------------------------------------------------------------------

# 15. System design and real-world scenario questions

For a five-year developer, these questions can be more important than
memorizing definitions.

  -----------------------------------------------------------------------
  Question                            What your answer should cover
  ----------------------------------- -----------------------------------
  Design an e-commerce backend        Microservices, database, caching,
                                      API Gateway

  Design a notification service       SNS, SQS, retries, DLQ

  Design a file upload service        React, API Gateway, S3, Lambda

  Design an order processing system   Kafka, event-driven architecture,
                                      Saga

  Design a URL shortener              Hashing, database, caching

  Design a product feed system        External APIs, scheduling, retries,
                                      SQL

  Design a workflow engine            React Flow, JSON persistence,
                                      backend orchestration

  Design a self-healing job processor Kubernetes, Claude API, retries,
                                      validation

  Design a scalable reporting service SQL, asynchronous processing, CSV
                                      generation

  Design authentication for           Keycloak, OAuth 2.0, JWT, RBAC
  microservices                       
  -----------------------------------------------------------------------

## Example: Design a product feed processing system

This scenario is particularly relevant to the Infosys project described
in the resume.

**Suggested architecture:**

1.  React frontend allows users to configure product feeds.
2.  Spring Boot API receives requests.
3.  Backend validates product data.
4.  Product data is stored in a relational database.
5.  A background job or message queue triggers processing.
6.  An integration service communicates with external marketplace APIs.
7.  Retry logic handles temporary failures.
8.  Processing status is stored in the database.
9.  React displays the latest processing results.

**Important follow-up questions:**

-   How would you prevent duplicate product submissions?
-   How would you handle API rate limits?
-   How would you process 100,000 products?
-   How would you handle partial failures?
-   How would you retry failed API requests?
-   How would you monitor the integration?
-   How would you secure external API credentials?

------------------------------------------------------------------------

# 16. JD skills that need additional preparation

These technologies are in the JD but are not clearly demonstrated in the
resume.

  Technology        Questions to prepare
  ----------------- -----------------------------------------------------
  DB2               DB2 versus PostgreSQL, indexing, joins, query plans
  PostgreSQL        JSONB, MVCC, indexes, EXPLAIN ANALYZE
  AWS ECS           Tasks, services, clusters, Fargate
  AWS EC2           Instance types, security groups, AMIs
  AWS IAM           Roles, policies, least privilege
  AWS SNS           Pub/sub, fan-out, subscriptions
  AWS API Gateway   Authentication, throttling, Lambda integration
  DynamoDB          Partition key, sort key, GSI, hot partitions
  CloudWatch        Logs, metrics, alarms
  CloudFormation    Templates, stacks, drift
  SSM               Parameter Store, Session Manager
  Mockito           Mocking, stubbing, verification
  SonarQube         Quality gates, code smells
  Splunk            Searching logs, dashboards, alerts
  Kong              API gateway, plugins, rate limiting
  JIRA              Epics, stories, tasks, sprint management

Do not claim hands-on experience with these tools if you have only
studied them. Interviewers often ask implementation-level follow-ups.

------------------------------------------------------------------------

# 17. Top 30 questions to prioritize

1.  Explain your current Infosys project architecture.
2.  Explain your Claude-based self-healing cron job.
3.  Explain Java 8 Stream API with practical examples.
4.  How does HashMap work internally?
5.  HashMap vs ConcurrentHashMap?
6.  CompletableFuture vs Future?
7.  How does Spring Boot auto-configuration work?
8.  Explain dependency injection and bean lifecycle.
9.  Explain @Transactional and propagation.
10. How do you implement global exception handling?
11. Monolith vs microservices?
12. Explain circuit breaker and Saga patterns.
13. How do you handle distributed transactions?
14. How do Kafka partitions and consumer groups work?
15. Kafka vs SQS vs SNS?
16. How do you design idempotent APIs?
17. Explain React hooks and useEffect.
18. useMemo vs useCallback?
19. How do you optimize React performance?
20. How do you integrate React with Spring Boot?
21. Write a SQL query for the second-highest salary.
22. Explain indexes and query optimization.
23. Explain ACID and transaction isolation.
24. PostgreSQL vs DB2?
25. Explain ECS, EC2, and Fargate.
26. How do you secure AWS applications using IAM?
27. How would you deploy Spring Boot on AWS?
28. Explain JUnit, Mockito, and TDD.
29. Explain Docker and Kubernetes deployment.
30. How do you validate Claude-generated code?

------------------------------------------------------------------------

# 18. Seven-day preparation schedule

  -----------------------------------------------------------------------
  Day                     Topics                  Target
  ----------------------- ----------------------- -----------------------
  1                       Java 8, Collections,    25 questions + coding
                          Streams, Multithreading 

  2                       Spring Boot, REST APIs, 20 questions
                          Transactions, Security  

  3                       Microservices, Kafka,   20 questions +
                          distributed systems     architecture

  4                       React, JavaScript,      20 questions
                          hooks, performance      

  5                       SQL, DB2, PostgreSQL,   15 questions + SQL
                          query optimization      practice

  6                       AWS, Docker,            25 questions
                          Kubernetes, Jenkins     

  7                       JUnit, Mockito, Claude, Mock interview
                          project deep dive       
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 19. Final interview strategy

Prepare in three layers:

**Layer 1 -- Technical fundamentals**

Be able to answer Java 8, Spring Boot, React, SQL, and microservices
questions without hesitation.

**Layer 2 -- Practical implementation**

Be ready to write Java Stream programs, SQL queries, JUnit tests, React
hooks, REST APIs, and explain AWS deployment architecture.

**Layer 3 -- Resume-based cross-questioning**

Spend significant preparation time on:

-   Your Claude self-healing implementation.
-   Your Spring Boot microservices architecture.
-   Your Kafka integration.
-   Your Keycloak implementation.
-   Your SQL and EhCache performance improvements.
-   Your Google Ads and Google Merchant Center integration.

Interviewers may spend 20--30 minutes exploring one project. Know the
architecture, trade-offs, failure handling, and your exact contribution.

**Recommended order:** Start with the resume-based project deep dive,
particularly the Claude self-healing cron job, and then move into Java 8
and Spring Boot. These areas provide the strongest opportunity to
demonstrate the experience already documented in your resume.

------------------------------------------------------------------------

## Notes on research provenance

This is a JD- and resume-tailored preparation guide, not a verified
scrape of LinkedIn, Glassdoor, Indeed, AmbitionBox, or other review
sites. The answers combine the supplied resume details with general
technical interview knowledge. Validate version-specific details against
the actual Java, Spring Boot, React, database, and AWS versions used by
the prospective employer.
