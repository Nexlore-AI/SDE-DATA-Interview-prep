# 250 Interview Questions: Software & Data Engineering

---

## Engineering Fundamentals

1. What is the difference between a process and a thread, and when would you choose one over the other?
2. How does virtual memory work, and what problem does it solve?
3. What is a context switch, and why does it have a performance cost?
4. Explain the difference between preemptive and cooperative multitasking.
5. What is a race condition, and how would you detect one in a production system?
6. What is a deadlock? Describe the four conditions required for one to occur.
7. How does a mutex differ from a semaphore?
8. What is the difference between parallelism and concurrency?
9. Explain what a memory leak is and describe strategies to prevent it.
10. What is the difference between stack memory and heap memory?
11. What is a page fault, and how does the OS handle it?
12. How does CPU caching work, and what is cache coherence?
13. What is the difference between a blocking and a non-blocking I/O call?
14. Explain the concept of a file descriptor in Unix-like systems.
15. What is the difference between TCP and UDP, and when would you use each?
16. What happens at each layer of the OSI model when you make an HTTP request?
17. What is a socket, and how does a TCP connection get established?
18. What is the difference between a process's address space and physical memory?
19. What is copy-on-write, and where is it used in operating systems?
20. How does the kernel scheduler decide which thread to run next?
21. What is the difference between a hard link and a symbolic link?
22. What is a system call, and how does it differ from a regular function call?
23. Explain how inter-process communication (IPC) mechanisms like pipes and message queues work.
24. What is the difference between synchronous and asynchronous I/O?
25. How does DNS resolution work from the moment you type a URL in a browser?
26. What is network latency, and what are its primary causes?
27. What is a load balancer, and what strategies can it use to distribute traffic?
28. What is the difference between horizontal and vertical scaling?
29. What is a CDN, and how does it reduce latency for end users?
30. Explain what happens during a TCP three-way handshake.
31. What is the TIME_WAIT state in a TCP connection, and why does it exist?
32. How does the Linux OOM (Out-of-Memory) killer work?
33. What is NUMA architecture, and how does it affect application performance?
34. What is a spin lock, and when is it preferable to a blocking mutex?
35. What is the difference between a monolithic kernel and a microkernel?
36. How does epoll differ from select/poll for handling many simultaneous connections?

---

## Programming Fundamentals

37. What is the difference between pass-by-value and pass-by-reference?
38. What is the difference between static typing and dynamic typing?
39. Explain the concept of type coercion and give an example where it can cause bugs.
40. What is a closure, and how does it capture variables from its enclosing scope?
41. What is the difference between shallow copy and deep copy of an object?
42. Explain the difference between an abstract class and an interface.
43. What is polymorphism, and what are its two main forms?
44. What is method overloading vs. method overriding?
45. What is the difference between composition and inheritance? When do you prefer one over the other?
46. What is a constructor, and what happens when you don't define one in a class?
47. What is a destructor, and when is it called?
48. What is the difference between checked and unchecked exceptions?
49. What does it mean for a function to be idempotent?
50. What is a pure function, and why is it valuable in software design?
51. What is the difference between mutable and immutable objects?
52. Explain what a generator is and how it differs from a regular function.
53. What is tail recursion, and does your language of choice optimize for it?
54. What is the difference between a compiled language and an interpreted language?
55. What are first-class functions, and why do they matter?
56. What is a higher-order function? Give a practical example.
57. What is the difference between == and === (or their equivalents) in the languages you use?
58. What is a null pointer exception, and how do you defensively avoid it?
59. What is the difference between a stack overflow and a heap overflow?
60. What is operator overloading, and what are the risks of using it?
61. How does garbage collection work in a managed runtime like the JVM or CPython?
62. What is reference counting, and what is its main weakness?
63. What is the difference between a class variable and an instance variable?
64. What is a lambda expression, and where would you use one over a named function?
65. What is the difference between eager evaluation and lazy evaluation?
66. What is an enum, and what advantages does it have over using raw constants?
67. What is the purpose of the final (or const/readonly) keyword in your primary language?
68. What is the difference between a struct and a class in languages that support both?
69. What is method chaining, and what design pattern does it typically implement?
70. What is the difference between synchronous and asynchronous function execution in application code?
71. What is a thread-local variable, and when is it useful?
72. What is the significance of the volatile keyword in concurrent programming?

---

## Coding Best Practices

73. What does the Single Responsibility Principle mean, and how do you apply it in practice?
74. How would you decide when to refactor code vs. rewrite it from scratch?
75. What is technical debt, and how do you communicate its impact to non-technical stakeholders?
76. What makes a function name good? Give an example of a poor name and a better one.
77. What is the DRY principle, and when is it acceptable to violate it?
78. What is the YAGNI principle, and how does it guide design decisions?
79. What is the difference between unit tests, integration tests, and end-to-end tests?
80. How do you write a meaningful unit test? What makes a test brittle?
81. What is test-driven development (TDD), and what are its practical benefits and drawbacks?
82. What is code coverage, and why is 100% coverage not always the right goal?
83. How would you approach testing code that depends on an external API?
84. What is a mock, a stub, and a fake? How do they differ?
85. What is the purpose of a code review, and what do you look for when reviewing someone else's code?
86. How do you handle a situation where you disagree with a reviewer's feedback?
87. What is the purpose of a linter, and how does it differ from a formatter?
88. What are the characteristics of clean code as you understand them?
89. What is a magic number in code, and how do you eliminate it?
90. How do you approach logging in an application? What should and shouldn't be logged?
91. What is the difference between an error and an exception in terms of handling strategy?
92. How do you make error messages useful for both developers and operators?
93. What is defensive programming, and can it be taken too far?
94. What is the principle of least privilege, and how does it apply to code design?
95. How would you document a function or module effectively?
96. What is the purpose of a README file, and what should it contain for a production service?
97. What is semantic versioning, and why does it matter for shared libraries?
98. How would you set up a CI/CD pipeline for a new service from scratch?
99. What is feature flagging, and what are its advantages over long-lived feature branches?
100. What is a code smell? Give three examples and explain why each is problematic.
101. How do you approach debugging a bug you cannot reproduce locally?
102. What is the purpose of an assertion in production code, and should assertions replace proper error handling?
103. What strategies do you use to keep dependencies up to date and secure?
104. What is pair programming, and when is it most effective?
105. How do you ensure backward compatibility when changing a public API?
106. What is the strangler fig pattern, and when would you use it?
107. How do you measure the quality of a software system beyond test coverage?
108. What is observability, and how does it differ from monitoring?

---

## SQL

109. What is the difference between an INNER JOIN, LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN?
110. What is a CROSS JOIN, and when would you legitimately use one?
111. What is the difference between WHERE and HAVING, and when must you use HAVING?
112. How does a GROUP BY clause work, and what restrictions does it impose on your SELECT list?
113. What is a subquery, and how does a correlated subquery differ from a non-correlated one?
114. What is a CTE (Common Table Expression), and what advantages does it have over a subquery?
115. What is a window function? Give a practical example of RANK(), ROW_NUMBER(), and LAG().
116. What is the difference between RANK() and DENSE_RANK()?
117. What is the difference between UNION and UNION ALL? When would you choose one over the other?
118. What is an index, and how does it speed up query execution?
119. What is the difference between a clustered index and a non-clustered index?
120. What is index selectivity, and why does it matter for query performance?
121. Under what circumstances can an index actually slow down a query?
122. What is a covering index?
123. What is query execution plan analysis, and how do you use EXPLAIN (or its equivalent)?
124. What is the N+1 query problem, and how do you fix it?
125. What is a transaction, and what does ACID stand for?
126. What is the difference between optimistic and pessimistic locking?
127. What are the SQL isolation levels, and what anomalies does each prevent?
128. What is a dirty read, a non-repeatable read, and a phantom read?
129. What is normalization? Explain 1NF, 2NF, and 3NF with examples.
130. What is denormalization, and when is it the right trade-off?
131. What is a foreign key constraint, and what are the implications of not using one?
132. What is the difference between DELETE, TRUNCATE, and DROP?
133. What is a stored procedure, and what are the pros and cons of heavy use of stored procedures?
134. What is a view, and how does a materialized view differ from a regular view?
135. What is the difference between a primary key and a unique constraint?
136. How would you find and eliminate duplicate rows in a table?
137. What is the difference between EXISTS and IN in a subquery, and which performs better in most databases?
138. How would you pivot rows into columns in SQL without using a pivot function?
139. What is a self-join, and give a real-world scenario where you would use one?
140. What is referential integrity, and how does the database enforce it?
141. What is a sequence or auto-increment column, and what pitfalls come with using it as a distributed primary key?
142. How do you handle NULL values in aggregations and comparisons in SQL?
143. What is the difference between a natural join and an explicit join condition?
144. How would you optimize a slow query where adding an index is not an option?

---

## Data Engineering

145. What is the difference between batch processing and stream processing? Give a use case for each.
146. What is an ETL pipeline, and how does it differ from an ELT pipeline?
147. What is a data warehouse, and how does it differ from a data lake?
148. What is a data lakehouse, and what problem does it solve?
149. What is schema-on-read vs. schema-on-write, and what are the trade-offs?
150. What is data partitioning in a distributed storage system, and why is it important?
151. What is data skew, and how do you diagnose and handle it in a Spark job?
152. What is the difference between a narrow transformation and a wide transformation in Spark?
153. What is a shuffle in Spark, and why is it expensive?
154. What is a broadcast join in Spark, and when should you use it?
155. What are the different Spark deployment modes, and when would you choose each?
156. What is lazy evaluation in Spark, and how does it affect performance optimization?
157. What is the difference between a DataFrame and a Dataset in Spark?
158. What is checkpointing in a streaming pipeline, and why is it necessary?
159. What does exactly-once semantics mean in a streaming system, and how hard is it to achieve?
160. What is watermarking in stream processing, and what problem does it address?
161. What is the difference between event time and processing time in streaming?
162. What is a Kafka topic partition, and how does partition count affect throughput and ordering?
163. What is a Kafka consumer group, and how does rebalancing work?
164. What is the difference between at-least-once, at-most-once, and exactly-once delivery in Kafka?
165. What is log compaction in Kafka, and when is it useful?
166. What is a data pipeline orchestrator, and what problems does it solve that a cron job does not?
167. What is Airflow's DAG, and what makes a good DAG design?
168. What is idempotency in a data pipeline, and how do you design a task to be idempotent?
169. What is backfilling in a data pipeline, and what challenges come with it?
170. What is incremental loading, and how does it differ from a full refresh?
171. What is a slowly changing dimension (SCD), and what are the differences between SCD Type 1, 2, and 3?
172. What is a star schema, and how does it compare to a snowflake schema?
173. What is data lineage, and why is it important for data governance?
174. What is data quality, and how would you implement a data quality framework in a pipeline?
175. What is the role of metadata management in a data platform?
176. What is CDC (Change Data Capture), and what are common approaches to implementing it?
177. What is the difference between a hot path and a cold path in a Lambda architecture?
178. What is Kappa architecture, and what problem does it address compared to Lambda architecture?
179. How would you handle schema evolution in a data pipeline without breaking downstream consumers?
180. What is a Delta table (Delta Lake), and what guarantees does it provide?

---

## MLflow Pipelines

181. What is MLflow, and what are its four core components?
182. What is an MLflow experiment, and how is it different from a run?
183. What parameters, metrics, and artifacts would you typically log for a classification model training run?
184. How does MLflow autologging work, and what are its limitations?
185. What is the MLflow Model Registry, and what problem does it solve?
186. What are the lifecycle stages in the MLflow Model Registry, and what does each represent?
187. How would you transition a model from Staging to Production using the MLflow Model Registry API?
188. What is an MLflow model flavor, and why does the concept exist?
189. What is the difference between logging a model with mlflow.sklearn.log_model vs. mlflow.pyfunc.log_model?
190. What is the pyfunc flavor in MLflow, and when would you use it over a framework-specific flavor?
191. How would you use MLflow to compare two experiments to select the better model?
192. What is MLflow Projects, and how does it help with reproducibility?
193. What is an MLproject file, and what can it specify?
194. How does MLflow integrate with popular ML frameworks like Scikit-learn, PyTorch, and TensorFlow?
195. What is the difference between logging a model as an artifact vs. registering it in the Model Registry?
196. How would you serve an MLflow model as a REST API using built-in tooling?
197. What environment dependencies does MLflow capture when you log a model, and how are they restored at serving time?
198. How would you implement a custom Python model in MLflow's pyfunc interface?
199. What is an MLflow tracking server, and what storage backends does it support?
200. How would you set up MLflow tracking in a multi-user, remote team environment?
201. What are model signatures in MLflow, and why should you always log them?
202. What is input example logging in MLflow, and how does it assist deployment?
203. How would you roll back to a previous model version in MLflow if a new deployment degrades performance?
204. What is the difference between run parameters and run tags in MLflow?
205. How do you handle nested runs in MLflow, and when are they useful?
206. What is the MLflow evaluate API, and what kind of metrics can it compute automatically?
207. How would you integrate MLflow into a CI/CD pipeline for automated model validation?
208. What challenges arise when using MLflow in a distributed training environment, and how do you address them?
209. How would you log and retrieve large artifacts like embedding matrices efficiently in MLflow?
210. What are the security and access control considerations when running a shared MLflow tracking server?

---

## Computer Science Fundamentals

211. What is the difference between a compiler and an interpreter?
212. What are the phases of compilation, and what does each phase produce?
213. What is an abstract syntax tree (AST), and how is it used during compilation?
214. What is the difference between static analysis and dynamic analysis of a program?
215. What is type inference, and how does a compiler perform it?
216. What is a runtime, and what services does it typically provide to a running program?
217. What is the difference between early binding and late binding?
218. What is a namespace, and why is it important in large codebases?
219. What is the difference between a library and a framework (inversion of control)?
220. What is a binary search tree, and what property must it maintain?
221. What is the difference between a hash map and a tree map in terms of performance guarantees?
222. What is a hash collision, and how do open addressing and chaining resolve it?
223. What is amortized analysis, and give an example where it applies (e.g., dynamic array)?
224. What is a graph, and what is the difference between a directed and an undirected graph?
225. What is the difference between breadth-first traversal and depth-first traversal of a graph?
226. What is a trie, and what kind of problems is it well-suited for?
227. What is eventual consistency, and how does it differ from strong consistency?
228. What is the CAP theorem, and what are its practical implications when choosing a database?
229. What is the difference between a relational database and a document store? When would you choose each?
230. What is a B-tree, and why is it used for database indexes rather than a binary search tree?
231. What is a bloom filter, and what is it useful for in system design?
232. What is locality of reference, and how does it influence data structure and algorithm selection?
233. What is memoization, and how does it relate to dynamic programming?
234. What is the difference between a deterministic and a non-deterministic algorithm?
235. What is the difference between P and NP in computational complexity?
236. What is a finite state machine, and give a practical example of where you would model something as one?
237. What is serialization and deserialization, and what formats are commonly used?
238. What is endianness, and when does it matter in practice?
239. What is the difference between a lossy and a lossless compression algorithm?
240. What is a checksum, and how is it different from a cryptographic hash?
241. What is public-key cryptography, and how does TLS use it to establish a secure connection?
242. What is a distributed hash table (DHT), and how is it used in peer-to-peer systems?
243. What is consistent hashing, and what problem does it solve in distributed caching?
244. What is a Merkle tree, and where is it used in distributed systems?
245. What is the difference between horizontal partitioning (sharding) and vertical partitioning of a database?
246. What is the difference between a message queue and a pub/sub system?
247. What is a two-phase commit protocol, and what are its limitations?
248. What is a vector clock, and how does it help reason about event ordering in distributed systems?
249. What is the difference between synchronous replication and asynchronous replication in distributed databases?
250. What is back pressure in a data pipeline or streaming system, and how do you handle it?