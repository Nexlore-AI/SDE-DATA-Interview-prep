==============================
FILE: Programming Fundamentals
==============================

### HIGH PRIORITY

---

Q1. What is the difference between pass-by-value and pass-by-reference? How does your primary language handle it?

A1.
So when you pass by value, the function gets a copy of the data — any changes inside the function don't affect the original. Pass by reference means the function gets a reference to the actual memory location, so mutations inside the function reflect outside too.

In Python, it's a bit nuanced — they call it "pass by object reference." If you pass a mutable object like a list, changes to it inside the function are visible outside. But if you reassign the variable itself to a new object, the caller doesn't see that. Java is similar — everything is pass by value, but for objects, the value being passed is a reference (pointer), so you can mutate the object but not replace it from the caller's perspective.

The practical implication: if you're not careful with mutable objects being passed around, you can get subtle bugs where one function accidentally modifies data another function depends on. That's why immutability is so valued in many codebases.

---

Q2. What are the four pillars of object-oriented programming, and why does each one matter?

A2.
The four pillars are encapsulation, abstraction, inheritance, and polymorphism.

Encapsulation is about bundling data and the methods that operate on it together, and hiding internal state. Like, you don't want external code reaching into an object and modifying its fields directly — that leads to chaos. You expose a controlled interface instead.

Abstraction is closely related — it's about exposing only what's necessary and hiding complexity. A database connection class, for example, hides all the socket and protocol details behind simple methods like `connect()` and `query()`.

Inheritance lets you create a hierarchy where child classes reuse behavior from parent classes. It's useful but can be overused — deep inheritance trees become really hard to reason about.

Polymorphism means you can treat different types through a common interface. A `process_payment(payment)` function shouldn't care if it's a CreditCardPayment or UPIPayment — each type knows how to handle itself. This is what makes code extensible without constant modifications.

---

Q3. What is the difference between an abstract class and an interface? When do you use each?

A3.
An abstract class is a partially implemented class — it can have both concrete methods with real logic and abstract methods that subclasses must implement. An interface, traditionally, is a pure contract — just method signatures, no implementation.

In modern languages the line has blurred. Java interfaces can have default methods now, Python doesn't have formal interfaces (you use ABCs). But the conceptual distinction still matters.

I'd use an abstract class when I have shared behavior across related classes — like a `BaseHTTPHandler` that has common parsing logic but forces subclasses to implement `handle_request()`. I'd use an interface when I want to define a capability that unrelated classes can implement — like `Serializable` or `Comparable`. A `Dog` and a `JSONConfig` aren't related, but both could implement `Serializable`.

The rule of thumb: abstract classes define what something *is*, interfaces define what something *can do*.

---

Q4. What is polymorphism? How does compile-time polymorphism differ from runtime polymorphism?

A4.
Polymorphism means "many forms" — it lets you write code that works with objects of different types through a common interface, without knowing the exact type at the point of writing.

Compile-time polymorphism is method overloading and generics — the compiler resolves which version of a function to call based on argument types at compile time. Like having `add(int, int)` and `add(float, float)`.

Runtime polymorphism is method overriding — where a subclass provides its own implementation of a parent's method, and the correct one is chosen at runtime based on the actual object type. If you have a `Shape` reference pointing to a `Circle`, calling `draw()` invokes Circle's version, not Shape's.

Runtime polymorphism is the more powerful one in practice — it's what enables the Open/Closed Principle. You can add new types without modifying existing code that uses the base type.

---

Q5. What is the difference between composition and inheritance? Why do experienced engineers often prefer composition?

A5.
Inheritance is an "is-a" relationship — a Dog is an Animal. Composition is a "has-a" relationship — a Car has an Engine. With inheritance, the child class gets everything from the parent, including stuff it might not need. With composition, you build objects by plugging in components.

Experienced engineers prefer composition because inheritance creates tight coupling. If you change the parent class, every child is affected. Deep inheritance hierarchies become really brittle — you end up with the "fragile base class" problem. Also, most languages only support single inheritance, so you quickly hit limits.

Composition gives you flexibility. Want to change behavior? Swap out a component. Want to combine behaviors from different sources? Inject multiple collaborators. It's easier to test too — you can mock individual components.

A practical example: instead of `class ElectricCar extends Car`, you'd have `class Car` with an `Engine` field. Want a gas car? Inject `GasEngine`. Electric? Inject `ElectricEngine`. Way more flexible than an inheritance tree.

---

Q6. What is a closure, and how does it capture variables from its enclosing scope?

A6.
A closure is a function that remembers and has access to variables from the scope where it was defined, even after that scope has finished executing.

Say you have an outer function that defines a variable `count = 0` and returns an inner function that increments and returns `count`. Even after the outer function returns, the inner function still has access to `count`. That's a closure — it "closes over" the variables it references.

In Python, JavaScript, and most modern languages, closures are fundamental. They power things like callbacks, decorators, partial application, and factory functions.

One gotcha: closures capture the *variable*, not the *value*. So if you create closures in a loop and they all reference the same loop variable, they'll all see the final value of that variable, not the value at the time they were created. That's a classic bug in Python and JavaScript.

---

Q7. What is the difference between mutable and immutable objects? Why does immutability matter in concurrent code?

A7.
A mutable object can be changed after creation — you can modify its fields, add items, etc. An immutable object, once created, cannot be altered. Any "modification" produces a new object.

In Python, strings and tuples are immutable; lists and dicts are mutable. In Java, `String` is immutable, `StringBuilder` is mutable.

Immutability matters hugely in concurrent code because if an object can't change, you don't need locks to share it across threads. No thread can modify it, so there are no race conditions. That's why functional programming languages emphasize immutability — it makes concurrent code fundamentally safer.

The trade-off is performance — creating new objects on every "change" can be expensive. But in practice, modern runtimes and data structures (like persistent data structures) handle this efficiently. For most applications, the safety gains far outweigh the overhead.

---

Q8. What is the difference between static typing and dynamic typing? What are the practical trade-offs?

A8.
Static typing means types are checked at compile time — you declare types and the compiler catches type errors before the code runs. Java, Go, TypeScript. Dynamic typing means types are checked at runtime — variables can hold any type, and errors show up when the code actually executes. Python, Ruby, JavaScript.

The practical trade-offs: static typing catches bugs earlier, gives you better IDE support (autocomplete, refactoring), and serves as living documentation. But it can slow down prototyping and add verbosity.

Dynamic typing lets you iterate faster and write less boilerplate — great for scripting, quick prototypes, and data exploration. But as projects grow, the lack of enforced types makes refactoring scary and bugs harder to catch.

In practice, the industry has moved toward a middle ground. Python has type hints, JavaScript has TypeScript. You get the best of both worlds — dynamic when you want speed, type annotations when you want safety. For production backend services, I strongly prefer static or gradually-typed code.

---

Q9. What is exception handling? How do you decide between throwing an exception, returning an error, and using a result type?

A9.
Exception handling is a mechanism to deal with unexpected situations during program execution. When something goes wrong, you throw/raise an exception, and the runtime unwinds the call stack until it finds a handler (try/catch block).

The decision depends on whether the error is expected or truly exceptional. For expected failures — like a user not being found in a database — returning an error value or result type is cleaner. The caller explicitly handles it. In Go, this is the norm: `user, err := findUser(id)`.

Exceptions work better for truly unexpected situations — network failures, out-of-memory, corrupted data. Things the immediate caller usually can't handle and need to bubble up.

Result types (like Rust's `Result<T, E>` or a typed union) are the most ergonomic in my experience. They force callers to handle the error case — you literally can't access the value without checking for errors first. It eliminates the "forgot to catch" problem.

The anti-pattern to avoid: using exceptions for control flow. It's slow, hard to trace, and makes code unreadable.

---

Q10. What is a lambda expression, and where would you use one over a named function?

A10.
A lambda is an anonymous, inline function — you define it right where you use it, without giving it a name. In Python: `lambda x: x * 2`. In Java: `(x) -> x * 2`. In JavaScript: `(x) => x * 2`.

I'd use a lambda when the function is short, used exactly once, and naming it wouldn't add clarity. The classic use case is in collection operations — `sorted(users, key=lambda u: u.age)` is cleaner than defining a separate `get_age()` function.

I'd use a named function when the logic is complex (more than one expression), when it's reused, or when a descriptive name makes the code self-documenting. If I have to read a lambda twice to understand it, it should be a named function.

In Python specifically, lambdas are limited to a single expression — no statements, no multiple lines. So anything non-trivial becomes a named function anyway.

---

### MEDIUM PRIORITY

---

Q11. What is type coercion, and what class of bugs does it introduce?

A11.
Type coercion is when the language automatically converts one type to another to make an expression work. Like in JavaScript, `"5" + 3` gives you `"53"` (string concatenation), but `"5" - 3` gives you `2` (numeric subtraction). Same operator, wildly different results based on implicit conversion rules.

The bugs it introduces are subtle and maddening. Comparisons like `0 == ""` being `true` in JavaScript, or `"" == false` being `true`. These aren't logical, they're just the coercion rules. You end up with conditions that pass when they shouldn't, or data silently changing type as it flows through your code.

This is exactly why JavaScript best practice is to always use `===` over `==`, and why Python's stronger typing (it'll throw a TypeError rather than coerce) prevents a whole class of these bugs. In statically typed languages, the compiler catches these at build time.

---

Q12. What is the difference between a shallow copy and a deep copy? When does it matter?

A12.
A shallow copy creates a new object but copies references to the nested objects — so the top-level container is independent, but the items inside it point to the same memory. A deep copy recursively copies everything — you get a fully independent clone.

It matters when you have nested mutable structures. Say you shallow copy a list of lists. If you modify an inner list in the copy, the original sees the change too — because both point to the same inner list objects. That can be a nasty surprise.

In Python: `copy.copy()` is shallow, `copy.deepcopy()` is deep. In practice, I reach for deep copy when I need a true independent snapshot — like saving state before a transformation. But deep copy can be expensive and has edge cases with circular references, so I use it deliberately, not as a default.

---

Q13. What are generics, and why are they important for writing reusable, type-safe code?

A13.
Generics let you write code that works with any type while still maintaining type safety. Instead of writing separate `IntList`, `StringList`, `UserList` classes, you write `List<T>` and the type parameter `T` gets filled in at usage time.

Without generics, you'd either duplicate code for each type or lose type safety by using something like `Object` or `Any` — and then you're back to casting and runtime errors.

The big win is that the compiler catches type mismatches. If you have `List<User>`, you can't accidentally add a `String` to it. The error shows up at compile time, not as a ClassCastException in production at 3 AM.

In Python, generics via type hints (`List[User]`, `Dict[str, int]`) serve a similar purpose for static analysis tools like mypy, even though Python doesn't enforce them at runtime.

---

Q14. What is a generator, and how does it differ from a function that returns a list?

A14.
A generator produces values one at a time, lazily — it yields a value, pauses, and resumes where it left off when the next value is requested. A function that returns a list computes everything upfront and holds it all in memory.

The difference is crucial for large datasets. If you're reading a 10GB file, a generator that yields one line at a time uses constant memory. A function that reads everything into a list would blow up your memory.

In Python, any function with `yield` becomes a generator. You can also use generator expressions: `(x**2 for x in range(1_000_000))` vs `[x**2 for x in range(1_000_000)]`. The generator version uses almost no memory.

Generators are also great for infinite sequences — like generating unique IDs or reading from a stream that never ends. You can't materialize infinity into a list.

---

Q15. What is the difference between eager evaluation and lazy evaluation? When does lazy evaluation produce better performance?

A15.
Eager evaluation computes the result immediately when an expression is encountered. Lazy evaluation defers computation until the result is actually needed.

Lazy evaluation wins when you don't need all the results. If you have a chain of transformations on a million records but only need the first 10 that match a filter, lazy evaluation stops after finding 10. Eager evaluation would process all million first.

Spark uses this — transformations like `map`, `filter` are lazy. Nothing executes until you call an action like `collect()` or `count()`. This lets Spark's optimizer look at the whole chain and figure out the most efficient execution plan.

The downside: lazy evaluation can make debugging harder because the error shows up when the value is consumed, not when the expression is defined. It can also cause memory issues if deferred computations accumulate (thunk leaks in Haskell, for example).

---

Q16. What is method overloading vs. method overriding? Can you have both in the same class?

A16.
Overloading is having multiple methods with the same name but different parameter types or counts — resolved at compile time. Overriding is a subclass providing its own implementation of a method defined in the parent — resolved at runtime.

Yes, you can have both in the same class (or class hierarchy). A parent class could overload `process(int)` and `process(String)`, and a child class could override `process(int)` with its own version. They're independent mechanisms.

Python doesn't support traditional overloading — defining a method with the same name just replaces the previous one. You'd use default arguments or `*args`/`**kwargs` instead. Java and C++ support both natively.

---

Q17. What is dependency injection, and how does it improve testability and decoupling?

A17.
Instead of an object creating its own dependencies internally, you pass them in from outside — through the constructor, method parameters, or a framework. That's dependency injection.

Say a `UserService` needs a `Database`. Without DI, it does `self.db = PostgresDB()` inside the constructor — now it's hardwired to Postgres. With DI, you pass the database in: `UserService(db)`. Now you can pass a `PostgresDB` in production and a `MockDB` in tests.

It improves testability because you can inject mocks and stubs trivially. It improves decoupling because the class depends on an abstraction (interface), not a concrete implementation. Swapping out the database, switching from S3 to GCS, or replacing a payment provider becomes a configuration change, not a code rewrite.

The trade-off: overusing DI frameworks (like Spring's heavy autowiring) can make code hard to follow — you lose the ability to just "go to definition" and see where things come from.

---

Q18. What is serialization and deserialization? Where is it used in distributed systems?

A18.
Serialization converts an in-memory object into a format that can be stored or transmitted — like JSON, Protobuf, Avro, or just plain bytes. Deserialization reverses it — taking that byte stream and reconstructing the object.

In distributed systems, it's everywhere. When two microservices talk over HTTP, the request body is serialized on the sender side and deserialized on the receiver. When you write to Kafka, the message is serialized. When you cache an object in Redis, it's serialized.

The format choice matters a lot. JSON is human-readable and universal but verbose and slow to parse. Protobuf and Avro are compact and fast but require a schema definition. For internal service-to-service communication where performance matters, binary formats like Protobuf win. For external APIs where developer experience matters, JSON is the standard.

One gotcha: serialization can be a security risk. Deserializing untrusted data (like Java's `ObjectInputStream` or Python's `pickle`) can execute arbitrary code. Always validate and prefer safe formats for external inputs.

---

### LOW PRIORITY

---

Q19. What is a coroutine, and how does it differ from a thread?

A19.
A coroutine is a function that can pause its execution and yield control back to a scheduler, then resume later from where it left off. Unlike threads, coroutines are cooperative — they decide when to yield, rather than being preemptively interrupted by the OS.

Threads are managed by the OS kernel, and switching between them involves a full context switch — saving registers, switching stacks, etc. That's expensive. Coroutines run in user space, and switching between them is just a function call — orders of magnitude cheaper.

The practical impact: you can have tens of thousands of coroutines running concurrently (Go goroutines, Python asyncio tasks) with minimal overhead. With OS threads, you'd run into memory and scheduling limits at a few thousand.

The catch: coroutines only provide concurrency, not parallelism (unless the runtime maps them to multiple OS threads like Go does). And one badly behaving coroutine that doesn't yield can block all others.

---

Q20. What is tail recursion, and which languages optimize for it?

A20.
Tail recursion is when the recursive call is the very last operation in the function — there's nothing left to do after the recursive call returns. In that case, the compiler can optimize it into a loop, reusing the same stack frame instead of pushing new ones. This prevents stack overflow for deep recursion.

Functional languages like Haskell, Scheme, and Erlang optimize for tail calls. Scala does too (with the `@tailrec` annotation). But Python, Java, and JavaScript (in most engines) do *not* optimize tail calls, so deep recursion will still blow the stack.

If you're in Python and need deep recursion, you'd convert it to an iterative loop manually. In Scala, writing tail-recursive functions is idiomatic and the compiler guarantees the optimization.

---

Q21. What is operator overloading, and what are the risks of misusing it?

A21.
Operator overloading lets you define what operators like `+`, `*`, `==`, `[]` mean for your custom types. In Python, you implement `__add__`, `__eq__`, etc. In C++, you use `operator+`.

Used well, it makes code intuitive. A `Vector` class where `v1 + v2` adds component-wise is natural. A `Money` class where `usd(10) + usd(20)` works is great.

Misused, it's a readability nightmare. If someone overloads `+` to mean "send a message" or `<<` to mean "log output" (looking at you, C++ iostream), readers have no idea what the code does without reading the implementation. The operator's meaning should be obvious and consistent with expectations.

---

Q22. What is the difference between a struct and a class in languages that support both?

A22.
In C++, the only technical difference is the default access level — struct members are public by default, class members are private. But conventionally, structs are used for simple data containers (plain old data), and classes for objects with behavior and encapsulation.

In C#, it's more significant — structs are value types (stored on the stack, copied on assignment) and classes are reference types (stored on the heap, passed by reference). This affects performance and semantics.

In Go, there are only structs — no classes at all. Methods are attached to structs externally.

In Rust, structs are the primary way to define custom types. There are no classes — you get behavior through `impl` blocks and traits.

---

Q23. What are first-class functions, and how do they enable higher-order programming patterns?

A23.
First-class functions mean functions are treated like any other value — you can assign them to variables, pass them as arguments, return them from other functions, and store them in data structures.

This enables higher-order functions — functions that take other functions as input or return them. Classic examples: `map`, `filter`, `reduce`. Instead of writing a loop to transform a list, you pass a transformation function to `map`.

It also enables patterns like callbacks, middleware chains, strategy pattern (without the class boilerplate), and decorators. In Python and JavaScript, first-class functions are fundamental to how the language is used daily. In Java, lambdas (added in Java 8) brought this capability to a classically OOP language.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q24. What is memory management? Explain stack vs heap allocation and when each is used.

A24.
The stack is a region of memory for function calls — it stores local variables, function parameters, and return addresses. It's LIFO (last in, first out), extremely fast, and automatically cleaned up when a function returns. Each thread gets its own stack. The downside: fixed size (usually 1-8 MB), and data must have a known size at compile time.

The heap is for dynamic memory — objects whose size isn't known at compile time, or that need to outlive the function that created them. You allocate on the heap explicitly (`new` in Java/C++, `malloc` in C). It's slower (allocation requires finding free space, possible fragmentation), and you're responsible for cleanup — either manually (C/C++) or via garbage collection (Java, Python, Go).

**When each is used**: Primitives and local variables → stack. Objects, dynamically-sized data (lists, strings), anything shared across functions → heap. In Java, all objects live on the heap, but the JVM may optimize small short-lived objects to the stack (escape analysis). In Rust, you choose explicitly — `Box<T>` allocates on the heap.

**Memory leaks happen** when heap-allocated memory is no longer referenced but not freed. In GC'd languages, this manifests as keeping unnecessary references (e.g., a growing list you forgot to clear).

---

Q25. How does garbage collection work? Compare mark-and-sweep, reference counting, and generational GC.

A25.
Garbage collection automatically reclaims memory that's no longer reachable by the program.

**Reference counting**: Each object tracks how many references point to it. When the count drops to zero, the object is freed immediately. Simple and predictable. **Problem**: Circular references — A references B, B references A, count never reaches zero. Python uses reference counting as its primary GC but adds a cycle detector to handle cycles. Swift uses ARC (Automatic Reference Counting) — the developer manages cycles with `weak` and `unowned` references.

**Mark-and-sweep**: Start from "root" references (stack variables, globals). Traverse all reachable objects (mark phase). Everything unmarked is garbage — free it (sweep phase). Handles cycles naturally. **Downside**: Pauses the application during collection ("stop the world").

**Generational GC** (Java, .NET, Go): Based on the observation that most objects die young. Divide the heap into generations — young (Eden/Survivor in Java), old (Tenured). Collect the young generation frequently (cheap — most objects are dead). Promote survivors to old generation. Collect old generation rarely (expensive but infrequent). This dramatically reduces pause times.

**Modern GCs** (G1, ZGC, Shenandoah) aim for sub-millisecond pauses even with huge heaps — critical for low-latency systems.

---

Q26. What is the difference between compiled and interpreted languages? Where do JIT-compiled languages fit?

A26.
**Compiled languages** (C, C++, Rust, Go): Source code → compiler → machine code (binary). Runs directly on the CPU. Fast execution, but you must compile for each target platform. Errors are caught at compile time.

**Interpreted languages** (Python, Ruby, JavaScript historically): Source code → interpreter reads and executes line by line (or statement by statement). No separate compile step. Slower execution, but immediate feedback and platform-independent. Errors surface at runtime.

**JIT-compiled languages** (Java, C#, modern JavaScript): Source code → bytecode (intermediate representation) → at runtime, a JIT compiler identifies "hot" code paths and compiles them to native machine code. You get portability of bytecode + near-native performance for hot code. Java compiles to bytecode for the JVM; the JVM's JIT (C2 compiler) optimizes frequently executed methods.

**Why it matters in practice**: Python is ~50-100x slower than C for compute-heavy tasks — that's why NumPy and TensorFlow are written in C/C++ with Python wrappers. Java starts slower (JIT warmup) but reaches near-C performance for long-running services. Go compiles to native binaries but has the simplicity of an interpreted language.

**Modern blur**: CPython is interpreted, but PyPy JIT-compiles Python for 5-10x speedups. TypeScript compiles to JavaScript, which is JIT-compiled. The line between compiled and interpreted is increasingly blurry.

---

Q27. What is async/await? How does asynchronous programming differ from multithreading?

A27.
Async/await is a language-level abstraction for writing asynchronous code that reads like synchronous code. Instead of callbacks or complex promise chains, you write `await fetch(url)` and the function pauses at that point, freeing the thread to do other work, then resumes when the result is available.

**Key difference from multithreading**: Multithreading uses multiple OS threads running in parallel — true parallelism on multiple CPU cores. Async uses a single thread (or a small thread pool) with cooperative multitasking — when one task waits for I/O (network, disk), another task runs. No parallelism, but massive concurrency.

**When to use async**: I/O-bound workloads — web servers handling thousands of concurrent requests (most time is spent waiting for database/API responses, not computing). A single-threaded async server can handle thousands of concurrent connections. Node.js and Python's asyncio are built entirely on this model.

**When to use multithreading**: CPU-bound workloads — image processing, scientific computation, compression. You need actual parallel execution on multiple cores.

**Python caveat**: The GIL (Global Interpreter Lock) prevents true multithreaded parallelism for CPU-bound Python code. Use `multiprocessing` for CPU parallelism, `asyncio` for I/O concurrency. In Java/Go/Rust, threads provide true parallelism.

---

Q28. What are access modifiers (public, private, protected)? How do they enforce encapsulation?

A28.
Access modifiers control the visibility of class members (fields and methods), enforcing encapsulation — hiding internal implementation details and exposing only what's necessary.

**Public**: Accessible from anywhere. The external API of your class. Keep this surface small — anything public becomes a commitment you can't easily change.

**Private**: Accessible only within the same class. Implementation details that can change freely without breaking external code. In Python, there's no true private — convention uses `_single_underscore` (protected by convention) and `__double_underscore` (name-mangled, harder to access but not truly private).

**Protected**: Accessible within the class and its subclasses. In Java, also accessible within the same package. Used when subclasses need access to internals but external code shouldn't.

**Why it matters**: Without access modifiers, every field and method is part of the public API. Changes to any internal detail can break dependent code. With them, you create a clear contract: "Use these public methods, ignore the rest." This is the foundation of maintainable code — internal refactoring doesn't break external users.

**Languages differ**: Java has public/protected/default(package)/private. Python relies on convention. JavaScript has `#private` fields (ES2022). Go uses capitalization — exported (public) vs unexported (private).

---

### IMPORTANT

---

Q29. What is the event loop? How does it enable non-blocking I/O in single-threaded environments?

A29.
The event loop is the core mechanism behind Node.js, browser JavaScript, and Python's asyncio. It's a loop that continuously checks for and dispatches events or tasks.

**How it works**: The event loop runs on a single thread. When code initiates an I/O operation (HTTP request, file read, database query), instead of blocking the thread, it registers a callback and moves on. The I/O operation happens in the background (OS kernel, thread pool for file I/O). When the operation completes, the callback is placed in a queue. The event loop picks up the callback and executes it.

**Call stack → Web APIs → Callback Queue → Event Loop**: In a browser, `setTimeout(fn, 1000)` moves `fn` to the browser's timer API. After 1 second, `fn` goes to the callback queue. The event loop pushes it to the call stack only when the stack is empty.

**Microtask vs macrotask queue**: Promises (microtasks) have higher priority than setTimeout/setInterval (macrotasks). All microtasks are processed before the next macrotask.

**Why it's powerful**: A single Node.js thread can handle 10,000+ concurrent connections because it never blocks waiting for I/O. Each connection uses minimal memory (no thread per connection). This is why Node.js dominates API servers and real-time applications.

**Pitfall**: CPU-intensive work blocks the event loop — everything stops. Solution: worker threads, or break computation into chunks with `setImmediate()`.

---

Q30. What is a race condition? How do you prevent it at the application level?

A30.
A race condition occurs when the behavior of a program depends on the relative timing of events — two or more operations access shared state, and the outcome changes based on execution order.

**Classic example**: Two threads read a counter (value = 5), both increment it, both write 6. Expected: 7. Actual: 6. The read-modify-write isn't atomic.

**Application-level examples**: Two users buy the last item in stock simultaneously — both see quantity = 1, both decrement, quantity goes to -1. Two API requests update the same user profile — the second overwrites the first's changes.

**Prevention strategies**:
- **Mutexes/Locks**: Ensure only one thread accesses the critical section. `threading.Lock()` in Python, `synchronized` in Java.
- **Atomic operations**: Use language-level atomic types. `AtomicInteger` in Java, `atomics` in Go.
- **Optimistic locking (database)**: Add a version column. Read the version, attempt update with `WHERE version = X`. If another transaction changed it, your update affects 0 rows — retry.
- **Database transactions with proper isolation**: Use SERIALIZABLE isolation for critical operations.
- **Message queues**: Serialize operations through a queue — process one at a time.

**In async code**: Even single-threaded async code can have race conditions — two coroutines interleave their execution at `await` points.

---

Q31. How does Python's GIL affect concurrency? What are the workarounds?

A31.
The GIL (Global Interpreter Lock) is a mutex in CPython that allows only one thread to execute Python bytecode at a time. Even with multiple threads and multiple CPU cores, only one thread runs Python code at any given moment.

**Why it exists**: CPython's memory management (reference counting) isn't thread-safe. The GIL simplifies the interpreter implementation and makes C extensions easier to write. It was a practical decision when Python was designed in the early 1990s.

**What it means**: Multithreading in Python doesn't provide parallelism for CPU-bound tasks. Four threads doing computation on a 4-core machine still use only one core. But for I/O-bound tasks (network, file, database), threads work fine — while one thread waits for I/O, the GIL is released and another thread runs.

**Workarounds**:
- **`multiprocessing`**: Spawn separate processes, each with its own GIL. True parallelism. Communication via pipes, queues, or shared memory. Overhead: process creation and IPC.
- **`asyncio`**: For I/O-bound concurrency. Single thread, cooperative multitasking. No GIL issue because there's no parallelism attempted.
- **C extensions**: NumPy, pandas release the GIL during heavy computation — their C code runs in parallel.
- **Sub-interpreters (Python 3.12+)**: Each sub-interpreter has its own GIL. Experimental path toward true multithreaded Python.

---

Q32. What is recursion? When should you prefer iteration over recursion?

A32.
Recursion is when a function calls itself to solve smaller sub-problems of the same structure. Every recursive solution has a base case (stop condition) and a recursive case (break the problem down).

**When recursion is natural**: Tree traversals, graph DFS, divide-and-conquer (merge sort, quick sort), problems with recursive structure (Fibonacci, factorial, permutations, backtracking).

**When iteration is better**: Simple loops (summing an array), when you need performance and the language doesn't optimize tail calls, when stack depth is a concern (Python's default recursion limit is 1000).

**Trade-offs**: Recursion is often cleaner and more intuitive for hierarchical/nested structures. Iteration avoids stack overflow risk and is typically faster (no function call overhead). Any recursive solution can be converted to iterative using an explicit stack — DFS with recursion vs DFS with a stack data structure.

**Tail recursion**: When the recursive call is the last operation in the function. Some languages (Scheme, Kotlin, Scala) optimize this to iteration (no extra stack frames). Python and Java do NOT optimize tail recursion — you'll still hit stack overflow for deep recursion.

---

Q33. What is the difference between a strongly typed and a weakly typed language?

A33.
**Strongly typed** (Python, Java, Rust): The language enforces type rules strictly. You can't add a string and an integer — `"5" + 3` raises a TypeError in Python. Types must be explicitly converted. Catches type errors early.

**Weakly typed** (JavaScript, PHP, C to some extent): The language automatically converts (coerces) types to make operations work. `"5" + 3` → `"53"` in JavaScript (string concatenation). `"5" - 3` → `2` in JavaScript (numeric subtraction). This implicit coercion is a major source of bugs.

**Common confusion**: Strong/weak typing is NOT the same as static/dynamic typing. Python is strongly typed (no implicit coercion) and dynamically typed (types checked at runtime). Java is strongly typed and statically typed (types checked at compile time). JavaScript is weakly typed and dynamically typed.

**Practical impact**: Weak typing leads to subtle bugs — `[] == false` is true in JS, `"" == 0` is true in JS. Strong typing forces explicit intention, reducing ambiguity. TypeScript adds static types to JavaScript partly to compensate for weak typing issues.

---

### GOOD-TO-HAVE

---

Q34. What is reflection/introspection in programming? When is it useful and what are the risks?

A34.
Reflection is the ability of a program to examine and modify its own structure and behavior at runtime — inspect classes, call methods by name, access fields dynamically.

**Use cases**: Serialization frameworks (Jackson inspects fields to generate JSON), dependency injection containers (Spring scans for annotations and creates objects), ORMs (map database columns to object fields), testing frameworks (discover and run test methods).

**Python introspection**: `type()`, `dir()`, `getattr()`, `hasattr()`, `inspect` module. Python makes reflection trivial — it's used constantly.

**Java reflection**: `Class.forName()`, `getMethod()`, `invoke()`. More verbose but powerful. Spring Framework is built on reflection.

**Risks**: Slower than direct calls (bypasses compiler optimizations). Breaks encapsulation (can access private fields). Harder to refactor — IDE can't trace reflective calls. Security concerns — code can invoke arbitrary methods. Use reflection in frameworks, not in application logic.

---

Q35. What are annotations/decorators? How do they modify behavior without changing the underlying function?

A35.
Decorators (Python) and annotations (Java) attach metadata or modify behavior of functions/classes without changing their source code.

**Python decorators**: A function that takes a function, wraps it, and returns the wrapper. `@login_required` above a view function adds authentication checking before the view runs. The original function's code is untouched — the decorator adds behavior around it.

**Java annotations**: `@Override`, `@Deprecated`, `@Autowired`. They're metadata — they don't modify code directly but are read by frameworks (Spring uses `@Autowired` to inject dependencies, `@Transactional` to wrap methods in database transactions).

**How decorators work internally**: `@decorator` above `def func()` is syntactic sugar for `func = decorator(func)`. The decorator returns a new function that typically calls the original function inside, adding logic before/after.

**Common uses**: Logging, timing, caching (`@lru_cache`), authentication, rate limiting, retry logic, input validation. They enable separation of concerns — the core function does its job, decorators handle cross-cutting concerns.

---

Q36. What is the difference between value types and reference types in memory?

A36.
**Value types**: Store the actual data directly. When you assign or pass a value type, a copy is made. Modifying the copy doesn't affect the original. In C#: `int`, `float`, `struct`, `enum`. In Java: primitives (`int`, `boolean`). Stored on the stack (usually).

**Reference types**: Store a reference (pointer) to the actual data. When you assign or pass a reference type, you copy the reference — both variables point to the same underlying data. In Java/C#: objects, arrays, strings (though strings are immutable). Stored on the heap, reference on the stack.

**Why it matters**: With value types, `a = b` means `a` is independent of `b`. With reference types, `a = b` means changes through `a` are visible through `b`. This is a constant source of bugs — modifying a list you thought was a copy actually modifies the original. Understanding this distinction is essential for debugging mutation bugs and reasoning about function side effects.