==============================
FILE: Python — Language-Specific Interview Questions
==============================

### HIGH PRIORITY

---

Q1. What is Python's GIL (Global Interpreter Lock), and how does it affect multithreading?

A1.
The GIL is a mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecode simultaneously in a single process. It exists because CPython's memory management (reference counting) is not thread-safe, so the GIL serialises execution to avoid corruption.

In practice this means CPU-bound multithreaded programs don't get parallelism — only one thread runs at a time. I/O-bound programs are fine because threads release the GIL while waiting for I/O, so you still get concurrency (just not true parallelism).

The workarounds: use `multiprocessing` for CPU-bound work — each process has its own GIL. Use async I/O (`asyncio`) for I/O-bound concurrency. Or drop into C extensions (NumPy, TensorFlow) that release the GIL themselves. Python 3.12+ has started work on making the GIL optional per-interpreter (PEP 703).

---

Q2. Explain Python's memory management and garbage collection.

A2.
Python primarily uses reference counting — every object tracks how many references point to it. When the count drops to zero, the memory is freed immediately. This is why you rarely think about memory management in Python — most cleanup happens automatically.

The problem is reference cycles. If object A references B and B references A, their counts never reach zero even if nothing else points to them. CPython has a cyclic garbage collector (`gc` module) that periodically looks for such cycles and frees them. You can trigger it manually with `gc.collect()`, or disable it if you're certain your code creates no cycles (for performance-sensitive code).

Memory is allocated from Python's private heap. CPython uses a layered allocator — objects under 512 bytes go through a pool allocator (`pymalloc`) for efficiency, avoiding the overhead of calling `malloc` for small objects.

---

Q3. What are Python decorators, and how do they work under the hood?

A3.
A decorator is syntactic sugar for wrapping a function with another function. When you write `@my_decorator` above a function definition, Python executes `func = my_decorator(func)` — your original function is replaced by whatever `my_decorator` returns.

Under the hood, decorators rely on the fact that functions are first-class objects in Python. You can pass them around, return them from other functions, and assign them to variables.

A typical decorator uses a `wrapper` inner function to add behaviour before/after the original call, then returns that `wrapper`. The `functools.wraps` decorator is important here — it copies the original function's `__name__`, `__doc__`, etc. to the wrapper, so debugging tools see the right name.

Decorators can also be classes (implementing `__call__`) or parameterised (a decorator factory that takes arguments and returns a decorator). They're used extensively in frameworks — Flask routes (`@app.route`), Django views, Python `@property`, `@staticmethod`, `@classmethod` are all decorators.

---

Q4. What is the difference between `@staticmethod`, `@classmethod`, and instance methods?

A4.
Instance methods take `self` as the first parameter and have access to the specific instance's state. They can also access the class via `self.__class__`. This is the default method type.

Class methods use `@classmethod` and take `cls` as the first parameter — they receive the class itself, not an instance. You can call them on the class or an instance. Common use case: alternative constructors, like `dict.fromkeys()` or a `from_json(cls, json_str)` factory method.

Static methods use `@staticmethod` and receive no implicit first argument. They're logically related to the class but don't need access to instance or class state — essentially regular functions namespaced inside the class. Use them for utility functions that conceptually belong to the class but are stateless.

---

Q5. Explain Python generators and the `yield` keyword. How do they differ from returning a list?

A5.
A generator is a function that uses `yield` to produce values lazily — one at a time, on demand — instead of computing and storing everything upfront. When you call a generator function, you get back a generator object; it doesn't execute any code yet. Calling `next()` on it runs until the next `yield`, suspends execution there, and returns the yielded value.

The key difference from returning a list is memory. If you need to process a billion rows, building a list requires holding all billion rows in memory. A generator produces one row at a time and discards it — constant O(1) memory regardless of input size.

Generator expressions (like list comprehensions but with parentheses) are a shorthand for simple generators. `(x**2 for x in range(1000000))` is lazy; `[x**2 for x in range(1000000)]` is eager and consumes memory immediately.

Generators are the foundation of Python's iteration protocol. They also enable coroutines — the `asyncio` event loop is built on generators and `yield from`.

---

Q6. What are Python's `*args` and `**kwargs`, and when would you use them?

A6.
`*args` collects positional arguments into a tuple. `**kwargs` collects keyword arguments into a dict. Both are used to write functions that accept a variable number of arguments.

`*args` is useful when you want a function to accept any number of positional values — like `print()` does. `**kwargs` is useful for passing named options without hardcoding every parameter — like `requests.get(url, **options)`.

They're also used for forwarding arguments. In wrapper functions or decorators you often write `def wrapper(*args, **kwargs): return original(*args, **kwargs)` to forward all arguments unchanged to the wrapped function.

In Python 3, you can also use `*` in function signatures to force keyword-only arguments: `def func(a, b, *, timeout=30)` means `timeout` must be passed by name, not positionally.

---

Q7. What are list comprehensions, and how do they compare to `map()`/`filter()`?

A7.
List comprehensions provide a concise, readable syntax to build lists: `[x**2 for x in range(10) if x % 2 == 0]`. They're generally preferred over `map()` and `filter()` in modern Python because they're more readable and Pythonic.

`map(func, iterable)` applies a function to every element lazily. `filter(pred, iterable)` keeps only elements where the predicate is true. Both return iterators in Python 3.

The main trade-off: `map`/`filter` with `lambda` are sometimes more concise for trivial transforms, but comprehensions win for readability once logic gets more complex. `map` with a named function (not lambda) can be cleaner and slightly faster in tight loops.

For large datasets, use generator expressions `(...)` instead of list comprehensions `[...]` to avoid loading everything into memory.

---

Q8. How does Python's `with` statement and context manager protocol work?

A8.
The `with` statement guarantees that cleanup code runs even if an exception occurs. When you write `with open('file') as f:`, Python calls `f.__enter__()` at the start, runs the block, then calls `f.__exit__()` at the end — whether the block succeeded or raised an exception.

`__exit__` receives exception info (`exc_type`, `exc_val`, `exc_tb`). If it returns a truthy value, the exception is suppressed; otherwise it propagates.

You can create your own context managers in two ways: implement `__enter__` and `__exit__` in a class, or use `contextlib.contextmanager` as a decorator on a generator function. The generator yields exactly once — code before `yield` is the setup, code after is the teardown.

Context managers are used for resource management (files, DB connections, locks), timing code blocks, temporary state changes (e.g., overriding settings in tests), and transaction management.

---

Q9. What is the difference between `is` and `==` in Python?

A9.
`==` checks value equality — whether two objects have the same content. `is` checks identity — whether two variables point to the exact same object in memory (same `id()`).

The confusion comes from Python's interning. Small integers (-5 to 256) and short strings are cached, so `a = 5; b = 5; a is b` returns `True`. But this is an implementation detail you should never rely on.

The rule: use `==` to compare values. Use `is` only for identity checks — most commonly `x is None` and `x is not None`. Never use `is` to compare strings or numbers in production code.

---

Q10. What are Python's mutable vs. immutable types, and why does it matter?

A10.
Immutable types cannot be changed after creation: `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`. Mutable types can be modified in-place: `list`, `dict`, `set`, `bytearray`, custom objects.

It matters in several practical ways. First, default mutable arguments — `def f(x=[])` is a famous gotcha. The list is created once when the function is defined and shared across all calls. Use `def f(x=None)` and `if x is None: x = []` instead.

Second, dictionary keys must be hashable (immutable). You can use a `tuple` as a dict key but not a `list`.

Third, passing mutable objects to functions: mutations inside the function are visible to the caller. If you don't want that, pass a copy.

Fourth, thread safety: immutable objects can be shared between threads without locks since they can't be mutated.

---

### MEDIUM PRIORITY

---

Q11. What is the difference between `deepcopy` and `shallow copy`?

A11.
A shallow copy creates a new container but does not copy the nested objects inside — the outer container is new, but references to inner objects are shared. `list.copy()`, `list[:]`, and `copy.copy()` all do shallow copies.

A deep copy recursively copies everything — a completely independent clone with no shared references to the original structure. Use `copy.deepcopy()` for this.

Why does it matter? If you shallow-copy a list of lists and mutate an inner list, the mutation shows up in both the copy and the original because they reference the same inner list. With a deep copy, changes are fully isolated.

Use shallow copy when the inner objects are immutable (strings, ints) or when sharing inner objects is intentional. Use deep copy when you need a fully independent duplicate of a nested structure.

---

Q12. Explain Python's method resolution order (MRO) and how multiple inheritance works.

A12.
Python uses the C3 linearisation algorithm to determine the MRO — the order in which classes are searched when looking up a method or attribute. You can inspect it with `ClassName.__mro__` or `ClassName.mro()`.

The algorithm ensures three things: a class appears before its parents, the order of parents as declared is respected, and the MRO is monotonic (no class appears before a class it inherits from). This prevents the "diamond problem" from causing ambiguous lookups.

With `super()`, Python looks up the MRO and calls the next class in line, not necessarily the direct parent. This is critical for cooperative multiple inheritance to work — each class in the chain must call `super().__init__()` for all constructors to execute.

The practical advice: keep inheritance hierarchies simple. Multiple inheritance for mixins (adding specific capabilities) is reasonable; deep diamond hierarchies are usually a sign to use composition instead.

---

Q13. What are Python's `__dunder__` (magic/special) methods, and why are they useful?

A13.
Dunder methods (double-underscore methods) are hooks that let your classes integrate with Python's built-in operations and syntax. They make custom objects feel like native Python types.

Key examples: `__init__` for construction, `__repr__` and `__str__` for string representations, `__len__` for `len()`, `__getitem__`/`__setitem__` for subscript access `obj[key]`, `__iter__`/`__next__` for iteration, `__enter__`/`__exit__` for context managers, `__eq__`/`__lt__` for comparisons, `__add__`/`__mul__` for operator overloading, `__call__` to make instances callable.

For data classes, always implement at minimum `__repr__` (for debuggability) and `__eq__` (so equality checks work). If you're in a set or as a dict key, you also need `__hash__`.

`@dataclass` from Python 3.7+ auto-generates `__init__`, `__repr__`, and `__eq__` based on class annotations, removing the boilerplate.

---

Q14. How does `asyncio` work in Python? What is an event loop?

A14.
`asyncio` provides cooperative concurrency using coroutines. The event loop is a single-threaded scheduler that manages coroutines — it runs one coroutine until it hits an `await` expression, then switches to another ready coroutine while the first waits for I/O.

Coroutines are defined with `async def` and suspended with `await`. When you `await` something (like a network request), control returns to the event loop, which can run other coroutines. When the awaited operation completes, the coroutine is resumed.

Key primitives: `asyncio.gather()` to run multiple coroutines concurrently, `asyncio.create_task()` to schedule a coroutine without waiting, `asyncio.Queue` for producer-consumer patterns.

The critical constraint: you must not call blocking I/O inside a coroutine — it blocks the entire event loop. Use `asyncio.to_thread()` (Python 3.9+) to run blocking code in a thread pool. Libraries like `aiohttp`, `asyncpg`, and `motor` provide async-native I/O.

---

Q15. What is a Python metaclass?

A15.
A metaclass is the class of a class. Just as instances are created from classes, classes are created from metaclasses. The default metaclass is `type`. When Python sees a `class` statement, it calls the metaclass to actually construct the class object.

You define a custom metaclass by inheriting from `type` and overriding `__new__` or `__init__` (called during class creation) or `__call__` (called when the class is instantiated). This lets you intercept and modify class creation — for example, automatically adding methods, validating class attributes, enforcing naming conventions, or registering subclasses.

ORMs like Django's use metaclasses extensively — when you define a `Model` subclass, the metaclass scans its fields, sets up the database mapping, and builds query managers automatically.

The practical advice: metaclasses are powerful but complex. Most use cases for metaclasses can be handled more simply with class decorators or `__init_subclass__` (Python 3.6+). Reach for metaclasses only when you genuinely need to control the class creation process itself.

---

Q16. Explain the difference between `multiprocessing`, `threading`, and `asyncio` in Python. When do you use each?

A16.
The right tool depends on what's bottlenecking you.

`threading` is for I/O-bound concurrency with blocking code. Threads share memory, are cheap to create, and Python's GIL releases during I/O. But due to the GIL, CPU-bound threads don't run truly in parallel. Use threads when you have blocking I/O (legacy APIs, `requests` library) and you need simple concurrency without rewriting code to be async.

`asyncio` is for I/O-bound concurrency with non-blocking code. A single-threaded event loop handles thousands of concurrent I/O operations with minimal overhead. Much more scalable than threads for high-concurrency I/O workloads (web servers, scraping, chat bots). Requires async-compatible libraries.

`multiprocessing` is for CPU-bound parallelism. Each process has its own Python interpreter and GIL, so they run truly in parallel across CPU cores. The trade-off: inter-process communication (IPC) is expensive, and processes have higher memory overhead than threads. Use for data processing, image manipulation, numerical computation.

Rule of thumb: I/O-bound + async libraries → `asyncio`; I/O-bound + blocking libraries → `threading`; CPU-bound → `multiprocessing`.

---

Q17. What are Python's built-in data structures and their time complexities?

A17.
`list`: Dynamic array. Append O(1) amortised, insert/delete at index O(n), random access O(1), search O(n).

`dict`: Hash map. Get/set/delete O(1) average, O(n) worst case. In Python 3.7+ dictionaries maintain insertion order.

`set`: Hash set. Add/remove/lookup O(1) average. Great for membership tests and deduplication.

`tuple`: Immutable sequence. Same as list for access but faster creation and smaller memory footprint. Use as a lightweight, hashable record.

`deque` (from `collections`): Doubly linked list. O(1) append/pop from both ends. Use instead of `list` when you need a queue (list.pop(0) is O(n)).

`heapq`: Min-heap implemented on a list. Push/pop O(log n). Use for priority queues, top-k problems.

`collections.Counter`: Dict subclass for counting. `collections.defaultdict`: Dict with a default factory. `collections.OrderedDict`: Dict that remembers insertion order (less relevant since Python 3.7).

---

Q18. How does Python's `property` decorator work?

A18.
`@property` turns a method into a getter for a computed or validated attribute, accessed with dot notation without the call `()`. It's part of Python's descriptor protocol.

You define the getter with `@property`, an optional setter with `@attr.setter`, and an optional deleter with `@attr.deleter`. If you only define the getter, the attribute is read-only.

This lets you start with a simple public attribute, and later add validation or computation without changing the API — existing code accessing `obj.price` continues to work whether `price` is a plain attribute or a property.

```python
class Product:
    def __init__(self, price):
        self._price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative")
        self._price = value
```

---

Q19. What is `__slots__` in Python, and when should you use it?

A19.
By default, Python stores instance attributes in a `__dict__` per instance — a hash map that's flexible but memory-heavy. `__slots__` replaces this dict with a fixed set of slot descriptors declared at the class level, eliminating the per-instance dict.

Benefits: lower memory usage (significant when creating millions of instances), slightly faster attribute access, and prevents accidental creation of new attributes (useful as a safety guard).

Drawbacks: you can only have the declared attributes, no dynamic attribute addition. Inheritance with slots is tricky. Default values can't be set in `__slots__` alone.

Use `__slots__` when: you're creating very large numbers of instances (millions), the attributes are fixed and known at design time, and memory footprint matters (e.g., parser nodes, coordinates, events).

---

Q20. What is the difference between `__new__` and `__init__`?

A20.
`__new__` creates the object — it's a static method that allocates and returns the new instance. `__init__` initialises it — it's called after `__new__` with the new instance as `self`, and sets up its state.

For regular classes, you almost never override `__new__`. You override it when: (1) subclassing an immutable type like `int` or `str` where `__init__` can't change the value after creation — you must intercept at `__new__`, (2) implementing a singleton pattern, or (3) writing a metaclass.

The sequence when you call `MyClass(args)`: Python calls `MyClass.__new__(MyClass, args)`, which returns an instance, then calls `instance.__init__(args)` on it. If `__new__` returns an instance of a different type, `__init__` is not called.

