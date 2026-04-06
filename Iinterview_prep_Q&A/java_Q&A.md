==============================
FILE: Java — Language-Specific Interview Questions
==============================

### HIGH PRIORITY

---

Q1. How does Java's memory model work? Explain the heap, stack, and method area.

A1.
Java memory is divided into several runtime data areas managed by the JVM.

The **heap** is where all objects and arrays are allocated. It's shared across all threads and managed by the garbage collector. The heap is further divided into the Young Generation (Eden + two Survivor spaces) and the Old Generation (Tenured space). New objects go to Eden; surviving objects are promoted to the Old Generation after several GC cycles.

The **stack** is per-thread. Each method invocation pushes a stack frame containing local variables, operand stack, and a reference to the runtime constant pool. Primitive values and object references (not the objects themselves) live on the stack. Stack frames are popped when methods return — no GC involved, very fast.

The **method area** (Metaspace in Java 8+) stores class-level data: bytecode, method definitions, field definitions, the runtime constant pool, and static variables. In Java 8+, Metaspace lives in native memory (outside the heap), eliminating the old PermGen OutOfMemoryErrors.

---

Q2. Explain Java's garbage collection and the difference between the major GC algorithms.

A2.
The JVM's garbage collector automatically reclaims memory occupied by objects no longer reachable from any GC root (thread stacks, static fields, JNI references). The core algorithm is mark-and-sweep: mark all live objects starting from GC roots, then sweep unreachable ones.

**Serial GC**: Single-threaded, stop-the-world pauses. Fine for small apps, not for production servers.

**Parallel GC (default pre-Java 9)**: Multi-threaded young-gen collection, still stop-the-world. Maximises throughput at the cost of longer pauses.

**G1GC (default Java 9+)**: Divides heap into equal-sized regions instead of fixed generations. Collects regions with the most garbage first ("Garbage First"). Targets configurable pause-time goals. Good balance of throughput and latency.

**ZGC / Shenandoah (Java 15+ production)**: Low-latency collectors with sub-millisecond pauses, even on terabyte heaps. Most work is done concurrently with application threads. Ideal for latency-sensitive services.

---

Q3. What is the difference between `==` and `.equals()` in Java?

A3.
`==` compares references — it checks whether two variables point to the exact same object in memory. For primitives, it compares values directly.

`.equals()` is a method defined on `Object` that, by default, also does reference comparison. But most classes override it to compare content. `String.equals()` compares character sequences, `Integer.equals()` compares numeric values, etc.

The classic gotcha is string comparison: `"hello" == "hello"` may be `true` due to string interning (both references point to the same pooled object), but `new String("hello") == new String("hello")` is always `false`. Always use `.equals()` for string comparison.

`String.equalsIgnoreCase()` for case-insensitive comparison. For null-safe equality, use `Objects.equals(a, b)` — it handles null without throwing NullPointerException.

---

Q4. What is the Java Collections Framework? Explain the key interfaces and common implementations.

A4.
The Java Collections Framework is a unified architecture for representing and manipulating collections.

Core interfaces: `Collection` is the root; `List` (ordered, duplicates allowed), `Set` (no duplicates), `Queue`/`Deque` (ordered for processing), `Map` (key-value pairs, not a true Collection).

Key implementations:

`ArrayList`: Dynamic array. O(1) random access, O(n) insert/delete in the middle. Best general-purpose list.

`LinkedList`: Doubly-linked list. O(1) insert/delete at ends, O(n) random access. Also implements `Deque`.

`HashMap`: Hash table. O(1) average get/put. Not ordered, not thread-safe. Uses separate chaining; converts to balanced tree when a bucket exceeds 8 entries.

`LinkedHashMap`: Maintains insertion order. `TreeMap`: Sorted by key, O(log n) operations. `HashSet`/`LinkedHashSet`/`TreeSet` are analogous.

`ArrayDeque`: Preferred stack/queue implementation over `Stack`/`LinkedList`. `PriorityQueue`: Min-heap for priority-based processing.

For thread safety: use `ConcurrentHashMap` (lock-striped, not fully synchronised), `CopyOnWriteArrayList` (optimised for many reads, few writes), or wrap with `Collections.synchronizedXxx()`.

---

Q5. What are generics in Java, and what is type erasure?

A5.
Generics allow you to write type-safe code that works with any type, with the type resolved at compile time. `List<String>` ensures the compiler catches type mismatches without casting.

**Type erasure** is how Java implements generics: all type parameter information is removed at compile time. At runtime, `List<String>` and `List<Integer>` are both just `List`. The compiler inserts casts where needed and enforces type safety statically, but no generic type info exists in bytecode at runtime.

Practical implications: you can't do `new T()`, `new T[]`, `instanceof List<String>`, or get the runtime class of a generic parameter without extra machinery (like passing `Class<T> clazz` to the constructor). This is why you sometimes see awkward patterns in Java frameworks.

Bounded type parameters: `<T extends Comparable<T>>` restricts T to types that implement Comparable. Wildcards: `List<?>` for unknown type, `List<? extends Animal>` for a covariant read-only list, `List<? super Dog>` for a contravariant write-only list (Producer Extends, Consumer Super — PECS).

---

Q6. Explain Java's exception hierarchy. What is the difference between checked and unchecked exceptions?

A6.
Java exceptions inherit from `Throwable`. `Error` (JVM-level problems like `OutOfMemoryError`, not for normal handling) and `Exception` (application-level problems) are the two main branches.

**Checked exceptions** extend `Exception` but not `RuntimeException`. The compiler forces you to either catch them or declare them with `throws`. Examples: `IOException`, `SQLException`. Used for recoverable conditions where callers should actively handle failure (file not found, network timeout).

**Unchecked exceptions** extend `RuntimeException`. No compiler enforcement. Examples: `NullPointerException`, `IllegalArgumentException`, `ArrayIndexOutOfBoundsException`. Used for programming errors — bugs in the code that shouldn't be caught and hidden.

Modern Java style tends to favour unchecked exceptions to avoid cluttered `throws` declarations. Spring and Hibernate convert checked SQL/IO exceptions to unchecked `RuntimeException` subclasses for this reason.

Always catch the most specific exception type first. Never catch `Exception` or `Throwable` unless you're at a top-level boundary (e.g., a servlet filter logging all errors). Use `finally` or try-with-resources to guarantee resource cleanup.

---

Q7. What are Java's access modifiers?

A7.
Java has four access levels, from most restrictive to least:

`private`: Accessible only within the same class. Use for internal implementation details.

package-private (no modifier): Accessible within the same package. Good for package-internal collaboration.

`protected`: Accessible within the same package and in subclasses (even in different packages). Use for methods intended to be overridden or used by subclasses.

`public`: Accessible from anywhere. Use for the public API of a class.

The guiding principle is encapsulation — expose the minimum necessary. Start with `private` and relax only when required. This limits the surface area of your API, making it easier to change implementation later without breaking callers.

For classes and interfaces at the top level, only `public` and package-private are valid. Inner classes can use all four.

---

Q8. What is the difference between `abstract class` and `interface` in Java?

A8.
An `abstract class` can have constructor logic, instance state (fields), and a mix of abstract and concrete methods. A class can extend only one abstract class (single inheritance).

An `interface` historically was a pure contract — only abstract method signatures. Since Java 8, interfaces can have `default` methods (concrete implementations) and `static` methods. Since Java 9, `private` methods too. But interfaces still can't have instance state (only `public static final` constants). A class can implement multiple interfaces.

When to use which: Use an abstract class when you have shared implementation and state across closely related classes. Use an interface to define a capability contract for unrelated classes. The `Comparable`, `Runnable`, `Serializable` interfaces are the classic examples — a `Dog`, a `Thread`, and a `JSONConfig` are unrelated but all can implement `Runnable`.

A common design pattern: define the interface for the API (dependency inversion), provide an abstract class with partial implementation for convenience, and let concrete classes complete both.

---

Q9. How does Java's `synchronized` keyword work? What are its limitations?

A9.
`synchronized` ensures that only one thread at a time can execute a block or method on a given object's intrinsic lock (monitor). When a thread enters a `synchronized` block, it acquires the lock; other threads trying to enter block until the lock is released.

`synchronized` on an instance method uses `this` as the lock. On a `static` method, it uses the Class object. You can also synchronise on a specific object: `synchronized(myLock) { ... }`.

Limitations: intrinsic locks are not interruptible — a thread blocked waiting for a lock can't be interrupted. There's no way to try to acquire a lock without blocking. You can't have separate read and write locks on the same monitor. All waiters compete for the same lock, even for independent operations.

Better alternatives: `java.util.concurrent.locks.ReentrantLock` for interruptible, try-lock semantics. `ReadWriteLock` for multiple concurrent readers. `StampedLock` for optimistic reads. High-level concurrent collections (`ConcurrentHashMap`, etc.) that avoid explicit locking. `AtomicInteger`/`AtomicReference` for simple atomic operations without locking.

---

Q10. What are Java streams and the Stream API? How do they differ from collections?

A10.
Streams (introduced in Java 8) represent a pipeline of operations on a sequence of elements. Unlike collections, streams are not data stores — they don't hold elements, they process them. A stream is a view over a source (a collection, array, or I/O channel).

Key distinction: collections are about storing and accessing data; streams are about computing results from data. You can only traverse a stream once; a collection can be iterated multiple times.

Stream operations are lazy — intermediate operations (`filter`, `map`, `flatMap`, `sorted`, `distinct`) are not executed until a terminal operation (`collect`, `forEach`, `reduce`, `count`, `findFirst`) is called. This enables the JVM to optimise the pipeline.

Streams support parallel processing with `.parallelStream()` — the ForkJoinPool splits the work across available CPU cores. Useful for CPU-bound operations on large datasets, but overhead makes it counterproductive for small collections.

Common pattern: `list.stream().filter(x -> x > 0).map(Object::toString).collect(Collectors.toList())`. Collectors like `Collectors.groupingBy()`, `Collectors.toMap()`, `Collectors.joining()` handle aggregation.

---

### MEDIUM PRIORITY

---

Q11. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?

A11.
`String` is immutable — every operation that appears to modify a String (concatenation, replace, etc.) creates a new String object. This is safe for sharing between threads but inefficient for repeated modifications.

`StringBuilder` is mutable and not thread-safe. For single-threaded string building (in loops, formatters), always use `StringBuilder` — it's dramatically faster than repeated `+` concatenation, which creates O(n) intermediate strings.

`StringBuffer` is `StringBuilder`'s thread-safe counterpart — every method is `synchronized`. Use it when multiple threads share the same buffer. In practice, this is rare; if you need concurrent string building, design so threads have their own `StringBuilder` and merge at the end.

The compiler automatically rewrites simple `a + b + c` into a `StringBuilder` chain for you. But inside a loop, it creates a new `StringBuilder` per iteration — write the `StringBuilder` manually in loops with many iterations.

---

Q12. Explain Java's `final` keyword. What does it mean on a variable, method, and class?

A12.
On a **variable**: the reference cannot be reassigned after initialisation. For primitives, this makes the value constant. For objects, the reference is fixed but the object's internal state can still be mutated (unless the object itself is immutable).

On a **method**: the method cannot be overridden by a subclass. Use this when you want to guarantee a behaviour that shouldn't be changed by inheritance.

On a **class**: the class cannot be subclassed. `String`, `Integer`, and all wrapper types are `final`. Makes the class easier to reason about and enables compiler/JIT optimisations. Also important for security — preventing someone from subclassing and overriding security-critical methods.

A `static final` field is a compile-time or runtime constant. `public static final` fields are the Java convention for constants (typically named in `UPPER_SNAKE_CASE`).

---

Q13. What is the Java Memory Model (JMM), and what does `volatile` do?

A13.
The JMM defines the rules for how threads interact through memory. Without synchronisation, threads can see stale cached values — the JVM and CPU are free to reorder instructions and cache values in registers or CPU caches.

`volatile` guarantees two things: visibility (every write to a `volatile` field is immediately visible to all threads, no caching in registers) and ordering (writes/reads of volatile fields cannot be reordered with preceding/following regular operations).

`volatile` is lighter than `synchronized` — it doesn't create mutual exclusion, just guarantees visibility and prevents certain reorderings. Use it for a simple status flag (`volatile boolean running`) that one thread writes and others read, without needing compound atomic operations.

The classic `volatile` use case is the double-checked locking singleton pattern:
```java
private volatile static Instance instance;
if (instance == null) {
    synchronized(Instance.class) {
        if (instance == null) instance = new Instance();
    }
}
```
Without `volatile`, the partially-constructed object could be visible to other threads.

---

Q14. What is the `Optional` class in Java 8, and when should you use it?

A14.
`Optional<T>` is a container that may or may not hold a non-null value. Its purpose is to make the absence of a value explicit in the type system, forcing callers to handle the null case rather than ignoring it.

Key methods: `Optional.of(value)` (throws NPE if null), `Optional.ofNullable(value)` (wraps null safely), `Optional.empty()`. Then `isPresent()`, `get()`, `orElse(default)`, `orElseGet(supplier)`, `orElseThrow()`, `map()`, `flatMap()`, `filter()`.

When to use: as a return type from methods that might not find a value (repository lookups, search methods). Do not use `Optional` as a parameter type, instance field, or in serialisation — it's designed specifically as a return type.

Avoid `optional.isPresent()` / `optional.get()` pairs — that's just a wrapped null check. Use `optional.orElse()`, `optional.map()`, `optional.ifPresent()` to write more expressive, functional code.

---

Q15. How does Java's `HashMap` handle collisions internally? How does it differ in Java 8+?

A15.
`HashMap` internally uses an array of buckets. When you put a key-value pair, the key's `hashCode()` is computed and mapped to a bucket index. If two keys land on the same bucket (collision), they're stored in a chain at that bucket.

**Pre-Java 8**: Each bucket is a simple linked list. If many keys hash to the same bucket (due to a bad hash function or a hash collision attack), lookups degrade to O(n) — you have to walk the entire chain.

**Java 8+**: When a bucket's chain exceeds 8 entries and the total map capacity is at least 64, the linked list at that bucket is converted to a balanced binary tree (red-black tree). This caps worst-case bucket lookup at O(log n) instead of O(n). The tree reverts to a linked list if the count drops below 6.

This change was motivated by hash-flooding attacks — sending inputs with deliberate hash collisions to degrade a HashMap to O(n) per operation. The tree conversion bounds the worst case.

Important: `hashCode()` and `equals()` must be consistent — if `a.equals(b)`, then `a.hashCode() == b.hashCode()`. Violating this breaks HashMap behaviour entirely.

---

Q16. What is the difference between `Comparable` and `Comparator` in Java?

A16.
`Comparable<T>` defines the natural ordering of a class. It has one method: `int compareTo(T other)`. A class implements `Comparable` when it has a single obvious ordering — `String` sorts alphabetically, `Integer` sorts numerically. `Collections.sort()` and `Arrays.sort()` use this natural order by default.

`Comparator<T>` defines an external ordering strategy. It has one functional method: `int compare(T o1, T o2)`. Use it when you want to sort the same objects differently in different contexts, or when you can't modify the class (e.g., sorting a third-party type). `Collections.sort(list, comparator)` or `list.sort(comparator)`.

Java 8 made `Comparator` a `@FunctionalInterface`, enabling lambda usage: `list.sort((a, b) -> a.getName().compareTo(b.getName()))`. `Comparator.comparing()`, `.thenComparing()`, `.reversed()` provide fluent composition: `Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName)`.

---

Q17. What is Java's `try-with-resources` statement?

A17.
`try-with-resources` (Java 7+) automatically closes resources at the end of the block. Any resource that implements `AutoCloseable` (or its subinterface `Closeable`) can be declared in the try clause: `try (Connection c = getConnection(); Statement s = c.createStatement()) { ... }`.

Resources are closed in reverse order of declaration. Closing happens even if an exception is thrown in the try block. If both the try block and `close()` throw exceptions, the try exception is propagated and the close exception is suppressed (accessible via `Throwable.getSuppressed()`).

This replaces the old boilerplate:
```java
Connection c = null;
try { c = getConnection(); ... } finally { if (c != null) c.close(); }
```
Which was error-prone — close exceptions could mask the original exception.

Use try-with-resources for any I/O resource: files, streams, connections, sockets.

---

Q18. What are functional interfaces and lambdas in Java 8?

A18.
A functional interface has exactly one abstract method. The `@FunctionalInterface` annotation is optional but signals intent and causes a compile error if you accidentally add a second abstract method.

The key built-in functional interfaces in `java.util.function`:
- `Function<T,R>`: takes T, returns R (`apply`)
- `Predicate<T>`: takes T, returns boolean (`test`)
- `Consumer<T>`: takes T, returns void (`accept`)
- `Supplier<T>`: takes nothing, returns T (`get`)
- `BiFunction<T,U,R>`, `BiPredicate`, etc. for two-argument variants.
- `UnaryOperator<T>`, `BinaryOperator<T>` for T→T operations.

A lambda is syntactic sugar for an anonymous implementation of a functional interface: `Predicate<String> notEmpty = s -> !s.isEmpty()`. Method references (`String::isEmpty`, `System.out::println`, `MyClass::new`) are a more readable shorthand when the lambda just calls an existing method.

Lambdas capture variables from the enclosing scope (they must be effectively final). They enable a functional programming style in Java — passing behaviour as data.

---

Q19. Explain Java's `record` type (Java 16+).

A19.
A `record` is a concise syntax for declaring data-carrying classes. When you write `record Point(int x, int y) {}`, the compiler auto-generates: a canonical constructor, private final fields, public accessor methods (`x()`, `y()`), `equals()` based on all fields, `hashCode()`, and a `toString()`.

Records are implicitly `final` — they cannot be subclassed. All fields are final — records are inherently immutable value objects.

You can add custom constructors (including a compact constructor that runs before field assignment), extra methods, and static members. You can also implement interfaces.

Records are ideal for DTOs, value types, compound map keys, event objects, and any case where you want an immutable data holder without verbose boilerplate. They signal intent clearly: this class exists to hold data, not to encapsulate mutable state.

Before records, you'd either write pages of boilerplate or use Lombok `@Value`. Records are the idiomatic modern Java approach.

---

Q20. What are Java's concurrency utilities in `java.util.concurrent`?

A20.
`java.util.concurrent` provides higher-level concurrency primitives that are safer and more flexible than raw `synchronized`/`wait`/`notify`.

`ExecutorService` and thread pools: `Executors.newFixedThreadPool(n)`, `newCachedThreadPool()`, `newSingleThreadExecutor()`. Submit `Runnable`/`Callable` tasks and get back `Future<T>` for async results. `CompletableFuture` (Java 8+) enables composable async pipelines.

Locks: `ReentrantLock` (interruptible, try-lock, fair mode), `ReadWriteLock` (multiple concurrent readers, exclusive writers), `StampedLock` (optimistic reads).

Atomic variables: `AtomicInteger`, `AtomicLong`, `AtomicReference` — lock-free thread-safe operations using CPU CAS (compare-and-swap) instructions.

Concurrent collections: `ConcurrentHashMap` (lock striped), `CopyOnWriteArrayList` (snapshot-on-write), `BlockingQueue` implementations (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`) for producer-consumer patterns.

Synchronisation aids: `CountDownLatch` (wait for N events), `CyclicBarrier` (wait for N threads to reach a point), `Semaphore` (limit concurrent access), `Phaser` (flexible multi-phase barrier).

