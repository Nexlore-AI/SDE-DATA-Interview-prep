# Compiler Design & Theory of Computation — Interview Q&A

---

## Theory of Computation

### 1. What are the types of automata and their corresponding languages?

| Automaton | Language Class | Example | Power |
|-----------|---------------|---------|-------|
| **DFA / NFA** | Regular | `a*b`, email patterns | Least |
| **PDA** (Pushdown Automaton) | Context-Free | Balanced parentheses, HTML nesting | ↑ |
| **LBA** (Linear Bounded) | Context-Sensitive | Natural language grammar | ↑ |
| **Turing Machine** | Recursively Enumerable | Any computable problem | Most |

**Chomsky Hierarchy:** Regular ⊂ Context-Free ⊂ Context-Sensitive ⊂ Recursively Enumerable

---

### 2. What is the difference between DFA and NFA?

| Feature | DFA | NFA |
|---------|-----|-----|
| Transitions | Exactly one per symbol | Zero, one, or multiple |
| ε-transitions | Not allowed | Allowed |
| Determinism | Deterministic | Non-deterministic |
| States explored | One at a time | Multiple simultaneously |
| Implementation | Direct, efficient | Needs subset construction |
| Power | Equal | Equal (can be converted to DFA) |

**Key theorem:** Every NFA can be converted to an equivalent DFA (subset construction), but the DFA may have exponentially more states.

---

### 3. What is a regular expression and what can't it do?

Regular expressions define **regular languages** — patterns over strings.

**Can do:** Pattern matching, lexical analysis, input validation
- `[a-z]+@[a-z]+\.[a-z]+` (simple email)
- `(0|1)*00` (binary strings ending in 00)

**Cannot do (Pumping Lemma):**
- Count matching pairs: `aⁿbⁿ` (n a's followed by n b's)
- Match nested brackets (arbitrary depth)
- Parse programming languages (need CFG)

---

### 4. What is a Context-Free Grammar (CFG)?

A CFG consists of:
- **Terminals:** Actual symbols (a, b, +, *)
- **Non-terminals:** Variables (S, A, B)
- **Productions:** Rules (S → aSb | ε)
- **Start symbol:** S

**Example (balanced parentheses):**
```
S → (S) | SS | ε
```
Generates: (), (()), ()(), (()()), ...

**Example (arithmetic expressions):**
```
E → E + T | T
T → T * F | F
F → (E) | id
```

---

### 5. What is the Pumping Lemma?

The Pumping Lemma is used to **prove a language is NOT regular** (or not context-free).

**For Regular Languages:**
If L is regular, then for sufficiently long strings s ∈ L, we can split s = xyz where:
1. |xy| ≤ p (pumping length)
2. |y| > 0
3. xyⁱz ∈ L for all i ≥ 0

**Proof technique:** Assume L is regular → find a string that can't be pumped → contradiction → L is not regular.

**Classic example:** L = {aⁿbⁿ | n ≥ 0} is NOT regular.

---

### 6. What is a Turing Machine?

A Turing Machine is a theoretical model of computation with:
- **Infinite tape** (memory)
- **Read/write head** (moves left/right)
- **State register** (current state)
- **Transition function:** δ(state, symbol) → (new_state, write_symbol, direction)

**Church-Turing Thesis:** Anything that can be computed can be computed by a Turing Machine. This defines the boundary of what's "computable."

**Undecidable problems:**
- **Halting problem:** Can we determine if a program will halt or loop forever? NO.
- **Rice's theorem:** Any non-trivial property of a program's behavior is undecidable.

---

### 7. What is the difference between decidable and undecidable problems?

| Type | Description | Example |
|------|-------------|---------|
| **Decidable** | TM always halts (yes or no) | "Is this string in the language of this DFA?" |
| **Semi-decidable** | TM halts on yes, may loop on no | "Does this TM accept this string?" |
| **Undecidable** | No TM can solve it | Halting problem, Post correspondence |

---

### 8. Explain P, NP, NP-Complete, and NP-Hard.

| Class | Definition | Example |
|-------|-----------|---------|
| **P** | Solvable in polynomial time | Sorting, shortest path, primality testing |
| **NP** | Verifiable in polynomial time | Sudoku (hard to solve, easy to verify) |
| **NP-Complete** | In NP AND every NP problem reduces to it | SAT, TSP (decision), Graph Coloring |
| **NP-Hard** | At least as hard as NP-Complete (may not be in NP) | TSP (optimization), Halting problem |

**P = NP?** Biggest open question in CS. Most believe P ≠ NP (meaning some problems are fundamentally harder to solve than to verify).

---

## Compiler Design

### 9. What are the phases of a compiler?

```
Source Code → Lexical Analysis → Syntax Analysis → Semantic Analysis
                  ↓                    ↓                    ↓
              Tokens              Parse Tree        Annotated Tree
                                                        ↓
                                            Intermediate Code Gen
                                                        ↓
                                                 Code Optimization
                                                        ↓
                                                Code Generation
                                                        ↓
                                                 Machine Code
```

| Phase | Input | Output | Tools |
|-------|-------|--------|-------|
| **Lexical Analysis** | Source code | Tokens | Lex, Flex |
| **Syntax Analysis** | Tokens | Parse tree (AST) | Yacc, Bison |
| **Semantic Analysis** | AST | Type-checked AST | Custom |
| **IR Generation** | AST | Intermediate code (3-address) | — |
| **Optimization** | IR | Optimized IR | — |
| **Code Generation** | IR | Machine/assembly code | — |

---

### 10. What is lexical analysis (scanning)?

The lexer converts a stream of characters into **tokens** (lexemes + categories).

**Input:** `int x = 42 + y;`
**Output:**
```
<keyword, int>
<identifier, x>
<operator, =>
<integer, 42>
<operator, +>
<identifier, y>
<separator, ;>
```

**Implementation:** Regular expressions → NFA → DFA → scanner.

---

### 11. What is parsing? Explain top-down vs bottom-up.

Parsing builds a **parse tree** from tokens according to the grammar.

| Feature | Top-Down | Bottom-Up |
|---------|----------|-----------|
| Direction | Start from root (S), expand | Start from leaves, reduce to S |
| Type | Recursive descent, LL(1) | LR, SLR, LALR |
| Grammar restriction | No left recursion | More general |
| Tool | Hand-written or LL parser | Yacc, Bison (LALR) |
| Approach | Predict which production to use | Shift tokens, reduce when pattern matches |

**LL(1):** Left-to-right, Leftmost derivation, 1 token lookahead.
**LALR(1):** Most common in practice (Yacc/Bison). Handles most programming language grammars.

---

### 12. What is an Abstract Syntax Tree (AST)?

An AST is a **simplified representation of the parse tree** that removes syntax sugar.

**Expression:** `3 + 4 * 5`
```
Parse Tree:           AST:
    E                  +
  / | \              /   \
 E  +  T            3     *
 |    /|\                / \
 3   T * F              4   5
     |   |
     4   5
```

The AST is what compilers and interpreters actually work with. It drops parentheses, keywords, and other syntactic details.

---

### 13. What is semantic analysis and type checking?

Semantic analysis verifies the **meaning** of the program (syntax is correct, but is it meaningful?).

**Checks:**
- **Type checking:** `int x = "hello"` → type error
- **Scope resolution:** Is variable `x` declared in this scope?
- **Function calls:** Correct number/types of arguments?
- **Array bounds:** Static bounds checking where possible
- **Undeclared identifiers:** Using a variable before declaration

**Symbol table:** Data structure mapping identifiers to their type, scope, and memory location.

---

### 14. What is intermediate code? What is 3-address code?

Intermediate Representation (IR) is a **platform-independent** version of the code.

**3-address code:** Each instruction has at most 3 operands.

```
Source: a = b + c * d

3-address code:
t1 = c * d
t2 = b + t1
a = t2
```

**Why IR:**
- Separates front-end (language-specific) from back-end (machine-specific)
- Optimization passes work on IR
- Same IR for multiple source languages and target architectures

**LLVM IR** is the most prominent example today.

---

### 15. What are common compiler optimizations?

| Optimization | What It Does | Example |
|-------------|-------------|---------|
| **Constant folding** | Evaluate constant expressions at compile time | `x = 3 + 4` → `x = 7` |
| **Dead code elimination** | Remove code that can never execute | `if (false) { ... }` → removed |
| **Common subexpression elimination** | Reuse repeated computations | `a = b+c; d = b+c` → `t=b+c; a=t; d=t` |
| **Loop invariant code motion** | Move unchanging computations outside loop | `x = y * z` inside loop → move out |
| **Strength reduction** | Replace expensive ops with cheaper ones | `x * 2` → `x << 1` |
| **Inlining** | Replace function call with function body | Small functions inlined |
| **Register allocation** | Map variables to CPU registers | Graph coloring algorithm |
| **Loop unrolling** | Duplicate loop body to reduce branch overhead | Reduces loop control overhead |
