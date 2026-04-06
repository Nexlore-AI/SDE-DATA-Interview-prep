==============================
FILE: C++ — Language-Specific Interview Questions
==============================

### HIGH PRIORITY

---

Q1. What is the difference between stack and heap allocation in C++? When do you use `new`/`delete` vs. automatic storage?

A1.
In C++, objects can be allocated in two main ways at runtime.

**Stack (automatic storage)**: When you declare a variable inside a function (`int x = 5;` or `MyObject obj;`), it lives on the call stack. Memory is allocated when the scope is entered and freed automatically when the scope exits — the constructor is called at creation and the destructor at scope exit. Fast, deterministic, no fragmentation. Limited in size (typically 1–8 MB).

**Heap (dynamic storage)**: `new` allocates memory on the heap and returns a pointer. `delete` (or `delete[]` for arrays) frees it. The heap is large but requires manual management — forgetting to `delete` causes memory leaks; deleting twice causes undefined behaviour.

Modern C++ (C++11 and later) strongly prefers smart pointers over raw `new`/`delete`. Use `std::unique_ptr` for exclusive ownership (replaces most `new`/`delete`) and `std::shared_ptr` for shared ownership. Raw `new`/`delete` is now a code smell in application code.

Use the heap when: the object's lifetime must outlast the current scope, size is only known at runtime, or the object is too large for the stack.

---

Q2. What are smart pointers in C++? Explain `unique_ptr`, `shared_ptr`, and `weak_ptr`.

A2.
Smart pointers are RAII wrappers around raw pointers that automatically manage the pointed-to object's lifetime, eliminating memory leaks and dangling pointer errors.

`std::unique_ptr<T>`: sole ownership. Only one `unique_ptr` can own the object at a time. When the `unique_ptr` goes out of scope, the object is destroyed. Non-copyable but movable — ownership can be transferred with `std::move()`. Zero overhead over a raw pointer. Use this as the default for heap-allocated objects.

`std::shared_ptr<T>`: shared ownership via reference counting. Multiple `shared_ptr`s can point to the same object; the object is destroyed when the last `shared_ptr` to it is destroyed. Has overhead: a control block with two atomic counters (strong ref count, weak ref count). Use when multiple owners genuinely need to share an object's lifetime.

`std::weak_ptr<T>`: non-owning observer of a `shared_ptr`-managed object. Doesn't affect the reference count. Must be converted to a `shared_ptr` via `lock()` before accessing the object (returns null if the object was already destroyed). Used to break reference cycles — if A and B `shared_ptr` to each other, neither is ever destroyed. Make one side a `weak_ptr`.

---

Q3. What is RAII (Resource Acquisition Is Initialization)?

A3.
RAII is C++'s most important idiom. The core idea: tie resource ownership to object lifetime. Acquire the resource in the constructor; release it in the destructor. Because destructors are called deterministically when objects go out of scope (or when a `unique_ptr` is destroyed), resources are always released even if exceptions are thrown.

This applies to any resource: heap memory (`unique_ptr`), file handles (`std::fstream`), mutex locks (`std::lock_guard`, `std::unique_lock`), database connections, sockets, GPU memory.

The benefit: you don't need to scatter `free`/`close`/`unlock` calls everywhere, and exception safety comes for free. Compare to Go/Java where you need `defer`/`finally` — in C++ the destructor handles it automatically.

Example: `std::lock_guard<std::mutex> lock(mtx);` acquires the mutex on construction and releases it when `lock` goes out of scope — no need to call `unlock()` and no lock leaks if an exception is thrown.

---

Q4. Explain C++ move semantics and rvalue references.

A4.
Before C++11, returning or copying objects that owned resources (like a `vector` with heap-allocated elements) required a deep copy — expensive. Move semantics allow transferring ownership of those resources instead of copying, leaving the source in a valid but empty state.

An rvalue reference `T&&` binds to temporaries and values that are about to be destroyed. The move constructor and move assignment operator take `T&&` parameters and "steal" the resource (e.g., the internal pointer and size of a vector) from the source, then null out the source.

`std::move(x)` is a cast that says "treat x as an rvalue" — it doesn't actually move anything, it just enables the move constructor/assignment to be called. After moving from `x`, don't use `x` except to assign or destroy it.

This is critical for performance. Returning a large `vector` from a function is essentially free — Return Value Optimisation (RVO) or the move constructor ensures no copy. Push objects into containers with `emplace_back` to construct in-place, or `push_back(std::move(obj))` to move.

`std::forward<T>(arg)` in template code "perfect-forwards" an argument — preserving its value category (lvalue or rvalue) to avoid unnecessary copies in generic code.

---

Q5. What are virtual functions and vtables? How do they enable polymorphism?

A5.
A `virtual` function in C++ enables runtime polymorphism — the correct function is chosen based on the actual dynamic type of the object, not the declared static type of the pointer/reference.

Under the hood, for every class with virtual functions, the compiler creates a **vtable** (virtual dispatch table) — an array of function pointers for each virtual function. Each object of such a class has a hidden **vptr** (virtual pointer) as its first field, pointing to the class's vtable.

When you call `base->virtualMethod()`, the CPU dereferences the vptr, looks up the function pointer in the vtable at the method's slot, and calls it. This is an indirect call with one extra pointer dereference — slightly more expensive than a direct call, but the cost is negligible for typical code.

A class with at least one pure virtual function (`= 0`) is abstract — it can't be instantiated. Subclasses must override all pure virtual functions.

**Important**: always declare destructors `virtual` in base classes that will be used polymorphically. Without a virtual destructor, `delete base_ptr` where `base_ptr` points to a derived object only calls the base destructor — the derived part's destructor is not called, leaking its resources.

---

Q6. What is the difference between `const`, `constexpr`, and `consteval` in C++?

A6.
`const` means the value won't be modified through this particular binding. It's a compile-time or runtime concept. A `const` function parameter can't be changed inside the function. A `const` member function doesn't modify the object's state (`*this` is const).

`constexpr` (C++11) means the value or function *can* be evaluated at compile time if all inputs are available at compile time. If called with runtime values, it's evaluated at runtime like a normal function. Apply it to variables for compile-time constants, and to functions to enable compile-time computation.

`consteval` (C++20) is stronger: the function *must* be evaluated at compile time. A compile error occurs if it's called with runtime arguments. Use for functions that only make sense at compile time.

The progression: `const` is a runtime safety guarantee; `constexpr` enables optional compile-time evaluation; `consteval` mandates compile-time evaluation.

In practice, prefer `constexpr` for constants over `#define` and `const` — it's type-safe, scoped, and debugger-visible.

---

Q7. What are templates in C++, and how do they differ from Java generics?

A7.
C++ templates are a compile-time code generation mechanism. When you write `template<typename T> T max(T a, T b)`, the compiler generates a separate specialised version of the function for each type `T` it's called with. This is **monomorphisation** — full type information is available at runtime, there's no boxing, and the compiler can inline and optimise each specialisation.

Java generics use **type erasure** — all type information is removed at compile time and a single generic version exists at runtime. Java can't do `new T()` or `T[]` without workarounds; C++ can.

C++ templates can express things Java can't: **template specialisation** (different implementations for specific types), **variadic templates** (`template<typename... Args>`), **SFINAE** and **concepts** (C++20) for constraining which types a template accepts.

**Concepts** (C++20) replace SFINAE with readable, compiler-enforced constraints:
```cpp
template<std::integral T>
T add(T a, T b) { return a + b; }
```
This replaces cryptic `enable_if` patterns.

Template errors are notoriously verbose — the error is reported at instantiation, not at definition. This is improving in C++20 with concepts.

---

Q8. What is undefined behaviour (UB) in C++? Why is it dangerous?

A8.
Undefined behaviour means the C++ standard makes no guarantees about what happens — the compiler is free to do anything, including producing code that appears to work, crashes randomly, or produces security vulnerabilities.

Common causes: dereferencing a null or dangling pointer, reading an uninitialised variable, signed integer overflow, out-of-bounds array access, data races (concurrent unsynchronised access to shared mutable state), using an object after it's been moved from, calling `delete` twice on the same pointer.

Why is it dangerous? The compiler optimises under the assumption that UB never occurs. If UB exists, the compiler's assumptions are violated and the generated code may behave in completely unexpected ways — not just crash, but silently produce wrong results or create exploitable security holes. The problem is often latent and only surfaces under specific optimisation levels, architectures, or data patterns.

Defences: use sanitisers (`-fsanitize=address,undefined` with clang/GCC), use smart pointers and RAII to avoid dangling pointers, use `std::array` instead of raw arrays (bounds checking available), use static analysis tools (clang-tidy, Coverity), and enable compiler warnings (`-Wall -Wextra -Werror`).

---

Q9. What is the Rule of Three / Rule of Five / Rule of Zero in C++?

A9.
These rules govern when you need to define special member functions.

**Rule of Three (pre-C++11)**: If you define a custom destructor, copy constructor, or copy assignment operator, you probably need to define all three. These are needed when your class manages a resource (raw pointer, file handle) — the compiler-generated versions do shallow copies, which would result in double-free or resource leaks.

**Rule of Five (C++11+)**: Adds the move constructor and move assignment operator to the three above. If you've defined any of the five, consider whether all five should be defined to ensure correct and efficient behaviour.

**Rule of Zero**: The best rule — design classes so that you need none of these. Use RAII types (`unique_ptr`, `vector`, `string`) to manage resources. The compiler-generated special members will do the right thing (member-wise copy/move/destroy). This is the modern C++ best practice.

If you must write a resource-managing class, define all five (or explicitly `= delete` / `= default` each one to make your intent explicit). An explicitly `= default` destructor is clearer than an empty `{}` destructor.

---

Q10. How does C++ handle exceptions? What is exception safety?

A10.
C++ exceptions propagate by unwinding the call stack — when an exception is thrown, C++ invokes the destructors of all local variables in scope as it unwinds (stack unwinding). This is why RAII ensures resources are released even in the presence of exceptions.

Exceptions are thrown with `throw` and caught with `try`/`catch`. You can throw and catch any type, but the convention is to throw types derived from `std::exception`. Always catch by reference (`catch (const std::exception& e)`) to avoid slicing.

**Exception safety guarantees** (from weakest to strongest):

- **No-throw (nothrow) guarantee**: The operation never throws. Declared with `noexcept`. Example: simple getters, destructors (destructors must not throw in C++11+).
- **Strong guarantee**: If the operation fails, the state is as if it never happened (transactional semantics). Achieved with copy-and-swap idiom.
- **Basic guarantee**: If the operation fails, the object is in a valid (but possibly changed) state. No resources are leaked.
- **No guarantee**: Anything can happen — broken invariants, leaked resources.

Aim for nothrow or strong where possible. Mark destructors, move constructors, and move assignments `noexcept` — the standard library uses this to optimise (e.g., `vector` will move instead of copy only if the element's move constructor is noexcept).

---

### MEDIUM PRIORITY

---

Q11. What are lambda expressions in C++11+?

A11.
Lambdas are anonymous function objects (closures) that can capture variables from the enclosing scope. Syntax: `[capture](params) -> return_type { body }`.

**Capture clause**: `[]` captures nothing, `[=]` captures all local variables by value, `[&]` captures all by reference, `[x, &y]` captures `x` by value and `y` by reference. C++14 adds init captures: `[v = std::move(p)]` to capture move-only types.

Return type is usually deduced. Lambdas with `mutable` can modify captured-by-value variables (which are otherwise `const`).

Lambdas are implemented by the compiler as anonymous structs with an `operator()`. A non-capturing lambda can be implicitly converted to a function pointer.

Used extensively with algorithms: `std::sort(v.begin(), v.end(), [](int a, int b){ return a > b; })`. Generic lambdas (C++14) use `auto` parameters: `[](auto x) { return x * 2; }`. Immediately invoked: `auto result = [](int x){ return x * 2; }(5);`.

---

Q12. What is `std::move` vs. `std::forward`?

A12.
Both are cast utilities — neither actually moves or forwards anything; they just cast to enable certain overload resolutions.

`std::move(x)` unconditionally casts `x` to an rvalue reference (`T&&`). Use it to signal "I'm done with this value, you can steal its resources." After `std::move`, treat the source as valid but unspecified.

`std::forward<T>(x)` is for use in templates. It "perfect-forwards" `x` — if `x` was an lvalue, it stays an lvalue; if it was an rvalue, it becomes an rvalue. This preserves the value category through a forwarding (universal) reference. Without `forward`, named parameters inside a template function are always lvalues, so you'd lose move semantics.

Pattern: templates with forwarding references use `std::forward` to pass arguments along while preserving their move/copy intent; application code uses `std::move` to explicitly transfer ownership.

---

Q13. What is the STL? Name key containers and algorithms.

A13.
The Standard Template Library is the part of the C++ standard library providing generic containers, algorithms, and iterators.

**Sequence containers**: `std::vector` (dynamic array, O(1) amortised append, O(1) random access), `std::deque` (double-ended O(1) push/pop), `std::list` (doubly linked, O(1) insert/delete with iterator), `std::array` (fixed-size on stack), `std::string`.

**Associative containers** (tree-based, sorted): `std::map`, `std::set`, `std::multimap`, `std::multiset` — O(log n) operations.

**Unordered containers** (hash-based): `std::unordered_map`, `std::unordered_set` — O(1) average, O(n) worst.

**Container adapters**: `std::stack`, `std::queue`, `std::priority_queue` — wrappers over sequence containers.

**Key algorithms** in `<algorithm>`: `std::sort` (introsort, O(n log n)), `std::find`, `std::count`, `std::copy`, `std::transform`, `std::accumulate`, `std::binary_search`, `std::lower_bound`/`upper_bound`, `std::for_each`, `std::remove`/`erase`.

Algorithms operate on iterator ranges and work with any compatible container. C++20 Ranges provide a more composable, pipeline-friendly syntax.

---

Q14. What are the differences between `struct` and `class` in C++?

A14.
In C++, `struct` and `class` are almost identical — they both define a type that can have member variables, member functions, constructors, destructors, inheritance, and templates.

The only difference is the **default access level**: members of a `struct` are `public` by default; members of a `class` are `private` by default. Similarly, `struct` inheritance is `public` by default; `class` inheritance is `private` by default.

By convention: use `struct` for passive data aggregates (plain data holders, POD types), where everything is intentionally public. Use `class` for objects with encapsulated state and behaviour, where the default `private` access expresses intent.

This convention aligns with C++ Core Guidelines. Following it makes code intent clearer — a `struct` signals "data holder", a `class` signals "encapsulated object with invariants."

---

Q15. What is `auto` in C++11+, and when should you use it vs. explicit types?

A15.
`auto` instructs the compiler to deduce the variable's type from its initialiser. `auto x = 5;` deduces `int`; `auto it = myVector.begin();` deduces the iterator type.

Benefits: reduces verbosity (especially for iterators and complex template types), avoids accidental type mismatches, and makes code resilient to type changes. If you change a function's return type, `auto` variables holding its result update automatically.

**AAA (Almost Always Auto)** style: Herb Sutter advocates using `auto` for most declarations. The counterargument: explicit types serve as documentation, making it clear what a variable holds.

Use explicit types when clarity matters — `int count = 0;` is clearer than `auto count = 0;`. Avoid `auto` when the type isn't obvious from the right-hand side. Always be explicit with floating-point to avoid precision surprises: `double d = 3.14;` not `auto d = 3.14f;` (which is `float`).

For range-based for loops, `const auto&` is often the right choice: `for (const auto& item : container)` — avoids copies and prevents modification.

---

Q16. What are `static` member variables and functions in C++?

A16.
A `static` member variable belongs to the class itself, not to any individual instance. There is exactly one copy shared across all objects of the class. Static member variables must be defined (not just declared) outside the class body (except for `inline static` or `constexpr static` in C++17+).

A `static` member function can be called without an object (`MyClass::doSomething()`). It has no `this` pointer and therefore cannot access instance members directly. Use for factory methods, utility functions logically tied to the class, or functions that need access to private static members.

`static` local variables inside functions are initialised once on first call and persist for the lifetime of the program. They're thread-safe by default in C++11+. This is the basis for the **Meyers Singleton**:
```cpp
static MyClass& getInstance() {
    static MyClass instance;
    return instance;
}
```

Note: `static` inside a regular (non-member) function or at file scope means "internal linkage" — the symbol is private to that translation unit. This is unrelated to the class-level meaning.

---

Q17. What is name mangling in C++, and why does it matter when interfacing with C?

A17.
C++ allows function overloading — multiple functions with the same name but different parameter types. To distinguish them at the linker level, the C++ compiler encodes the function name, parameter types, and namespace into a unique mangled symbol name.

C does not mangle names — a function `void foo(int)` in C has the symbol `_foo` (or just `foo` depending on the ABI). If C++ code tries to link with a C library without special handling, the mangled symbol `_Z3fooi` won't match the C symbol `_foo`, causing a linker error.

The fix: `extern "C"` tells the C++ compiler to use C-style linkage (no mangling) for the enclosed declarations:
```cpp
extern "C" {
    void foo(int x);  // C linkage, no mangling
}
```
C headers intended to be used from C++ typically wrap their declarations in:
```cpp
#ifdef __cplusplus
extern "C" {
#endif
// C declarations
#ifdef __cplusplus
}
#endif
```

This is important when writing C++ libraries that need to be called from C, Python (via ctypes/cffi), or any language that links against C ABIs.

---

Q18. What are the differences between `#include <header>` and `#include "header"` in C++?

A18.
`#include <header>` searches only the system/compiler include paths. Used for standard library headers (`<iostream>`, `<vector>`) and installed third-party libraries.

`#include "header"` searches first in the directory of the including file (relative path), then falls back to the system include paths. Used for project-local headers.

**Header guards** or `#pragma once` prevent a header from being included multiple times in a single translation unit, which would cause duplicate definition errors:
```cpp
#ifndef MY_HEADER_H
#define MY_HEADER_H
// declarations
#endif
```
`#pragma once` is a non-standard but universally supported shorthand.

Modern C++20 **modules** (`import std;`, `import mymodule;`) replace headers with a compiled module system that avoids textual inclusion entirely — faster build times, no include order issues, no macro leakage. As of C++20, all major compilers (GCC 11+, Clang 16+, MSVC 2019+) and build systems (CMake 3.28+) have broad support for modules.

---

Q19. What is the difference between `delete` and `delete[]` in C++?

A19.
`new` allocates a single object; `delete` frees it and calls its destructor. `new[]` allocates an array; `delete[]` frees the array and calls the destructor on each element. Using `delete` on a `new[]`-allocated array (or vice versa) is undefined behaviour.

The array size is typically stored by the runtime in a header before the allocated block, so `delete[]` can call the right number of destructors. The mechanism is implementation-defined.

In modern C++, you should rarely use `new[]` directly. Prefer `std::vector<T>` for dynamic arrays and `std::make_unique<T[]>(n)` for unique-ownership arrays. Both handle the `delete[]` automatically.

The rule: every `new` is paired with a `delete`; every `new[]` is paired with `delete[]`. In well-designed modern C++ with RAII and smart pointers, you should have neither in application code.

---

Q20. What are inline functions and the ODR (One Definition Rule)?

A20.
The **One Definition Rule** (ODR) states that in a C++ program, every non-inline function, variable, class, and template must be defined exactly once across all translation units. Defining the same function in two `.cpp` files and linking them causes a linker error for duplicate symbols.

`inline` functions are an exception: the same inline function definition can appear in multiple translation units (this is required when you define a function in a header included by multiple `.cpp` files). The `inline` keyword originally suggested inlining the function body at call sites to avoid function call overhead, but modern compilers make that decision independently of the keyword.

In practice, `inline` in header-defined functions is mostly just an ODR exemption signal to the compiler and linker. Compiler-level inlining is controlled by heuristics, profile-guided optimisation, and `__attribute__((always_inline))` / `[[msvc::forceinline]]`.

`constexpr` functions are implicitly `inline`. Template functions defined in headers are also implicitly inline-eligible for ODR purposes. For class member functions defined inside the class body, they're implicitly inline too.

