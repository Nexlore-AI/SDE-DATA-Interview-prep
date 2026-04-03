==============================
FILE: Data Structures & Algorithms (DSA)
==============================

### HIGH PRIORITY

---

Q1. What is Big-O notation, and why is it more useful than benchmarking for comparing algorithms?

A1.
Big-O describes how an algorithm's time or space grows relative to input size — it gives you the *rate of growth*, not the exact runtime. O(n) means linear growth, O(n²) means quadratic, O(log n) means it barely grows.

Benchmarking tells you how fast something runs on *this* machine, *this* data, *right now*. Big-O tells you how it *scales*. An O(n²) algorithm might beat an O(n log n) one for 100 items, but at a million items, the O(n²) one is toast. Big-O captures that structural difference.

It's also hardware-independent. You can compare algorithms on a whiteboard without running code. That's why it's the standard language in interviews and design discussions — it communicates scalability in a single expression.

---

Q2. What is the difference between an array and a linked list at the memory level? How does this affect access, insertion, and deletion?

A2.
An array is a contiguous block of memory — elements sit right next to each other. A linked list is a chain of nodes, where each node holds a value and a pointer to the next node, scattered across memory.

This has direct performance implications. Arrays give you O(1) random access — `arr[i]` is just base address plus offset. Linked lists need O(n) to reach element i because you have to follow pointers from the head.

But insertion and deletion at the beginning or middle? A linked list does it in O(1) once you have a reference to the node — just rewire pointers. An array needs to shift all subsequent elements, which is O(n).

There's also cache performance. Arrays are cache-friendly because of spatial locality — sequential elements are in the same cache line. Linked lists scatter nodes across memory, causing cache misses. In practice, this means arrays are way faster than linked lists for most real-world workloads, even when linked lists have the theoretical advantage.

---

Q3. How does a hash map work internally? What happens during a hash collision, and what strategies resolve it?

A3.
A hash map stores key-value pairs. When you insert a key, it runs the key through a hash function that produces an index into an internal array (the bucket array). The value is stored at that index.

A hash collision happens when two different keys hash to the same index. There are two main strategies to handle this:

**Chaining**: Each bucket holds a linked list (or a tree for Java's HashMap when chains get long). Colliding entries are just appended to the list at that bucket. Lookups walk the chain.

**Open addressing**: If a bucket is occupied, you probe for the next available slot — linear probing (check next slot), quadratic probing (check slot+1, slot+4, slot+9...), or double hashing. All entries live in the array itself.

The hash function quality is critical. A bad hash function that clusters keys into a few buckets degrades O(1) lookups to O(n). A good hash function distributes keys uniformly.

The load factor (entries / buckets) determines when the map resizes — typically at 0.75, the internal array doubles and everything gets rehashed. That's why insertions have O(1) *amortized* cost.

---

Q4. What is a stack vs. a queue? Give a real-world use case for each.

A4.
A stack is LIFO — Last In, First Out. The most recent item is the first to come out. Think of a stack of plates — you add to the top and remove from the top.

A queue is FIFO — First In, First Out. The earliest item is processed first. Think of a checkout line — first person in line gets served first.

Real-world use cases: the call stack is literally a stack — each function call pushes a frame, and returning pops it. Undo functionality in editors is also stack-based — the last action is undone first.

Queues are used everywhere in backend systems — task queues (Celery, SQS), message queues (RabbitMQ, Kafka partitions), request buffering in web servers. Any time you need fair, ordered processing, you reach for a queue.

---

Q5. What is the difference between a binary search tree and a balanced BST (e.g., AVL, Red-Black)? Why does balancing matter?

A5.
A binary search tree (BST) organizes nodes so that for any node, everything in the left subtree is smaller and everything in the right subtree is larger. This gives O(log n) search, insert, and delete — *if the tree is balanced*.

The problem is, if you insert sorted data into a plain BST, it degenerates into a linked list — every node only has a right child. Now your O(log n) operations become O(n). That defeats the whole purpose.

Balanced BSTs like AVL trees and Red-Black trees fix this by automatically rebalancing after insertions and deletions. AVL trees are strictly balanced (height difference of at most 1 between subtrees), so lookups are faster. Red-Black trees are less strictly balanced but rebalance faster, so insertions and deletions are cheaper.

In practice, you rarely implement these yourself. But understanding why balancing matters explains why TreeMap in Java and std::map in C++ use Red-Black trees internally.

---

Q6. When would you use a heap (priority queue), and what is its time complexity for insert and extract-min?

A6.
A heap is used when you need quick access to the minimum (or maximum) element. It's a complete binary tree where each parent is smaller than its children (min-heap).

Insert is O(log n) — you add to the bottom and "bubble up." Extract-min is O(log n) — you remove the root, put the last element at the root, and "bubble down." Peeking at the min is O(1).

Use cases: scheduling (always process the highest-priority task next), Dijkstra's shortest path, finding the k largest elements in a stream, merge k sorted lists. Anytime you're repeatedly asking "what's the current best?" — that's a heap.

In Python, `heapq` gives you a min-heap. In Java, `PriorityQueue`. They're surprisingly common in real system code too — like priority scheduling in OS kernels.

---

Q7. What is the difference between BFS and DFS in terms of use cases and space complexity?

A7.
BFS (Breadth-First Search) explores level by level — all neighbors first, then neighbors' neighbors. Uses a queue. DFS (Depth-First Search) goes as deep as possible down one path before backtracking. Uses a stack (or recursion).

Space complexity: BFS uses O(w) where w is the maximum width of the graph/tree — at the widest level, all those nodes sit in the queue. DFS uses O(h) where h is the maximum depth — it only keeps the current path in memory.

Use case alignment: BFS finds the shortest path in unweighted graphs — it's guaranteed to find the closest match first. DFS is better for exploring all possibilities — like maze solving, topological sorting, or detecting cycles.

For trees, BFS gives you level-order traversal (useful for "print tree level by level"). DFS gives inorder, preorder, postorder traversals.

---

Q8. What is the difference between a tree and a graph? How does this affect traversal strategies?

A8.
A tree is a special case of a graph — it's a connected, acyclic graph with a root node. Every node has exactly one path from the root. A graph can have cycles, multiple paths, and no designated root.

This affects traversal because in a tree, you never revisit nodes — there are no cycles. You can do simple recursive DFS without tracking visited nodes. In a graph, you *must* track visited nodes, or you'll loop forever in cycles.

Trees have hierarchical traversals (preorder, inorder, postorder, level-order) that don't apply to general graphs. Graphs need BFS and DFS with a visited set, and additional algorithms for things like shortest paths (Dijkstra, Bellman-Ford) and minimum spanning trees (Kruskal, Prim).

---

Q9. Why is sorting important as a preprocessing step? Compare the trade-offs between merge sort, quick sort, and heap sort.

A9.
Sorting unlocks efficient algorithms. Once data is sorted, binary search becomes O(log n), finding duplicates becomes O(n), merge operations are trivial, and many problems become simpler (two-pointer techniques, for example). A lot of real-world data processing starts with sorting.

Merge sort: O(n log n) guaranteed, stable (preserves order of equal elements), but uses O(n) extra space. Great when stability matters or data is on disk (external sorting).

Quick sort: O(n log n) average but O(n²) worst case (with bad pivot choices). In practice, it's the fastest for in-memory sorting because of excellent cache locality. It's what most standard libraries use (with optimizations to avoid the worst case).

Heap sort: O(n log n) guaranteed, in-place (O(1) extra space), but poor cache performance because it jumps around the array. Rarely used in practice for full sorting, but the heap data structure is invaluable for partial sorting (top-k problems).

---

Q10. What is hashing, and what makes a good hash function for a hash table?

A10.
Hashing transforms an input (key) into a fixed-size integer (hash code) that serves as an index into an array. The goal is to make lookups O(1) by jumping directly to where the value should be stored.

A good hash function for a hash table needs:
- **Uniformity**: distributes keys evenly across buckets. Clustering kills performance.
- **Determinism**: same key always produces the same hash.
- **Speed**: it's called on every lookup, so it needs to be fast.
- **Avalanche effect**: small changes in input produce large changes in output — `"abc"` and `"abd"` should hash to very different values.

Note that hash table hash functions are *not* the same as cryptographic hash functions (SHA-256). Crypto hashes need to be one-way and collision-resistant at a much stricter level. Hash table hashes prioritize speed.

---

### MEDIUM PRIORITY

---

Q11. What is amortized analysis? Give an example of a data structure whose worst-case per-operation cost is misleading without it.

A11.
Amortized analysis looks at the average cost per operation over a sequence of operations, rather than the worst-case cost of a single operation. Some individual operations are expensive, but they happen rarely enough that the average stays low.

The classic example is dynamic array (ArrayList, Python list). Appending is O(1) most of the time — you just put the element at the end. But when the array is full, it needs to resize (typically double), which copies all elements — O(n). However, this doubling happens so infrequently that spread over n inserts, the total cost is O(n), giving O(1) amortized per insert.

If you only looked at the worst case (O(n) per insert), you'd think dynamic arrays are slow. Amortized analysis reveals they're actually efficient for append-heavy workloads.

---

Q12. What is the difference between a trie and a hash map for string lookups? When would you prefer one?

A12.
A hash map gives you O(1) average lookup for exact keys. A trie (prefix tree) stores strings character by character in a tree structure — each path from root to leaf is a string.

The key difference: a trie supports prefix operations natively. Finding all words that start with "auto" is O(k) where k is the prefix length — you just walk down the tree. With a hash map, you'd need to scan all keys.

Use a hash map for exact key-value lookups — it's faster and simpler. Use a trie when you need autocomplete, prefix matching, or spell-checking. Search engines and IP routing tables (longest prefix match) use tries.

The trade-off: tries use more memory because each character gets its own node. Hash maps are more memory-efficient for most use cases.

---

Q13. What is a bloom filter, and where is it used in practice?

A13.
A bloom filter is a space-efficient probabilistic data structure that answers "is this element in the set?" It can tell you "definitely not" or "probably yes" — it has false positives but no false negatives.

It works by hashing the element through multiple hash functions and setting bits in a bit array. To check membership, you hash and check if all bits are set. If any bit is 0, the element is definitely not in the set. If all bits are 1, it's *probably* in the set (other elements might have set those bits).

Real-world uses: databases use bloom filters to avoid expensive disk reads — check the bloom filter first, and only hit disk if it says "probably yes." CDNs use them to check if content is cached. Chrome used to use one for malicious URL checking. Cassandra and HBase use them to skip SSTables that don't contain a requested key.

---

Q14. What is the difference between a directed and an undirected graph? How does this affect shortest-path algorithms?

A14.
In an undirected graph, edges have no direction — if A connects to B, then B connects to A. In a directed graph (digraph), edges have direction — A→B doesn't imply B→A.

For shortest-path algorithms, both BFS (unweighted) and Dijkstra (weighted, non-negative) work on both types. But directed graphs introduce complexities: a node might be reachable in one direction but not the other. You might have cycles in one direction that don't exist in the other.

Directed graphs also enable topological sorting (only works on DAGs — directed acyclic graphs) and have concepts like strongly connected components that don't exist in undirected graphs.

Bellman-Ford handles negative edge weights and works on directed graphs. For undirected graphs with negative weights, you generally can't have negative cycles (a negative-weight undirected edge creates a cycle by traversing it back and forth).

---

Q15. When would you use a disjoint set (union-find) data structure?

A15.
Union-find is used when you need to efficiently track which elements belong to the same group and merge groups together. It supports two operations: `find(x)` — which group does x belong to? And `union(x, y)` — merge the groups containing x and y.

With path compression and union by rank, both operations are nearly O(1) — amortized O(α(n)), where α is the inverse Ackermann function (practically constant).

Classic use cases: Kruskal's minimum spanning tree algorithm (add edges and check if they create a cycle), connected components in a network, detecting if two nodes are in the same group, and social network clustering.

---

Q16. What is the difference between time complexity and space complexity? When would you trade one for the other?

A16.
Time complexity is how the runtime grows with input size. Space complexity is how the memory usage grows. They're often in tension.

You'd trade space for time in most production systems — memory is cheaper than latency. Caching (memoization) is the classic example: store previously computed results to avoid recomputing. Hash maps trade O(n) space for O(1) lookups instead of O(n) linear search.

You'd trade time for space in memory-constrained environments — embedded systems, mobile, or when dealing with massive datasets. Streaming algorithms, for example, process data in one pass with minimal memory instead of loading everything.

---

Q17. What is a monotonic stack, and what class of problems does it solve efficiently?

A17.
A monotonic stack maintains elements in strictly increasing or decreasing order. When you push a new element, you pop everything that violates the monotonic property. The key insight is that each element is pushed and popped at most once, so the total work across n elements is O(n).

It's perfect for problems like "next greater element" — for each element in an array, find the first element to the right that's larger. Brute force is O(n²), but a monotonic decreasing stack solves it in O(n).

It's also used for histogram-based problems (largest rectangle in histogram), stock span problems, and trapping rain water. The pattern is: you need to find some relationship between elements based on their relative order and proximity.

---

Q18. What is topological sorting, and where is it used in real systems?

A18.
Topological sort orders the nodes of a directed acyclic graph (DAG) so that for every edge A→B, A comes before B in the ordering. It only works on DAGs — if there's a cycle, no valid ordering exists.

Real-world uses: build systems (Make, Bazel) use it to determine compilation order — compile dependencies before dependents. Package managers use it to resolve installation order. Airflow DAGs are literally topological — tasks run in dependency order.

Spreadsheet cell evaluation order, course prerequisite planning, and database migration sequencing all use topological sorting. Kahn's algorithm (BFS-based) and DFS-based approaches both run in O(V + E).

---

### LOW PRIORITY

---

Q19. What is a skip list, and how does it compare to a balanced BST?

A19.
A skip list is a layered linked list where higher layers act as express lanes — you skip ahead at the top and descend to lower layers as you approach your target. Think of it like an index over an index.

It gives O(log n) search, insert, and delete — same as a balanced BST — but with a simpler implementation. Balanced BSTs (AVL, Red-Black) need complex rotation logic. Skip lists just use randomized promotion of nodes to higher levels.

Redis uses skip lists for sorted sets instead of balanced trees — the simplicity makes the code easier to maintain and debug. The trade-off: skip lists use more memory (the extra layers of pointers) and have probabilistic guarantees rather than strict worst-case bounds.

---

Q20. What is an LRU cache, and what data structures power an O(1) implementation?

A20.
LRU (Least Recently Used) cache evicts the least recently accessed item when the cache is full. It needs O(1) for both get and put operations.

The implementation combines a hash map and a doubly linked list. The hash map maps keys to nodes in the linked list, giving O(1) lookup. The linked list maintains access order — the most recently used item is at the head, least recently used at the tail. On access, you move the node to the head (O(1) with doubly linked list). On eviction, you remove the tail (O(1)).

Every caching layer uses this pattern — from CPU caches to application caches (Memcached, Redis eviction policies). Python's `functools.lru_cache` and Java's `LinkedHashMap` implement it out of the box.

---

Q21. What is the difference between a B-tree and a B+ tree? Why are they used in databases?

A21.
A B-tree stores keys and values in both internal nodes and leaf nodes. A B+ tree stores values only in leaf nodes — internal nodes just hold keys for navigation. Also, B+ tree leaf nodes are linked together, allowing efficient range scans.

Databases use B+ trees because:
1. Internal nodes only store keys, so more keys fit per node, meaning the tree is shallower — fewer disk reads to reach any key.
2. Linked leaf nodes make range queries (WHERE age BETWEEN 20 AND 30) sequential reads instead of tree traversals.
3. All values are at the same depth, so query time is predictable.

This is why indexes in PostgreSQL, MySQL, and most relational databases are B+ tree based. The fan-out matches disk block sizes perfectly, minimizing I/O.

---

Q22. What is consistent hashing, and why is it important in distributed systems?

A22.
Regular hashing (key % N servers) breaks badly when you add or remove a server — almost every key remaps, causing a massive cache stampede. Consistent hashing puts both keys and servers on a hash ring. Each key is assigned to the first server clockwise from its position.

When you add a server, only the keys between it and the previous server need to move — roughly 1/N of all keys instead of almost all of them. Same when removing a server.

It's used in distributed caches (Memcached), CDNs, distributed databases (Cassandra, DynamoDB), and load balancers. Virtual nodes (multiple positions per server on the ring) help with load balancing when servers have different capacities.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q23. What is dynamic programming? How do you identify when a problem can be solved with DP? Explain memoization vs tabulation.

A23.
Dynamic programming solves problems by breaking them into overlapping sub-problems, solving each sub-problem once, and storing the result. It works when a problem has two properties: **optimal substructure** (optimal solution contains optimal solutions to sub-problems) and **overlapping sub-problems** (same sub-problems are solved repeatedly).

**How to identify DP problems**: The question says "find the minimum/maximum," "count the number of ways," or "is it possible to...". The brute-force solution has exponential time due to redundant computation. Classic signals: Fibonacci, knapsack, longest common subsequence, coin change, grid paths.

**Memoization (top-down)**: Write the recursive solution naturally, add a cache (dictionary/array) to store results of sub-problems. If a sub-problem is already solved, return the cached result. Start from the big problem, go down.

**Tabulation (bottom-up)**: Build a table iteratively from the smallest sub-problems up to the final answer. No recursion, no stack overflow risk. Usually more space-efficient because you can often optimize to keep only the last row/column.

**Example — Fibonacci**: Recursive = O(2^n). Memoized = O(n) time, O(n) space. Tabulated = O(n) time, O(1) space (only need the last two values).

**Interview tip**: Start with the recursive brute-force, identify overlapping sub-problems, add memoization, then optimize to tabulation if needed.

---

Q24. What is the two-pointer technique? What types of problems does it solve?

A24.
Two pointers uses two indices that move through a data structure (usually a sorted array or linked list) to find pairs, subarrays, or partitions in O(n) time instead of O(n²).

**Common patterns**:
- **Opposite-direction pointers**: Start one at the beginning, one at the end. Move inward based on conditions. Classic: two-sum in a sorted array — if sum < target, move left pointer right; if sum > target, move right pointer left.
- **Same-direction (fast and slow)**: Both start at the beginning, move at different speeds. Detect cycles in linked lists (Floyd's algorithm). Find the middle of a linked list.
- **Partition**: Move elements based on a condition. Dutch National Flag (sort 0s, 1s, 2s). Quicksort partition.

**Problems it solves**: Two-sum (sorted), three-sum, container with most water, remove duplicates from sorted array, merge sorted arrays, palindrome checking, trapping rain water.

**Why interviewers love it**: Tests if you can optimize a brute-force O(n²) to O(n) by exploiting sorted order or other structure. It's a pattern you should recognize instantly.

---

Q25. What is the sliding window technique? When do you use fixed vs variable-size windows?

A25.
Sliding window maintains a contiguous subarray (window) and slides it across the array, updating the answer as the window moves. Converts O(n²) brute-force into O(n).

**Fixed-size window**: Window size k is given. Slide the window one position at a time — add the new element entering, remove the element leaving. Example: maximum sum subarray of size k, moving average.

**Variable-size window**: Window size changes based on a condition. Expand the right boundary until a condition is violated, then shrink the left boundary until the condition is restored. Example: longest substring without repeating characters, smallest subarray with sum ≥ target, longest substring with at most k distinct characters.

**Template**:
```
left = 0
for right in range(n):
    add arr[right] to window
    while window violates condition:
        remove arr[left] from window
        left += 1
    update answer
```

**Key insight**: Both left and right pointers only move forward — total work is O(n) even though there's a nested loop. Each element is added and removed at most once.

---

Q26. What is binary search? What are its lesser-known applications beyond sorted arrays?

A26.
Binary search finds a target in O(log n) by repeatedly halving the search space. The key requirement: a monotonic property — there's a clear boundary where a condition flips from false to true.

**Beyond sorted arrays**:
- **Search on answer (binary search the answer space)**: "What is the minimum capacity to ship packages within D days?" The answer is between max(weights) and sum(weights). Binary search this range — for each candidate capacity, check if it's feasible in O(n). Total: O(n log S).
- **Finding boundaries**: `bisect_left` and `bisect_right` — find the first/last position where an element can be inserted. Useful for range queries.
- **Rotated sorted array**: Modified binary search — determine which half is sorted and search accordingly.
- **Finding peak element**: Binary search works even on unsorted arrays if there's a local property you can exploit.
- **Minimizing/maximizing with monotonic feasibility**: "What is the minimum time to complete all tasks?" If it's feasible for time T, it's feasible for T+1. Binary search for the minimum feasible T.

**Common mistake**: Off-by-one errors in boundary conditions (`left <= right` vs `left < right`, `mid` vs `mid+1`). Practice the template until it's automatic.

---

Q27. What are greedy algorithms? How do you prove a greedy choice is optimal?

A27.
Greedy algorithms make the locally optimal choice at each step, hoping it leads to a globally optimal solution. They're fast (usually O(n log n) due to sorting) but only work when the greedy choice property holds.

**When greedy works**: The problem has optimal substructure AND making the best local choice doesn't prevent finding the global optimum. Classic examples: activity selection (pick the activity that ends earliest), Huffman coding, Dijkstra's shortest path (non-negative weights), fractional knapsack, interval scheduling.

**When greedy fails**: 0/1 knapsack (taking the highest value/weight ratio item can lead to suboptimal total), longest path (need DP or backtracking), coin change with arbitrary denominations.

**Proving correctness (exchange argument)**: Assume an optimal solution that doesn't use the greedy choice. Show you can swap in the greedy choice without making the solution worse. Therefore, there exists an optimal solution that includes the greedy choice.

**Interview tip**: If you suspect greedy, try to find a counterexample. If you can't, go with greedy and explain your reasoning. Interviewers care more about your thought process than a formal proof.

---

### IMPORTANT

---

Q28. What is backtracking? How does it differ from brute-force enumeration?

A28.
Backtracking builds solutions incrementally, abandoning a candidate ("pruning") as soon as it's clear it can't lead to a valid solution. It's a refined brute-force — instead of generating all possibilities and checking each, you cut off invalid branches early.

**Template**: Make a choice → recurse → undo the choice (backtrack). The "undo" step is what gives it the name.

**Examples**: N-Queens (place queens row by row, backtrack if a placement attacks existing queens), Sudoku solver, generating all permutations/combinations, word search in a grid, subset sum.

**How it differs from brute-force**: Brute-force generates ALL possible configurations, then filters. Backtracking prunes the search tree — never explores branches that are known to fail. For N-Queens, brute-force checks all n^n placements. Backtracking explores far fewer because it skips entire subtrees early.

**Pruning is the key**: The better your pruning conditions, the faster backtracking runs. Sometimes ordering choices (most constrained first) dramatically reduces the search space.

---

Q29. What is Dijkstra's algorithm, and when does it fail? How does Bellman-Ford handle negative edges?

A29.
**Dijkstra's**: Finds the shortest path from a source to all other vertices. Uses a min-heap (priority queue). At each step, pick the unvisited vertex with the smallest tentative distance, update its neighbors. Time: O((V + E) log V) with a binary heap.

**Where it fails**: Negative edge weights. Dijkstra assumes once a vertex is finalized, its shortest distance won't change. A negative edge can create a shorter path through an already-finalized vertex, breaking this invariant.

**Bellman-Ford**: Handles negative edges. Relaxes all edges V-1 times (in the worst case, the shortest path has at most V-1 edges). If a further relaxation is possible after V-1 iterations, a negative cycle exists. Time: O(V × E) — slower than Dijkstra.

**When to use which**: All non-negative weights → Dijkstra (faster). Negative weights possible → Bellman-Ford. Need all-pairs shortest path → Floyd-Warshall O(V³). Dense graphs → Floyd-Warshall may be simpler.

---

Q30. What is the difference between in-place and out-of-place algorithms? Give examples.

A30.
**In-place**: Uses O(1) extra space (or O(log n) for recursion stack). The input is modified directly. Quicksort (in-place partition), heapsort, reversing an array in-place. Space-efficient but modifies the input — sometimes undesirable.

**Out-of-place**: Creates new data structures to hold results. Merge sort (needs O(n) auxiliary array), creating a new sorted list, most functional programming operations. Uses more memory but preserves the original input.

**Trade-off**: In-place saves memory but complicates code and destroys the input. Out-of-place is simpler to reason about but uses more memory. For interviews, in-place is often preferred because it demonstrates low-level control — but mention the trade-off.

**Subtle cases**: Python's `sorted()` is out-of-place (returns a new list), `.sort()` is in-place. Both are O(n log n) time but differ in space.

---

Q31. What is a segment tree or Fenwick tree (BIT)? What class of problems do they solve?

A31.
Both solve range query problems efficiently — answering questions like "what's the sum of elements from index L to R?" while supporting updates.

**Naive approach**: Array with O(1) updates and O(n) range queries, or prefix sum with O(1) queries but O(n) updates.

**Fenwick tree (Binary Indexed Tree)**: O(log n) for both point updates and prefix sum queries. Compact — uses an array of size n. Leverages binary representation of indices. Simpler to implement but limited to prefix queries and operations that are invertible (sum, XOR — not min/max).

**Segment tree**: O(log n) for range queries and range/point updates. More flexible — supports min, max, sum, GCD, or any associative operation. Can handle range updates with lazy propagation. More code-heavy than Fenwick but strictly more powerful.

**Use cases**: Range sum/min/max queries with updates, count of elements in a range, computational geometry, competitive programming. In production, similar concepts appear in database indexing and time-series aggregation.

---

### GOOD-TO-HAVE

---

Q32. What is the difference between stable and unstable sorting algorithms? When does stability matter?

A32.
A **stable sort** preserves the relative order of equal elements. If two students have the same grade, their original order is maintained after sorting by grade. Merge sort, Timsort (Python's default), and insertion sort are stable.

An **unstable sort** may change the relative order of equal elements. Quicksort and heapsort are unstable.

**When stability matters**: Sorting by multiple criteria sequentially. Sort by name first, then by grade (stable sort). Students with the same grade remain alphabetically ordered. If the sort were unstable, the alphabetical order could be lost.

**In practice**: Most language defaults are stable (Python, Java's `Arrays.sort` for objects, JavaScript). If stability matters and your sort is unstable, you can add the original index as a tiebreaker.

---

Q33. What are common bit manipulation techniques used in interviews?

A33.
- **Check if power of 2**: `n & (n - 1) == 0` (and n > 0). Powers of 2 have exactly one set bit.
- **Count set bits (Hamming weight)**: `n & (n - 1)` removes the lowest set bit. Repeat until zero. Or use `bin(n).count('1')` in Python.
- **XOR tricks**: `a ^ a = 0`, `a ^ 0 = a`. Find the single non-duplicate in an array — XOR all elements. Two non-duplicates — partition by a differing bit and XOR each group.
- **Get/set/clear bit i**: Get: `(n >> i) & 1`. Set: `n | (1 << i)`. Clear: `n & ~(1 << i)`.
- **Swap without temp**: `a ^= b; b ^= a; a ^= b`. Works but unnecessary in practice.

**Interview applications**: Single number (XOR), counting bits, power of two, bit masking for subsets (iterate all subsets of n elements with 2^n masks), encode/decode flags.

---

Q34. What is the difference between iterative and recursive DFS? When does one have an advantage?

A34.
Both visit the same nodes, but the mechanics differ:

**Recursive DFS**: Uses the call stack implicitly. Cleaner code — just call `dfs(neighbor)` for each unvisited neighbor. But limited by stack depth (Python default: 1000). For a graph with 100K nodes in a single chain, recursive DFS hits a stack overflow.

**Iterative DFS**: Uses an explicit stack data structure. More verbose but no stack overflow risk. You control the stack — can handle millions of nodes.

**Subtle difference**: The order of neighbor processing can differ. Recursive DFS processes the first neighbor's entire subtree before the second. Iterative DFS (push all neighbors to stack) processes the last-pushed neighbor first — it visits in reverse neighbor order. To match recursive order exactly, push neighbors in reverse.

**When to choose**: Small inputs or tree problems → recursive (cleaner). Large graphs or production systems → iterative (safe). Some problems (topological sort, cycle detection) are more natural with recursive DFS and backtracking.