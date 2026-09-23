# Java, Spring Boot, SQL, and React Interview Questions & Answers

## 1. Comparable vs Comparator

### Comparable
`Comparable` is used when a class defines its **natural/default ordering**. Implement `Comparable<T>` and override `compareTo()`.

```java
class Employee implements Comparable<Employee> {
    int id;
    String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public int compareTo(Employee e) {
        return Integer.compare(this.id, e.id);
    }
}
```

Usage:

```java
Collections.sort(employees);
```

### Comparator
`Comparator` is used for **custom or multiple sorting strategies** without changing the original class.

```java
Comparator<Employee> byName =
    (e1, e2) -> e1.name.compareTo(e2.name);

Collections.sort(employees, byName);
```

| Comparable | Comparator |
|---|---|
| `java.lang.Comparable` | `java.util.Comparator` |
| `compareTo()` | `compare()` |
| Logic inside class | Logic outside class |
| Natural ordering | Multiple custom orderings |

**Interview answer:** Comparable defines natural ordering inside the class; Comparator defines custom or multiple sorting strategies outside the class.

---

## 2. Internal Working of HashSet

`HashSet` internally uses a `HashMap`.

Conceptually:

```java
HashMap<E, Object> map;
```

When:

```java
set.add("Java");
```

HashSet internally performs something similar to:

```java
map.put("Java", PRESENT);
```

### Flow
1. Call `hashCode()`.
2. Determine the bucket.
3. Check the bucket.
4. If needed, use `equals()` to detect duplicates.

```text
add(object)
    ↓
hashCode()
    ↓
Find bucket
    ↓
Existing entry?
    ↓
Yes → equals()
    ↓
Equal → Don't add
Not equal → Collision; both may exist
```

### Important contract

> If two objects are equal according to `equals()`, they must have the same `hashCode()`.

Average complexity:
- `add()` → O(1)
- `remove()` → O(1)
- `contains()` → O(1)

---

## 3. ConcurrentHashMap vs SynchronizedMap

### SynchronizedMap

```java
Map<Integer, String> map =
    Collections.synchronizedMap(new HashMap<>());
```

It provides thread safety by synchronizing access.

### ConcurrentHashMap

```java
Map<Integer, String> map =
    new ConcurrentHashMap<>();
```

It is designed for high-concurrency access and scales better.

| SynchronizedMap | ConcurrentHashMap |
|---|---|
| Wrapper around a map | Concurrent collection |
| Coarser synchronization | Fine-grained/concurrent mechanisms |
| Lower concurrency | Higher concurrency |
| Allows nulls according to wrapped map | No null keys or values |
| External synchronization needed for safe iteration patterns | Weakly consistent iterators |

Useful atomic methods:

```java
map.putIfAbsent(key, value);
map.computeIfAbsent(key, k -> createValue());
map.compute(key, (k, v) -> v == null ? 1 : v + 1);
```

**Interview answer:** ConcurrentHashMap is designed for scalable concurrent access and provides atomic operations, while a synchronized map provides thread safety through synchronization and generally has lower concurrency.

---

## 4. Fail-Fast vs Fail-Safe / Weakly Consistent Iterator

### Fail-Fast
A fail-fast iterator detects structural modification during iteration and may throw `ConcurrentModificationException`.

```java
List<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);

for (Integer n : list) {
    list.add(30); // ConcurrentModificationException
}
```

Examples:
- `ArrayList`
- `HashMap`
- `HashSet`
- `LinkedList`

### Fail-Safe / Weakly Consistent
Concurrent collections can tolerate concurrent modifications during iteration.

Examples:
- `ConcurrentHashMap`
- `CopyOnWriteArrayList`

Strictly speaking, **fail-safe** is informal terminology. `ConcurrentHashMap` provides **weakly consistent iterators**.

**Interview answer:** Fail-fast iterators detect structural modifications and may throw `ConcurrentModificationException`, whereas concurrent collection iterators can tolerate concurrent modifications without that exception.

---

## 5. Immutable Class in Java

An immutable object's state cannot change after creation.

Example: `String`.

Rules for an immutable class:
1. Make the class `final`.
2. Make fields `private final`.
3. Initialize fields in the constructor.
4. Do not provide setters.
5. Use defensive copies for mutable fields.

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

---

## 6. What if an Immutable Class Has an ArrayList Field?

Making an `ArrayList` reference `final` is not enough. The contents can still change.

### Incorrect

```java
final class Employee {
    private final ArrayList<String> skills;

    public Employee(ArrayList<String> skills) {
        this.skills = skills;
    }

    public ArrayList<String> getSkills() {
        return skills;
    }
}
```

### Correct: Defensive copies

```java
final class Employee {
    private final List<String> skills;

    public Employee(List<String> skills) {
        this.skills = new ArrayList<>(skills);
    }

    public List<String> getSkills() {
        return new ArrayList<>(skills);
    }
}
```

A modern alternative:

```java
public List<String> getSkills() {
    return List.copyOf(skills);
}
```

**Key point:** `final` makes the reference non-reassignable; it does not make the referenced `ArrayList` immutable.

---

## 7. Checked vs Unchecked Exceptions

### Checked Exceptions
Checked by the compiler. Must be handled or declared.

```java
public void readFile() throws IOException {
    FileReader file = new FileReader("test.txt");
}
```

Examples:
- `IOException`
- `SQLException`
- `FileNotFoundException`

### Unchecked Exceptions
Not required to be handled or declared. Usually programming errors and extend `RuntimeException`.

Examples:
- `NullPointerException`
- `ArithmeticException`
- `IllegalArgumentException`
- `ArrayIndexOutOfBoundsException`

| Checked | Unchecked |
|---|---|
| Compiler-enforced handling/declaration | No mandatory handling |
| Compile-time checking | Occurs at runtime |
| Usually extends `Exception` directly | Extends `RuntimeException` |

---

## 8. Predicate in Java

`Predicate<T>` is a functional interface used to test a condition and return `true` or `false`.

```java
Predicate<Integer> isEven = n -> n % 2 == 0;

System.out.println(isEven.test(10)); // true
```

Main method:

```java
boolean test(T t);
```

Example with streams:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(System.out::println);
```

Useful methods:
- `and()`
- `or()`
- `negate()`

| Interface | Input | Return |
|---|---|---|
| `Predicate<T>` | T | boolean |
| `Function<T, R>` | T | R |
| `Consumer<T>` | T | void |
| `Supplier<T>` | None | T |

---

## 9. Singleton Design Pattern

Singleton ensures that only **one instance** of a class exists and provides global access to it.

### Basic lazy Singleton

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### Holder Pattern

```java
public class Singleton {

    private Singleton() {
    }

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Enum Singleton

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing something");
    }
}
```

**Interview answer:** Singleton is a creational pattern ensuring one instance of a class and providing a global access point.

---

## 10. Runnable vs Callable

### Runnable

Used for tasks that do not return a result.

```java
Runnable task = () -> {
    System.out.println("Task is running");
};
```

Main method:

```java
void run();
```

### Callable

Used for tasks that return a result and can throw checked exceptions.

```java
Callable<Integer> task = () -> 10 + 20;
```

Main method:

```java
V call() throws Exception;
```

With `ExecutorService`:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Callable<Integer> task = () -> 10 + 20;
Future<Integer> future = executor.submit(task);

System.out.println(future.get());

executor.shutdown();
```

| Runnable | Callable |
|---|---|
| `run()` | `call()` |
| No return value | Returns a value |
| No checked exceptions directly | Can throw checked exceptions |
| Can run with `Thread` | Usually used with `ExecutorService` |

---

## 11. Double-Checked Locking

Used for a thread-safe lazy Singleton while avoiding synchronization after initialization.

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

### Why two checks?
- First check avoids synchronization once initialized.
- Second check prevents multiple instances when multiple threads race.

### Why `volatile`?

```java
private static volatile Singleton instance;
```

It ensures proper visibility and safe publication under the Java Memory Model.

---

## 12. Does clone() Return a New Object?

Yes. `clone()` generally creates a new object, but `Object.clone()` performs a **shallow copy** by default.

```java
Employee e2 = (Employee) e1.clone();

System.out.println(e1 == e2); // false
```

For nested mutable objects, references may still point to the same object.

### Deep copy example

```java
@Override
public Employee clone() throws CloneNotSupportedException {
    Employee copy = (Employee) super.clone();
    copy.skills = new ArrayList<>(this.skills);
    return copy;
}
```

A class normally needs to implement `Cloneable`, otherwise `super.clone()` throws `CloneNotSupportedException`.

---

## 13. How to Unit Test a Private Method in JUnit

Best practice: **Do not directly test private methods.**

Test them through the public API.

```java
class Calculator {

    public int calculate(int a, int b) {
        return add(a, b);
    }

    private int add(int a, int b) {
        return a + b;
    }
}
```

Test:

```java
@Test
void shouldCalculateSum() {
    Calculator calculator = new Calculator();

    assertEquals(10, calculator.calculate(4, 6));
}
```

Reflection can technically access private methods, but it is generally discouraged.

**Interview answer:** Private methods are implementation details. Test observable behavior through public methods. If a private method contains substantial independent logic, consider extracting it into a separate class.

---

## 14. Second Highest Salary in SQL

### Using MAX()

```sql
SELECT MAX(salary) AS second_highest
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

### Using DENSE_RANK()

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employee
) t
WHERE rnk = 2;
```

### MySQL/PostgreSQL

```sql
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

`DENSE_RANK()` handles duplicate salaries correctly when finding the second **distinct** highest salary.

---

## 15. Default Fetch Type for One-to-Many

In JPA:

> `@OneToMany` defaults to `FetchType.LAZY`.

```java
@OneToMany
private List<Employee> employees;
```

Equivalent to:

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Employee> employees;
```

| Relationship | Default Fetch Type |
|---|---|
| `@OneToOne` | EAGER |
| `@ManyToOne` | EAGER |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

---

## 16. @SpringBootApplication

`@SpringBootApplication` is the main Spring Boot application annotation.

```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

It combines:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

### Responsibilities
- `@Configuration`: configuration and bean definitions.
- `@EnableAutoConfiguration`: automatic configuration based on dependencies.
- `@ComponentScan`: scans for components such as `@Service`, `@Repository`, and `@Controller`.

---

## 17. Changing Spring Boot Response from JSON to XML

Add the Jackson XML dependency:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

Specify XML:

```java
@GetMapping(
    value = "/employees",
    produces = MediaType.APPLICATION_XML_VALUE
)
public Employee getEmployee() {
    return new Employee(1, "John");
}
```

Clients can also request:

```http
Accept: application/xml
```

Spring uses the appropriate HTTP message converter.

---

## 18. POST vs PUT

### POST
Usually used to create a resource.

```http
POST /employees
```

Typically non-idempotent.

### PUT
Used to create or completely replace/update a resource at a known URI.

```http
PUT /employees/101
```

PUT is idempotent: repeating the same request should lead to the same final state.

| POST | PUT |
|---|---|
| Usually creates | Creates or replaces/updates at known URI |
| Usually collection URI | Usually specific resource URI |
| Non-idempotent | Idempotent |
| Server commonly assigns ID | URI identifies resource |

---

## 19. State vs Props in React

### Props
Data passed from parent to child. Props are read-only from the child's perspective.

```jsx
function Child({ name }) {
    return <h1>Hello {name}</h1>;
}

function Parent() {
    return <Child name="John" />;
}
```

### State
Data managed by a component that can change over time.

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            {count}
        </button>
    );
}
```

| Props | State |
|---|---|
| Passed from parent | Managed by component |
| Read-only | Updated via state setter |
| Used to configure/pass data | Used for changing data |

---

## 20. Class Component in React

A class component extends `React.Component`.

```jsx
class Welcome extends React.Component {
    render() {
        return <h1>Hello {this.props.name}</h1>;
    }
}
```

State example:

```jsx
class Counter extends React.Component {

    constructor(props) {
        super(props);

        this.state = {
            count: 0
        };
    }

    increment = () => {
        this.setState({
            count: this.state.count + 1
        });
    };

    render() {
        return (
            <button onClick={this.increment}>
                {this.state.count}
            </button>
        );
    }
}
```

Lifecycle methods include:
- `componentDidMount()`
- `componentDidUpdate()`
- `componentWillUnmount()`

---

## 21. Class Component vs Functional Component

| Class Component | Functional Component |
|---|---|
| ES6 class | JavaScript function |
| `this.props` | `props` |
| `this.state` | `useState()` |
| `this.setState()` | State setter |
| Lifecycle methods | Hooks such as `useEffect()` |
| Older style | Modern preferred style |

Functional component:

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            {count}
        </button>
    );
}
```

Modern React generally favors functional components and Hooks.

---

## 22. React Unnecessary Re-Renders and Optimization

### Parent re-render

Use `React.memo` when appropriate:

```jsx
const Child = React.memo(function Child() {
    return <div>Child</div>;
});
```

### Object reference changes

Avoid creating unnecessary objects on every render when referential stability matters:

```jsx
const user = useMemo(() => ({
    name: "John"
}), []);
```

### Function reference changes

```jsx
const handleClick = useCallback(() => {
    console.log("hello");
}, []);
```

This is most useful when passing callbacks to memoized children.

### Important
Do not use `React.memo`, `useMemo`, or `useCallback` everywhere blindly. Use React DevTools Profiler to identify real rendering bottlenecks first.

---

## 23. Find the Kth Smallest Element in an Unsorted Unique Array

### Sorting approach

```java
int[] arr = {7, 2, 9, 4, 1, 5};
int k = 3;

Arrays.sort(arr);

System.out.println(arr[k - 1]); // 4
```

Time complexity: **O(n log n)**.

### QuickSelect approach

Average time complexity: **O(n)**.

```java
public static int kthSmallest(int[] arr, int k) {
    if (k < 1 || k > arr.length) {
        throw new IllegalArgumentException("Invalid k");
    }

    int left = 0;
    int right = arr.length - 1;
    int target = k - 1;

    while (left <= right) {
        int pivotIndex = partition(arr, left, right);

        if (pivotIndex == target) {
            return arr[pivotIndex];
        } else if (pivotIndex > target) {
            right = pivotIndex - 1;
        } else {
            left = pivotIndex + 1;
        }
    }

    throw new IllegalStateException();
}

private static int partition(int[] arr, int left, int right) {
    int pivot = arr[right];
    int i = left;

    for (int j = left; j < right; j++) {
        if (arr[j] <= pivot) {
            swap(arr, i, j);
            i++;
        }
    }

    swap(arr, i, right);
    return i;
}

private static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[i] = temp;
}
```

> Note: The `swap` method above should use the standard implementation below:

```java
private static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

**Interview answer:** Sort for a simple O(n log n) solution, or use QuickSelect for O(n) average time when better performance is required.
