# 250 Interview Questions — Software Engineering & Data Engineering

---

## Engineering Fundamentals

1. What happens when you type a URL into a browser and press Enter? Walk through the full network lifecycle.
2. Explain the difference between a process and a thread. When would you prefer one over the other?
3. What is a context switch, and why is it expensive?
4. How does virtual memory work, and what role does the page table play?
5. What is a deadlock? What are the four necessary conditions for a deadlock to occur?
6. Explain the difference between optimistic and pessimistic concurrency control.
7. What is a race condition, and how would you prevent one in a multi-threaded application?
8. Describe the differences between TCP and UDP. Give a real-world use case for each.
9. What is the purpose of the TLS handshake, and what happens during it?
10. Explain the difference between symmetric and asymmetric encryption. Where is each used in practice?
11. What is DNS resolution, and what are the different types of DNS records?
12. How does a load balancer distribute traffic, and what are common load-balancing algorithms?
13. What is the OSI model, and why is it useful as a mental framework for debugging network issues?
14. What is the difference between latency and throughput? How do you optimize for each?
15. Explain how a mutex differs from a semaphore. When would you use each?
16. What is memory-mapped I/O, and when is it advantageous over standard file I/O?
17. Describe the role of the kernel in an operating system. What is the difference between user space and kernel space?
18. What is a file descriptor, and how does the operating system manage open files?
19. What are CPU caches (L1, L2, L3), and how does cache locality affect application performance?
20. Explain how connection pooling works and why it improves performance in networked applications.
21. What is the difference between blocking and non-blocking I/O? How do event-driven architectures leverage non-blocking I/O?
22. What is a socket, and how do you establish a TCP connection using sockets?
23. Explain the concept of thrashing in the context of virtual memory. What causes it?
24. How does the operating system manage heap vs. stack memory? What happens during a stack overflow?
25. What is the CAP theorem, and how does it influence your choice of distributed data stores?
26. What is a reverse proxy, and how does it differ from a forward proxy?
27. Explain the concept of eventual consistency. Give an example of a system where it is acceptable.
28. What are the differences between horizontal and vertical scaling? What are the trade-offs of each?
29. How do containers differ from virtual machines at the OS level?
30. What is the purpose of a CDN, and how does it reduce latency for end users?
31. Explain what happens at the hardware level when a process performs a disk read.
32. How does a garbage collector work at a high level? What are the trade-offs of stop-the-world pauses?
33. What is NUMA architecture, and how does it affect performance in multi-socket servers?
34. Explain the concept of back-pressure in distributed systems. Why is it important?
35. What is the difference between a stateful and a stateless service? How does this affect scaling?
36. How does the ARP protocol work, and what role does it play in local network communication?

---

## Programming Fundamentals

37. What is the difference between pass-by-value and pass-by-reference? Give an example in a language you use.
38. Explain the four pillars of object-oriented programming with real-world analogies.
39. What is polymorphism, and how does runtime polymorphism differ from compile-time polymorphism?
40. When would you use an abstract class versus an interface? What are the design implications?
41. What is the difference between a shallow copy and a deep copy? When does this distinction matter?
42. Explain what closures are and give a practical use case for them.
43. What are generics, and why are they useful for writing type-safe, reusable code?
44. What is the difference between a strongly typed and a weakly typed language? Give examples of each.
45. Explain how exception handling works. What is the difference between checked and unchecked exceptions?
46. What is a lambda expression, and how does it relate to functional programming concepts?
47. What is immutability, and why is it advantageous in concurrent programming?
48. Explain the difference between static and dynamic typing. What are the trade-offs?
49. What is the difference between an enumeration and a constant? When would you use each?
50. How does method overloading differ from method overriding?
51. What is a callback function? How does it differ from a promise or a future?
52. Explain the concept of scope and lifetime of a variable. What is the difference between local, global, and block scope?
53. What is type coercion, and what kinds of bugs can it introduce?
54. What is the difference between composition and inheritance? When should you prefer one over the other?
55. How does a hash map work internally? What happens during a hash collision?
56. What are first-class functions, and how do they enable higher-order programming patterns?
57. Explain the difference between mutable and immutable data structures. Give examples of each.
58. What is the purpose of access modifiers (public, private, protected)? How do they enforce encapsulation?
59. What are decorators or annotations, and how are they used in real-world frameworks?
60. What is recursion, and what are the risks of using it without a proper base case?
61. Explain the difference between synchronous and asynchronous execution. How does async/await work?
62. What is the purpose of a constructor? How does it differ from a factory method?
63. What is the difference between an iterable and an iterator? How is lazy evaluation related?
64. How do you handle null or missing values safely in a language you use regularly?
65. What is dependency injection, and why does it improve testability?
66. Explain the concept of serialization and deserialization. When is it used in distributed systems?
67. What is the difference between a struct and a class in languages that support both?
68. How do you use enums effectively to replace magic numbers or string constants?
69. What is the difference between a set, a list, and a tuple? When would you choose each?
70. Explain the concept of operator overloading. What are the dangers of overusing it?
71. What is a coroutine, and how does it differ from a thread?
72. What is the purpose of the `this` or `self` keyword in object-oriented languages?

---

## Coding Best Practices

73. What makes code "clean"? How would you evaluate if a codebase follows clean code principles?
74. How do you decide when to refactor existing code versus rewriting it from scratch?
75. What is the Single Responsibility Principle, and how does violating it create maintenance problems?
76. Explain the Open/Closed Principle with a real-world code example.
77. What is the DRY principle, and when can overly aggressive de-duplication actually hurt readability?
78. How do you write meaningful variable and function names? What naming conventions do you follow?
79. What is the purpose of code reviews? What do you look for when reviewing someone else's code?
80. How do you decide between writing a unit test, an integration test, and an end-to-end test?
81. What is test-driven development (TDD)? What are its advantages and disadvantages?
82. How do you handle technical debt in a fast-moving team?
83. What is the purpose of linting, and how does it improve code quality across a team?
84. Explain the concept of code coverage. Why is 100% code coverage not always a good goal?
85. What are magic numbers, and how do you eliminate them from a codebase?
86. How do you structure error messages to be useful for both developers and end users?
87. When should you use comments in code? When are comments a sign of a problem?
88. What is the Boy Scout Rule in software development?
89. How do you design functions to be easily testable? What characteristics make a function hard to test?
90. Explain the concept of defensive programming. When is it appropriate and when is it overkill?
91. What are feature flags, and how do they help with safe deployments?
92. How do you manage configuration across different environments (dev, staging, production)?
93. What is the difference between a monorepo and a polyrepo? What are the trade-offs?
94. How do you approach logging in production? What information should you log, and what should you avoid?
95. Explain the importance of idempotency in API design. How do you ensure an API endpoint is idempotent?
96. What is semantic versioning, and how does it communicate changes to consumers of your library?
97. How do you avoid tightly coupled code? What design patterns help with decoupling?
98. What are the SOLID principles, and how do they guide everyday software design decisions?
99. How do you handle secrets and credentials in your codebase and CI/CD pipeline?
100. What is continuous integration, and how does it differ from continuous deployment?
101. How do you write a good commit message? What information should it contain?
102. What is the strangler fig pattern, and when would you use it during a migration?
103. How do you prioritize bugs when multiple issues are reported simultaneously?
104. What is pair programming, and in what situations is it most effective?
105. How do you decide on the right level of abstraction for a given problem?
106. What is the YAGNI principle, and how does it prevent over-engineering?
107. Explain the difference between a smoke test and a regression test.
108. What strategies do you use to debug a problem in production that you cannot reproduce locally?

---

## SQL

109. Explain the difference between an INNER JOIN, LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN with examples.
110. What is a subquery, and when would you use it instead of a JOIN?
111. Explain the difference between WHERE and HAVING. Why can't WHERE be used with aggregate functions?
112. What is database normalization? Describe the first three normal forms (1NF, 2NF, 3NF).
113. When would you intentionally denormalize a database? What are the trade-offs?
114. What is an index, and how does it speed up query execution?
115. What is the difference between a clustered index and a non-clustered index?
116. How would you identify and fix a slow SQL query? Walk through your approach.
117. Explain the EXPLAIN or EXPLAIN ANALYZE command. What information does it provide?
118. What is a composite index, and how does the order of columns in it affect query performance?
119. What is a covering index, and when does it eliminate the need for a table lookup?
120. Explain the difference between UNION and UNION ALL. When should you use each?
121. What are window functions? Give an example using ROW_NUMBER(), RANK(), or DENSE_RANK().
122. How do you use the PARTITION BY clause, and how does it differ from GROUP BY?
123. What is a Common Table Expression (CTE), and when is it preferable to a subquery?
124. What is a recursive CTE? Give a practical use case.
125. Explain the concept of a database transaction. What are the ACID properties?
126. What are isolation levels in a database? Describe READ COMMITTED, REPEATABLE READ, and SERIALIZABLE.
127. What is a phantom read, and which isolation level prevents it?
128. Explain the concept of a database lock. What is the difference between a shared lock and an exclusive lock?
129. What is a deadlock in a database? How would you detect and resolve it?
130. What is a stored procedure, and what are the pros and cons of using them?
131. What is the difference between a view and a materialized view?
132. How do you handle NULL values in SQL? What pitfalls should you be aware of with NULLs in comparisons and aggregations?
133. What is a foreign key constraint, and how does it enforce referential integrity?
134. Explain the difference between DELETE, TRUNCATE, and DROP.
135. What is a correlated subquery, and how does it differ from a regular subquery in terms of execution?
136. How do you calculate running totals or cumulative sums in SQL?
137. What is the purpose of the COALESCE function? Give an example.
138. How would you find duplicate records in a table?
139. What are triggers in SQL? When are they useful and when should they be avoided?
140. Explain the concept of query execution plans. How does the query optimizer decide which plan to use?
141. What is an N+1 query problem, and how do you solve it at the application or ORM level?
142. How does database sharding work, and when should you consider it?
143. What is the difference between OLTP and OLAP databases? Give examples of each.
144. How do you write a pagination query efficiently for large datasets?

---

## Data Engineering

145. What is an ETL pipeline? How does it differ from an ELT pipeline?
146. Explain the difference between batch processing and stream processing. When do you use each?
147. What is a data lake, and how does it differ from a data warehouse?
148. What is a data lakehouse, and why has it gained popularity?
149. Describe the medallion architecture (bronze, silver, gold layers). What purpose does each layer serve?
150. What is Apache Spark, and when would you choose it over a traditional SQL engine?
151. Explain the concept of lazy evaluation in Spark. Why is it important?
152. What is the difference between a Spark DataFrame and an RDD? When would you use each?
153. How does Spark handle data partitioning, and why is partition strategy critical for performance?
154. What is a shuffle in Spark, and why is it one of the most expensive operations?
155. What is data skew, and how do you handle it in a distributed processing framework?
156. Explain the difference between narrow and wide transformations in Spark.
157. What is Apache Kafka, and how does it differ from a traditional message queue?
158. Explain the concept of a Kafka topic, partition, and consumer group.
159. How does Kafka guarantee ordering of messages? What are the limitations?
160. What is exactly-once semantics in stream processing, and why is it difficult to achieve?
161. What is backfill in the context of data pipelines, and how do you design pipelines to support it?
162. How do you handle schema evolution in a data pipeline? What tools or formats help with this?
163. What is the difference between Parquet, Avro, and ORC file formats? When would you choose each?
164. Explain the concept of data partitioning in a data lake. How does it improve query performance?
165. What is Apache Airflow, and how does it orchestrate data pipelines?
166. Explain the difference between a DAG and a task in Airflow. How do you define dependencies?
167. How do you handle failures and retries in an Airflow DAG?
168. What is idempotency in the context of data pipelines, and why is it critical?
169. How do you monitor and alert on data pipeline failures in production?
170. What is data lineage, and why is it important for compliance and debugging?
171. Explain the concept of a slowly changing dimension (SCD). What are Type 1, Type 2, and Type 3 SCDs?
172. What is a surrogate key, and why is it used in data warehousing instead of a natural key?
173. How do you perform data quality checks in a pipeline? What tools or frameworks have you used?
174. What is a data contract, and how does it help prevent breaking changes between producers and consumers?
175. Explain the concept of change data capture (CDC). What tools enable it?
176. What is the difference between a star schema and a snowflake schema in data warehousing?
177. How do you deal with late-arriving data in a streaming pipeline?
178. What is a watermark in stream processing, and how does it handle event-time vs. processing-time?
179. How do you secure sensitive data (PII) in a data pipeline?
180. What is data cataloging, and what role does it play in data governance?

---

## MLflow Pipelines

181. What is MLflow, and what are its core components?
182. How does MLflow Tracking work? What information does it log for each experiment run?
183. What is an MLflow experiment, and how does it relate to runs?
184. How do you log parameters, metrics, and artifacts in an MLflow run?
185. What is the MLflow Model Registry, and what problem does it solve?
186. Explain the model lifecycle stages in MLflow (e.g., Staging, Production, Archived). How do you transition between them?
187. How do you compare multiple experiment runs in MLflow to select the best model?
188. What is an MLflow Project, and how does it ensure reproducibility?
189. How does MLflow handle model versioning? How do you roll back to a previous model version?
190. What file formats and flavors does MLflow support for saving models (e.g., sklearn, pytorch, tensorflow)?
191. How do you serve an MLflow model as a REST API endpoint?
192. What is the MLflow Model Signature, and why is it important for input validation?
193. How do you integrate MLflow with a CI/CD pipeline for automated model training and deployment?
194. What are MLflow Recipes (formerly Pipelines), and how do they standardize ML workflows?
195. How do you log custom metrics or visualizations (e.g., confusion matrix, ROC curve) as MLflow artifacts?
196. How would you set up MLflow in a team environment with a shared tracking server and artifact store?
197. What is the difference between the MLflow file store and a remote tracking server backed by a database?
198. How do you use MLflow tags to organize and filter experiment runs?
199. Explain how you would use MLflow to track hyperparameter tuning experiments.
200. How do you handle model drift detection and trigger retraining using MLflow?
201. What are the security considerations when exposing an MLflow tracking server to a team?
202. How does MLflow integrate with cloud-based ML platforms (e.g., Databricks, AWS SageMaker)?
203. How do you log and version datasets alongside models in MLflow?
204. What is the role of the MLmodel file in an MLflow model artifact?
205. How do you automate model promotion from Staging to Production using MLflow APIs?
206. What is a registered model in MLflow, and how does it differ from a logged model?
207. How do you use environment management (conda, virtualenv) with MLflow Projects?
208. Explain how MLflow handles nested runs. When would you use them?
209. How would you evaluate a deployed model's performance over time using MLflow tracking?
210. What are the limitations of MLflow, and when might you choose an alternative tool?
211. How do you set up access control and permissions in the MLflow Model Registry?
212. How does MLflow handle distributed training experiments across multiple nodes?
213. What strategies do you use to manage storage costs for MLflow artifacts at scale?
214. How do you reproduce a specific MLflow experiment run on a different machine?
215. What is the relationship between MLflow and feature stores? How do they complement each other?

---

## Computer Science Fundamentals

216. What is the difference between a compiler and an interpreter? How does a JIT compiler combine both?
217. Explain the concept of abstraction in computer science. Why is it fundamental to managing complexity?
218. What are the stages of compilation (lexing, parsing, code generation)? What happens at each stage?
219. What is the difference between the stack and the heap in program memory? How does each get allocated and freed?
220. Explain what a pointer is. Why are pointer-related bugs among the most dangerous in systems programming?
221. What is the difference between a linked list and an array at the memory level? How does this affect access patterns?
222. What is Big-O notation, and why is it useful for comparing algorithms without benchmarking?
223. Explain the difference between best-case, average-case, and worst-case time complexity. When does each matter?
224. What is a hash function? What properties make a hash function suitable for use in a hash table?
225. What is the difference between a tree and a graph? Give a real-world example of each.
226. Explain the concept of amortized analysis. Give an example of a data structure that benefits from it.
227. What is a finite state machine, and where is it used in real-world software?
228. Explain the concept of Turing completeness. Why does it matter in programming language design?
229. What is the halting problem, and why is it significant in computer science theory?
230. Describe the difference between concurrency and parallelism. Can you have one without the other?
231. What is the call stack, and how does the runtime use it during function execution?
232. What is tail call optimization, and why do some languages support it while others don't?
233. Explain the difference between deterministic and non-deterministic behavior in a program.
234. What is memoization, and how does it improve performance for overlapping subproblems?
235. What is the memory hierarchy (registers, cache, RAM, disk), and how does it affect program performance?
236. Explain the concept of garbage collection. How do reference counting and tracing collectors differ?
237. What is the difference between a process's text, data, BSS, heap, and stack segments?
238. What is the difference between compiled and interpreted languages at runtime? Give trade-offs.
239. What is a type system, and how does static type checking differ from dynamic type checking?
240. Explain the concept of endianness (big-endian vs. little-endian). When does it matter in practice?
241. What is the Von Neumann bottleneck, and how do modern architectures try to mitigate it?
242. Explain the difference between pass-by-name, pass-by-value, and pass-by-reference evaluation strategies.
243. What is a symbol table, and what role does it play during compilation?
244. What is the difference between strong typing and weak typing? How does it affect runtime safety?
245. Explain the concept of a race condition at the hardware level (e.g., in CPU caches or memory ordering).
246. What is a binary representation of negative numbers, and how does two's complement work?
247. What is the difference between a process control block (PCB) and a thread control block (TCB)?
248. Explain the producer-consumer problem. What synchronization primitives solve it?
249. What is instruction-level parallelism, and how do modern CPUs exploit it through pipelining?
250. Explain the concept of locality of reference (temporal and spatial). How do caches exploit it?