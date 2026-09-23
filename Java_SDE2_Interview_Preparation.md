# Java SDE-2 Interview Preparation — Streams, Core Java, Concurrency, Coding & SQL

## Assumptions and model classes

The Stream examples below assume a simple `Employee` model:

```java
class Employee {
    private int id;
    private String name;
    private String department;
    private double salary;

    public Employee(int id, String name, String department, double salary) {
        this.id = id;
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }

    @Override
    public String toString() {
        return name + " - " + department + " - " + salary;
    }
}
```

For salary calculations in production financial code, prefer `BigDecimal` over `double`.

---

# Part 1 — Frequently Asked Streams Coding Questions

## 1. Find the second-highest salary

### Distinct salary values

```java
Optional<Double> secondHighest =
    employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst();
```

### Complexity

Sorting makes this approximately `O(n log n)`.

### Interview point

Ask whether "second-highest" means the second-highest **distinct** salary. `distinct()` handles that interpretation.

---

## 2. Find the highest salary in each department

```java
Map<String, Employee> highestByDepartment =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.collectingAndThen(
                Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary)),
                Optional::get
            )
        ));
```

If only the salary is required:

```java
Map<String, Double> highestSalaryByDepartment =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.collectingAndThen(
                Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary)),
                e -> e.get().getSalary()
            )
        ));
```

---

## 3. Find duplicate elements in a list

```java
Set<Integer> seen = new HashSet<>();

Set<Integer> duplicates =
    numbers.stream()
        .filter(n -> !seen.add(n))
        .collect(Collectors.toSet());
```

Example:

```text
Input:  [1, 2, 3, 2, 4, 1, 5]
Output: [1, 2]
```

### Important

`HashSet.add()` returns `false` when the element already exists.

---

## 4. Find the first non-repeated character in a string

A `LinkedHashMap` preserves insertion order.

```java
Optional<Character> firstNonRepeated =
    input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(),
            LinkedHashMap::new,
            Collectors.counting()
        ))
        .entrySet()
        .stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst();
```

For Unicode-heavy input, be aware that Java `char` represents a UTF-16 code unit rather than always a complete Unicode code point.

---

## 5. Find frequency of each character

```java
Map<Character, Long> frequency =
    input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        ));
```

To preserve character order:

```java
Map<Character, Long> frequency =
    input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(),
            LinkedHashMap::new,
            Collectors.counting()
        ));
```

---

## 6. Find frequency of each word in a sentence

```java
Map<String, Long> wordFrequency =
    Arrays.stream(sentence.toLowerCase().split("\\s+"))
        .collect(Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        ));
```

For punctuation-aware input, normalize punctuation first rather than treating `"java"` and `"java,"` as different words.

---

## 7. Find the first repeated character in a string

```java
Set<Character> seen = new HashSet<>();

Optional<Character> firstRepeated =
    input.chars()
        .mapToObj(c -> (char) c)
        .filter(c -> !seen.add(c))
        .findFirst();
```

Example:

```text
Input:  "swiss"
Output: 's'
```

---

## 8. Find employees whose salary is greater than average

```java
double average =
    employees.stream()
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0.0);

List<Employee> result =
    employees.stream()
        .filter(e -> e.getSalary() > average)
        .collect(Collectors.toList());
```

### Interview point

This uses two traversals: one for the average and one for filtering. That is usually clearer than trying to force the entire operation into one complicated stream.

---

## 9. Group employees by department

```java
Map<String, List<Employee>> employeesByDepartment =
    employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

Example result:

```text
IT        -> [Alice, Bob]
Finance   -> [Charlie]
HR        -> [David, Eve]
```

---

## 10. Sort employees by salary using Streams

Ascending:

```java
List<Employee> sorted =
    employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary))
        .collect(Collectors.toList());
```

Descending:

```java
List<Employee> sorted =
    employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
        .collect(Collectors.toList());
```

---

## 11. Find maximum/minimum element from a list

For integers:

```java
Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);

Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);
```

Or:

```java
OptionalInt max = numbers.stream()
    .mapToInt(Integer::intValue)
    .max();

OptionalInt min = numbers.stream()
    .mapToInt(Integer::intValue)
    .min();
```

For employees:

```java
Optional<Employee> highestPaid =
    employees.stream()
        .max(Comparator.comparingDouble(Employee::getSalary));
```

---

## 12. Find the top 3 highest salaries

```java
List<Double> top3 =
    employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .limit(3)
        .collect(Collectors.toList());
```

If duplicate salary values should count as separate employees, remove `distinct()`.

---

## 13. Find the Nth highest salary

For the Nth-highest **distinct** salary:

```java
int n = 3;

Optional<Double> nthHighest =
    employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(n - 1)
        .findFirst();
```

Validation:

```java
if (n <= 0) {
    throw new IllegalArgumentException("n must be greater than 0");
}
```

---

## 14. Partition numbers into even and odd

```java
Map<Boolean, List<Integer>> partitioned =
    numbers.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

Interpretation:

```text
true  -> even numbers
false -> odd numbers
```

---

## 15. Find sum/average of salaries

### Sum

```java
double sum =
    employees.stream()
        .mapToDouble(Employee::getSalary)
        .sum();
```

### Average

```java
double average =
    employees.stream()
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0.0);
```

---

## 16. Convert List<Employee> to Map<Id, Employee>

```java
Map<Integer, Employee> employeeMap =
    employees.stream()
        .collect(Collectors.toMap(
            Employee::getId,
            Function.identity()
        ));
```

### What if IDs are duplicated?

Provide a merge function:

```java
Map<Integer, Employee> employeeMap =
    employees.stream()
        .collect(Collectors.toMap(
            Employee::getId,
            Function.identity(),
            (existing, replacement) -> existing
        ));
```

Without a merge function, duplicate keys cause `IllegalStateException`.

---

## 17. Find employees whose names start with a particular character

```java
char prefix = 'A';

List<Employee> result =
    employees.stream()
        .filter(e -> e.getName().startsWith(String.valueOf(prefix)))
        .collect(Collectors.toList());
```

Case-insensitive:

```java
List<Employee> result =
    employees.stream()
        .filter(e -> e.getName().toLowerCase().startsWith(
            String.valueOf(prefix).toLowerCase()
        ))
        .collect(Collectors.toList());
```

---

## 18. Find the longest string in a list

```java
Optional<String> longest =
    strings.stream()
        .max(Comparator.comparingInt(String::length));
```

If you need deterministic tie handling, define what should happen when multiple strings have the same maximum length.

---

## 19. Remove duplicates from a list using Streams

```java
List<Integer> unique =
    numbers.stream()
        .distinct()
        .collect(Collectors.toList());
```

`distinct()` preserves encounter order for an ordered sequential stream.

---

## 20. Find common elements between two lists

If duplicates are not important:

```java
Set<Integer> second =
    new HashSet<>(list2);

List<Integer> common =
    list1.stream()
        .filter(second::contains)
        .distinct()
        .collect(Collectors.toList());
```

This is generally preferable to repeatedly calling `list2.contains()` when `list2` is large.

---

# Part 2 — Stream API Conceptual Questions

## 21. Stream vs Collection

| Collection | Stream |
|---|---|
| Stores data | Processes data |
| Can usually be traversed multiple times | Usually consumed once |
| Eager data structure | Lazy processing pipeline |
| Supports adding/removing elements depending on type | Does not store elements |
| Example: `List`, `Set`, `Map` | Example: `list.stream()` |

A Stream is not a replacement for a Collection. A collection represents data; a stream represents a computation over data.

---

## 22. Intermediate vs terminal operations

### Intermediate operations

Examples:

```java
filter()
map()
flatMap()
sorted()
distinct()
limit()
skip()
peek()
```

They return another Stream and are generally lazy.

### Terminal operations

Examples:

```java
collect()
forEach()
reduce()
count()
min()
max()
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
```

A terminal operation triggers stream processing.

Example:

```java
List<String> result =
    names.stream()
        .filter(name -> name.startsWith("A"))
        .map(String::toUpperCase)
        .collect(Collectors.toList());
```

`filter()` and `map()` are intermediate; `collect()` is terminal.

---

## 23. map() vs flatMap()

### `map()`

One input produces one output.

```java
List<String> names = List.of("alice", "bob");

List<Integer> lengths =
    names.stream()
        .map(String::length)
        .collect(Collectors.toList());
```

Conceptually:

```text
Stream<T> -> Stream<R>
```

### `flatMap()`

One input can produce multiple outputs, and nested streams are flattened.

```java
List<List<Integer>> nested =
    List.of(
        List.of(1, 2),
        List.of(3, 4)
    );

List<Integer> flattened =
    nested.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());
```

Conceptually:

```text
Stream<Stream<T>> -> Stream<T>
```

---

## 24. filter() vs map()

`filter()` decides whether an element remains.

```java
numbers.stream()
    .filter(n -> n > 10);
```

`map()` transforms an element.

```java
numbers.stream()
    .map(n -> n * 2);
```

Think:

```text
filter = keep/remove
map    = transform
```

---

## 25. map() vs peek()

`map()` is intended for transformation:

```java
List<String> result =
    names.stream()
        .map(String::toUpperCase)
        .collect(Collectors.toList());
```

`peek()` is primarily intended for observing elements while debugging or inspecting a pipeline:

```java
names.stream()
    .peek(System.out::println)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

Do not use `peek()` as a substitute for business logic or required side effects.

---

## 26. findFirst() vs findAny()

`findFirst()` returns the first element according to the stream's encounter order when one exists.

```java
Optional<Integer> result =
    numbers.stream().findFirst();
```

`findAny()` returns some element and is particularly useful when encounter order is not important, including parallel processing.

```java
Optional<Integer> result =
    numbers.parallelStream().findAny();
```

For an ordered sequential stream, `findAny()` often appears to return the first element, but code should not rely on that behavior as a semantic guarantee.

---

## 27. reduce() vs collect()

`reduce()` combines stream elements into a single result.

```java
int sum =
    numbers.stream()
        .reduce(0, Integer::sum);
```

`collect()` is designed for mutable reduction into result containers such as lists, maps, and grouped structures.

```java
List<Integer> result =
    numbers.stream()
        .filter(n -> n > 10)
        .collect(Collectors.toList());
```

Rule of thumb:

```text
reduce  -> combine into one value
collect -> accumulate into a container/result structure
```

---

## 28. groupingBy() vs partitioningBy()

`groupingBy()` groups by arbitrary keys:

```java
Map<String, List<Employee>> result =
    employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

`partitioningBy()` divides elements into two groups based on a boolean predicate:

```java
Map<Boolean, List<Integer>> result =
    numbers.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

Conceptually:

```text
groupingBy     -> many possible groups
partitioningBy -> two boolean groups
```

---

## 29. What is lazy evaluation in Streams?

Intermediate operations are not executed immediately.

```java
Stream<Integer> stream =
    numbers.stream()
        .filter(n -> {
            System.out.println(n);
            return n > 10;
        });
```

Nothing is printed yet.

Processing starts when a terminal operation is invoked:

```java
long count = stream.count();
```

Laziness enables stream implementations to fuse operations and short-circuit in suitable cases.

For example:

```java
Optional<Integer> result =
    numbers.stream()
        .filter(n -> n > 10)
        .findFirst();
```

The stream does not necessarily process every element.

---

## 30. Can a Stream be reused after a terminal operation?

No.

```java
Stream<Integer> stream = numbers.stream();

stream.count();

stream.forEach(System.out::println); // IllegalStateException
```

Create a new stream from the source:

```java
numbers.stream().count();
numbers.stream().forEach(System.out::println);
```

---

## 31. Sequential Stream vs Parallel Stream

Sequential:

```java
numbers.stream()
    .map(this::process)
    .collect(Collectors.toList());
```

Parallel:

```java
numbers.parallelStream()
    .map(this::process)
    .collect(Collectors.toList());
```

A parallel stream can split work across threads, commonly using the common `ForkJoinPool`.

Parallelism is not automatically faster. It depends on workload size, operation cost, source characteristics, CPU availability, ordering requirements, contention, and other factors.

---

## 32. When should you NOT use parallelStream()?

Avoid blindly using it when:

- The collection is small.
- Each operation is very cheap.
- Work is I/O-bound.
- Operations mutate shared state.
- Thread safety is not guaranteed.
- Ordering is important and expensive to maintain.
- The application already has significant thread-pool contention.
- Performance has not been measured.

Prefer profiling/benchmarking over assuming parallelism is beneficial.

---

## 33. How does distinct() work?

`distinct()` removes duplicate elements according to `equals()` semantics, with hash-based state used by the stream implementation.

For objects, `equals()` and `hashCode()` therefore matter.

```java
List<Integer> result =
    numbers.stream()
        .distinct()
        .collect(Collectors.toList());
```

For custom objects, implement equality consistently if logical duplicates should be removed.

---

## 34. How does sorted() work internally?

`sorted()` is a stateful intermediate operation.

It must generally buffer elements before it can produce the correctly ordered result. Therefore, unlike `filter()`, it cannot usually process and emit each element independently.

Example:

```java
List<Integer> result =
    numbers.stream()
        .sorted()
        .collect(Collectors.toList());
```

Typical comparison-based sorting is `O(n log n)`.

In a parallel stream, sorting can involve additional partitioning and merging work.

---

## 35. What happens when a Stream operation throws an exception?

The exception propagates out of the stream pipeline unless it is handled inside the pipeline or by the caller.

Example:

```java
try {
    numbers.stream()
        .map(n -> 10 / n)
        .collect(Collectors.toList());
} catch (ArithmeticException e) {
    System.out.println("Division by zero");
}
```

Streams do not automatically recover from exceptions.

Checked exceptions also require special handling because standard functional interfaces such as `Function<T,R>` do not declare checked exceptions.

---

# Part 3 — Deloitte-Style Java Interview Questions

## Core Java & Java 8

## 36. What are the key features introduced in Java 8?

Major Java 8 features include:

1. Lambda expressions
2. Functional interfaces
3. Stream API
4. Default and static methods in interfaces
5. Method and constructor references
6. `Optional`
7. New Date/Time API (`java.time`)
8. `CompletableFuture`
9. Improved type inference and related language/API enhancements
10. Nashorn JavaScript engine (introduced in Java 8 and later removed from the JDK)

Example lambda:

```java
List<Integer> numbers = List.of(1, 2, 3);

numbers.forEach(n -> System.out.println(n));
```

---

## 37. Difference between map() and flatMap()

### `map()`

Transforms each element:

```java
List<String> names = List.of("A", "BB", "CCC");

List<Integer> lengths =
    names.stream()
        .map(String::length)
        .collect(Collectors.toList());
```

Result:

```text
[1, 2, 3]
```

### `flatMap()`

Transforms and flattens nested structures:

```java
List<List<String>> data =
    List.of(
        List.of("A", "B"),
        List.of("C", "D")
    );

List<String> result =
    data.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());
```

Result:

```text
[A, B, C, D]
```

---

## 38. Intermediate and terminal Stream operations

### Intermediate

```text
filter
map
flatMap
distinct
sorted
limit
skip
peek
```

Return another Stream and are generally lazy.

### Terminal

```text
collect
reduce
forEach
count
min
max
findFirst
findAny
anyMatch
allMatch
noneMatch
```

Trigger execution.

---

## 39. Functional interface vs marker interface

### Functional interface

Has exactly one abstract method.

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

Can be used with a lambda:

```java
Calculator add = (a, b) -> a + b;
```

A functional interface may still have `default` and `static` methods; those do not count as abstract methods.

### Marker interface

Contains no methods and is used to communicate metadata/capability.

Examples include:

```java
Serializable
Cloneable
```

A marker interface does not become a functional interface simply because it is used as a type.

---

## 40. Can we overload main()? Which one does JVM execute?

Yes, `main()` can be overloaded.

```java
public static void main(String[] args) {
    System.out.println("JVM entry point");
}

public static void main(int x) {
    System.out.println(x);
}
```

The JVM looks for the conventional entry point:

```java
public static void main(String[] args)
```

(or the equivalent varargs declaration).

The overloaded `main(int x)` is an ordinary method and is not selected as the application entry point.

---

## 41. Difference between this and super

### `this`

Refers to the current object.

```java
this.name = name;
```

It can also invoke another constructor in the same class:

```java
this("default");
```

### `super`

Refers to the superclass portion of the current object.

```java
super.display();
```

It can invoke a superclass constructor:

```java
super();
```

Key distinction:

```text
this  -> current class/object
super -> parent class
```

A constructor call using `this(...)` or `super(...)` must be the first statement in the constructor.

---

## 42. How do you create and handle custom exceptions?

### Checked custom exception

```java
class InvalidSalaryException extends Exception {
    public InvalidSalaryException(String message) {
        super(message);
    }
}
```

Use:

```java
void validateSalary(double salary)
        throws InvalidSalaryException {

    if (salary < 0) {
        throw new InvalidSalaryException("Salary cannot be negative");
    }
}
```

Handle:

```java
try {
    validateSalary(-100);
} catch (InvalidSalaryException e) {
    System.out.println(e.getMessage());
}
```

### Unchecked custom exception

```java
class InvalidSalaryException extends RuntimeException {
    public InvalidSalaryException(String message) {
        super(message);
    }
}
```

Use a checked exception when callers are expected to explicitly handle/recover from the condition; use an unchecked exception for programming errors or conditions that do not normally require mandatory caller handling.

---

# Part 4 — Multithreading & Concurrency

## 43. Runnable vs Callable

### Runnable

```java
Runnable task = () -> {
    System.out.println("Running");
};
```

- Does not return a result.
- `run()` cannot declare checked exceptions.

### Callable

```java
Callable<Integer> task = () -> {
    return 42;
};
```

- Returns a result.
- `call()` can throw checked exceptions.

Example:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Future<Integer> future = executor.submit(() -> 42);

Integer result = future.get();

executor.shutdown();
```

---

## 44. start() vs run()

```java
Thread thread = new Thread(() -> {
    System.out.println("Work");
});

thread.start();
```

`start()` asks the JVM to start a new thread, after which that thread executes `run()`.

Calling:

```java
thread.run();
```

directly is just a normal method invocation and does not start a new thread.

---

## 45. wait() vs sleep()

| `wait()` | `sleep()` |
|---|---|
| Defined by `Object` | Defined by `Thread` |
| Used for thread coordination | Used to pause execution |
| Must be called while owning the object's monitor | Does not require owning a monitor |
| Releases the object's monitor while waiting | Does not release monitors held by the thread |
| Usually paired with `notify()` / `notifyAll()` | No notification mechanism |

Example:

```java
synchronized (lock) {
    lock.wait();
}
```

versus:

```java
Thread.sleep(1000);
```

`wait()` can also wake spuriously, so condition waiting should normally be expressed with a loop that re-checks the condition.

---

## 46. What is synchronization in Java?

Synchronization controls access to shared mutable state so that operations that must be mutually exclusive do not execute concurrently.

Example:

```java
public synchronized void increment() {
    count++;
}
```

Or:

```java
synchronized (lock) {
    count++;
}
```

Synchronization also establishes visibility/happens-before relationships under the Java Memory Model.

For concurrent counters, alternatives such as `AtomicInteger` may be more appropriate:

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

---

## 47. Explain Java memory areas: Heap, Stack, Method Area, Program Counter

### Heap

Stores objects and arrays.

```java
Employee e = new Employee(...);
```

The `Employee` object is allocated on the heap.

### Stack

Each thread has its own JVM stack. Stack frames contain method-local information such as local variables and operand-stack state.

### Method Area

The JVM specification defines a method area for per-class/per-interface structures. In HotSpot, class metadata is implemented using **Metaspace** in native memory.

### Program Counter (PC)

Each JVM thread has its own program counter register, which indicates the current JVM instruction being executed.

### Important interview distinction

Do not say that every variable lives on the heap or every primitive lives on the stack. The JVM specification defines runtime areas and implementation details can vary.

---

## 48. What is garbage collection? What is the purpose of finalize()?

### Garbage Collection

The JVM automatically identifies objects that are no longer reachable and reclaims memory associated with them.

Example:

```java
Employee e = new Employee(...);

e = null;
```

The object may eventually become eligible for garbage collection if no other reachable reference exists.

### `finalize()`

`Object.finalize()` was an old mechanism intended to allow cleanup before object reclamation, but it was unreliable and has been deprecated for removal. Modern Java code should not depend on it.

For resources such as files, sockets, and database connections, use explicit resource management:

```java
try (BufferedReader reader =
         new BufferedReader(new FileReader("data.txt"))) {

    // use resource

}
```

The `try-with-resources` construct calls `close()` automatically.

---

# Part 5 — Coding & SQL

## 49. Find the minimum-length subarray whose sum equals a target

This question needs an important assumption.

### If all numbers are positive

Use a sliding-window algorithm rather than Streams. This is a classic case where Streams are not necessarily the best interview tool.

```java
public static int minSubarrayLength(int[] nums, int target) {
    int left = 0;
    int sum = 0;
    int minLength = Integer.MAX_VALUE;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];

        while (sum >= target) {
            if (sum == target) {
                minLength = Math.min(minLength, right - left + 1);
            }

            sum -= nums[left++];
        }
    }

    return minLength == Integer.MAX_VALUE ? 0 : minLength;
}
```

Example:

```text
Input:
nums   = [2, 3, 1, 2, 4, 3]
target = 7

Output:
2
```

The subarray `[4, 3]` has sum 7 and length 2.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

### Important caveat

The sliding-window solution above assumes positive numbers. If negative numbers are allowed, use a different technique such as prefix sums plus a suitable data structure/algorithm.

---

# Part 6 — SQL Interview Questions

## 50. Write an SQL query to rank employees based on experience

Assume:

```text
employee(id, name, experience_years, salary)
```

Rank employees by experience, highest first:

```sql
SELECT
    id,
    name,
    experience_years,
    RANK() OVER (
        ORDER BY experience_years DESC
    ) AS experience_rank
FROM employee;
```

If you want the rank to continue without gaps after ties:

```sql
SELECT
    id,
    name,
    experience_years,
    DENSE_RANK() OVER (
        ORDER BY experience_years DESC
    ) AS experience_rank
FROM employee;
```

If every row must get a unique sequential number:

```sql
SELECT
    id,
    name,
    experience_years,
    ROW_NUMBER() OVER (
        ORDER BY experience_years DESC, id
    ) AS experience_rank
FROM employee;
```

---

## 51. Difference between RANK(), DENSE_RANK(), and ROW_NUMBER()

Suppose experience is:

```text
10
10
8
5
```

### RANK()

```text
10 -> 1
10 -> 1
8  -> 3
5  -> 4
```

Ties receive the same rank and create gaps.

### DENSE_RANK()

```text
10 -> 1
10 -> 1
8  -> 2
5  -> 3
```

Ties receive the same rank without gaps.

### ROW_NUMBER()

```text
10 -> 1
10 -> 2
8  -> 3
5  -> 4
```

Every row receives a unique number.

### Summary

| Function | Same rank for ties? | Gaps after ties? |
|---|---:|---:|
| `RANK()` | Yes | Yes |
| `DENSE_RANK()` | Yes | No |
| `ROW_NUMBER()` | No | No |

For deterministic `ROW_NUMBER()`, include a unique tie-breaker such as `id` in the `ORDER BY`.

---

## 52. Explain the order of execution of SQL query clauses

A simplified logical processing order is:

```text
1. FROM
2. JOIN / ON
3. WHERE
4. GROUP BY
5. HAVING
6. SELECT
7. DISTINCT
8. ORDER BY
9. LIMIT / OFFSET
```

Example:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employee
WHERE salary > 50000
GROUP BY department
HAVING AVG(salary) > 70000
ORDER BY avg_salary DESC
LIMIT 5;
```

Conceptually:

1. `FROM` identifies the source rows.
2. `WHERE` filters individual rows.
3. `GROUP BY` forms groups.
4. `HAVING` filters groups.
5. `SELECT` produces the result columns.
6. `DISTINCT` removes duplicate result rows when requested.
7. `ORDER BY` sorts the result.
8. `LIMIT/OFFSET` restricts the final result.

### Interview trap

Because `WHERE` logically occurs before `GROUP BY`, you cannot normally use an aggregate result such as `AVG(salary)` directly in `WHERE`. Use `HAVING` for filtering groups based on aggregates.

---

# Part 7 — Quick Interview Cheat Sheet

## Streams

```text
filter()       -> keep elements matching predicate
map()          -> transform elements
flatMap()      -> transform + flatten
distinct()     -> remove duplicates
sorted()       -> sort elements
limit()        -> take first N
skip()         -> skip first N
peek()         -> inspect elements; mainly debugging
collect()      -> gather results
reduce()       -> combine into one value
findFirst()    -> first element by encounter order
findAny()      -> any element
```

## Collectors

```text
groupingBy()       -> group by key
partitioningBy()   -> split into true/false groups
toMap()            -> build a Map
counting()         -> count elements
joining()          -> concatenate strings
mapping()          -> map values inside a collector
summarizingDouble()-> count/sum/min/max/average
```

## Concurrency

```text
Runnable  -> no result
Callable  -> result + checked exceptions
start()   -> starts a new thread
run()     -> ordinary method call when invoked directly
wait()    -> releases monitor and waits
sleep()   -> pauses; does not release held monitors
```

## SQL ranking

```text
RANK()        -> ties + gaps
DENSE_RANK()  -> ties + no gaps
ROW_NUMBER()  -> unique sequential numbers
```

---

# Part 8 — Common SDE-2 Follow-Up Questions

After solving a Streams problem, be ready for these follow-ups:

### 1. What is the time complexity?

For example:

```java
stream.sorted()
```

typically requires `O(n log n)` comparison-based sorting work.

### 2. Can this be done without sorting?

For second-highest or Nth-highest problems, yes. A single-pass algorithm can reduce the time complexity in some cases, at the cost of more state and more complicated handling of duplicates.

### 3. What happens with duplicate salaries?

Clarify whether the interviewer wants:

- Nth employee by salary position, or
- Nth distinct salary.

### 4. What happens with an empty list?

Use `Optional` where the result may not exist:

```java
Optional<Double> max = ...
```

rather than assuming a value exists.

### 5. Is parallelStream() automatically faster?

No. Benchmark the actual workload.

### 6. Is Stream API always better than a loop?

No. Streams are often expressive for transformations and aggregations, but loops can be clearer for stateful algorithms, complex control flow, early mutation, or algorithms such as sliding windows.

### 7. Can streams modify the source collection?

A stream pipeline should generally be non-interfering. Do not structurally modify the source while traversing it.

Bad pattern:

```java
list.stream()
    .forEach(x -> list.remove(x));
```

Prefer a dedicated operation such as:

```java
list.removeIf(x -> condition(x));
```

---

# Final Revision Checklist

Before an SDE-2 Java interview, make sure you can explain and code:

- [ ] `filter`, `map`, `flatMap`
- [ ] `reduce` vs `collect`
- [ ] `groupingBy` and `partitioningBy`
- [ ] `toMap` and duplicate-key handling
- [ ] `distinct`
- [ ] `sorted`
- [ ] `findFirst` vs `findAny`
- [ ] Lazy evaluation
- [ ] Stream reuse rules
- [ ] Sequential vs parallel streams
- [ ] `Optional`
- [ ] Functional interfaces and lambdas
- [ ] `Runnable` vs `Callable`
- [ ] `start()` vs `run()`
- [ ] `wait()` vs `sleep()`
- [ ] Synchronization and Java Memory Model basics
- [ ] Heap/stack/method-area/PC concepts
- [ ] Garbage collection and modern resource management
- [ ] Sliding-window algorithms
- [ ] SQL window functions
- [ ] SQL logical query processing order
- [ ] Complexity analysis and edge cases

The strongest interview answers do not just produce working code: they clarify assumptions, discuss edge cases, explain complexity, and justify why a particular approach is appropriate.
