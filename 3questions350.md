# Supplementary Interview Questions — 10 New Topics
### 350 Questions to Complement the Original 250

---

## 1. Python-Specific Fundamentals (35 Questions)

1. What is the GIL (Global Interpreter Lock) in CPython, and how does it affect multi-threaded Python programs?
2. If the GIL prevents true parallelism, when does using Python threads still make sense?
3. What is the difference between `multiprocessing` and `threading` in Python, and when would you choose each?
4. How does Python manage memory, and what is the role of reference counting in CPython?
5. What is a circular reference in Python, and how does the cyclic garbage collector handle it?
6. What are Python decorators, and how do you implement one from scratch?
7. How do you write a decorator that accepts arguments? Walk through the additional layer of wrapping required.
8. What is a context manager, and how does the `with` statement work under the hood?
9. How do you implement a custom context manager using both a class (`__enter__`/`__exit__`) and `contextlib.contextmanager`?
10. What is the difference between a list comprehension and a generator expression? When does the generator form matter?
11. What is a Python generator, and how does the `yield` keyword change a function's execution model?
12. What is `yield from`, and how does it simplify working with nested generators?
13. What is the difference between `__str__` and `__repr__`? Which one should you always implement first?
14. What are Python's dunder (magic) methods? Name five commonly used ones and what they enable.
15. What is `__slots__`, and when would you use it to optimize a class?
16. What is the MRO (Method Resolution Order) in Python, and how does it resolve multiple inheritance?
17. What is the difference between `@staticmethod`, `@classmethod`, and a regular instance method?
18. How do Python's `*args` and `**kwargs` work, and how do you use them when designing flexible APIs?
19. What is the difference between mutable default arguments and immutable default arguments? What classic bug does this cause?
20. What is the difference between `is` and `==` in Python? Give an example where they produce different results.
21. What are Python's built-in data structures (`list`, `tuple`, `dict`, `set`), and what are the time complexities of their common operations?
22. What is a `defaultdict`, and when would you use it instead of a plain `dict`?
23. What is a `namedtuple`, and what advantages does it have over a plain tuple or a dict?
24. What is a `dataclass`, and how does it compare to writing a class manually or using `namedtuple`?
25. How does Python's `functools.lru_cache` work, and when should you use it?
26. What is the difference between `deepcopy` and `copy` in Python's `copy` module?
27. What is a virtual environment, and why is it important to isolate Python project dependencies?
28. What is the difference between `pip` and `conda` for package management?
29. How do you structure a Python package? What is the role of `__init__.py`?
30. What is the difference between `requirements.txt` and `pyproject.toml`/`setup.cfg` for dependency management?
31. How does Python's `import` system work? What is the difference between absolute and relative imports?
32. What is monkey patching in Python, and when is it acceptable vs. dangerous?
33. What is the purpose of `__all__` in a Python module?
34. How do you profile a Python program to identify performance bottlenecks?
35. What is the difference between `asyncio`, `threading`, and `multiprocessing`? How do you choose the right concurrency model for a Python task?

---

## 2. Cloud & Infrastructure (35 Questions)

36. What is IAM (Identity and Access Management), and how do you apply the principle of least privilege in a cloud environment?
37. What is the difference between an IAM role and an IAM user in AWS? When would you use each?
38. What is an IAM policy, and how does an allow/deny evaluation work when multiple policies apply to the same resource?
39. What is object storage (e.g., S3, GCS), and how does it differ from block storage and file storage?
40. What is an S3 bucket policy vs. an S3 ACL? Which should you prefer for modern access control?
41. How does S3 versioning work, and what are its storage cost implications?
42. What is S3 lifecycle policy, and how do you use it to manage data across storage tiers?
43. What is the difference between S3 Standard, S3 Infrequent Access, and S3 Glacier? How do you choose?
44. What is a VPC (Virtual Private Cloud), and what problem does it solve in cloud networking?
45. What is the difference between a public subnet and a private subnet in a VPC?
46. What is a NAT gateway, and why is it needed for outbound traffic from a private subnet?
47. What is a security group vs. a network ACL in AWS, and how do they differ in how they evaluate traffic?
48. What is serverless computing, and what are the trade-offs of using Lambda functions vs. containerized services?
49. What is a cold start in a serverless function, and what strategies reduce its impact?
50. What is the difference between containers and VMs at the infrastructure level? When does each make sense?
51. What is Kubernetes at a conceptual level, and what problems does it solve for container orchestration?
52. What is a Kubernetes Pod, Deployment, and Service? How do they relate to each other?
53. What is Infrastructure as Code (IaC), and why is it preferable to managing infrastructure manually?
54. What is the difference between Terraform and CloudFormation? What are the trade-offs of each?
55. What is Terraform state, and why is remote state (e.g., stored in S3) important in a team environment?
56. What is idempotency in the context of IaC, and how does Terraform enforce it?
57. What is a Terraform module, and how does it promote reusability across infrastructure configurations?
58. What is a managed service vs. a self-hosted service in the cloud? When do the cost and operational trade-offs favor each?
59. What is auto-scaling, and what metrics would you use to trigger scaling events for a data processing workload?
60. What is a multi-region architecture, and what challenges does it introduce for data consistency?
61. What is a cloud storage presigned URL, and what is a practical use case for it?
62. How do you secure data at rest and data in transit in a cloud environment?
63. What is a service account, and how does it differ from a human user account in a cloud platform?
64. What is the shared responsibility model in cloud security, and what does it mean for a data engineering team?
65. What is FinOps, and how do you approach cost visibility and optimization for cloud-based data pipelines?
66. What is a spot/preemptible instance, and when is it appropriate to use one for data workloads?
67. What is a cloud cost anomaly, and what tooling or practices help you detect and respond to one?
68. What is a deployment region vs. an availability zone, and how do they factor into high-availability design?
69. What is the difference between push-based and pull-based cloud event architectures?
70. What is a cloud-native logging stack, and how do you centralize logs from distributed services?

---

## 3. API Design (35 Questions)

71. What are the core constraints of REST, and why does violating them cause problems for API consumers?
72. How do you design resource-oriented URL structures? What makes a URL scheme intuitive and consistent?
73. What is the difference between PUT and PATCH? When should you use each?
74. What HTTP status codes should a well-designed API use for success, client errors, and server errors?
75. What is idempotency in API design, and which HTTP methods are required to be idempotent?
76. How do you implement pagination for a large dataset API response? Compare offset-based vs. cursor-based pagination.
77. What is API versioning, and what are the strategies (URI versioning, header versioning, content negotiation)?
78. How do you design an API for backward compatibility? What constitutes a breaking change?
79. What is REST vs. gRPC, and when would you choose gRPC for internal service communication?
80. What is GraphQL, and what problems does it solve that REST APIs struggle with?
81. What are the trade-offs of GraphQL compared to REST in terms of caching, complexity, and over-fetching?
82. What is OpenAPI (Swagger), and why is it important for API documentation and tooling?
83. What is the difference between synchronous and asynchronous API patterns? When would you use webhooks or polling?
84. What is OAuth 2.0, and how does the authorization code flow work?
85. What is the difference between authentication and authorization in an API context?
86. What is a JWT (JSON Web Token), and what are the security risks of using them improperly?
87. What is the difference between an API key and a bearer token? When would you use each?
88. What is rate limiting, and what strategies (token bucket, leaky bucket, fixed window) can you use to implement it?
89. How do you design an API to be resilient to downstream service failures? What patterns help?
90. What is an API gateway, and what cross-cutting concerns does it handle?
91. What is CORS (Cross-Origin Resource Sharing), and why does it exist?
92. How do you handle API errors consistently across an API surface? What should an error response contain?
93. What is HATEOAS, and is it practical to implement in real-world REST APIs?
94. How do you design an idempotent POST endpoint (e.g., for payment processing)?
95. What is the difference between a public API and an internal API? How do design and governance differ?
96. What is content negotiation in HTTP, and how does it work using the `Accept` header?
97. How do you test an API contract between a producer and a consumer? What is consumer-driven contract testing?
98. What is API throttling vs. rate limiting? How are they different in intent and implementation?
99. What is the N+1 problem in GraphQL, and how does DataLoader solve it?
100. How do you design for API discoverability? What metadata should an API expose about itself?
101. What is mTLS (mutual TLS), and when would you require it for service-to-service communication?
102. How do you handle long-running operations in a REST API (e.g., async job submission and polling)?
103. What is the difference between SOAP and REST? Is there still a place for SOAP in modern systems?
104. How do you approach caching in an API? What HTTP headers control caching behavior?
105. What is an ETag, and how does it enable conditional requests to reduce bandwidth?

---

## 4. Security Fundamentals (35 Questions)

106. What is SQL injection, and how do parameterized queries prevent it?
107. What is cross-site scripting (XSS), and how do you prevent it in web applications?
108. What is cross-site request forgery (CSRF), and what mitigation techniques are effective?
109. What is the OWASP Top 10, and can you name at least five categories from it?
110. What is the principle of least privilege, and how do you apply it to a data pipeline running in the cloud?
111. What is defense in depth, and how does it differ from relying on a single security control?
112. How do you manage secrets (database passwords, API keys) in a production environment without hardcoding them?
113. What is a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault), and how does secret rotation work?
114. What is encryption at rest vs. encryption in transit? Who is each protecting against?
115. What is key management, and why should you never encrypt data with keys you also store in the same system?
116. What is a TLS certificate, and what happens when it expires? How do you automate renewal?
117. What is the difference between hashing and encryption? When should you hash instead of encrypt?
118. How do you store passwords securely? Why is MD5 or SHA-256 alone insufficient?
119. What is a rainbow table attack, and how does salting a password hash prevent it?
120. What is multi-factor authentication (MFA), and how does TOTP (time-based one-time password) work?
121. What is OAuth 2.0 scope, and how does it limit what a token is authorized to do?
122. What is token expiry and refresh token rotation? Why is short-lived access tokens a security best practice?
123. What is a supply chain attack, and how do you reduce dependency risk in a software project?
124. What is static application security testing (SAST), and where does it fit in a CI/CD pipeline?
125. What is dynamic application security testing (DAST), and how does it differ from SAST?
126. What is a CVE, and how do you manage known vulnerabilities in your third-party dependencies?
127. What is a WAF (Web Application Firewall), and what kind of attacks does it help mitigate?
128. What is network segmentation, and how does it limit the blast radius of a security breach?
129. What is the difference between a vulnerability, an exploit, and a threat?
130. What is a penetration test, and how does it differ from a bug bounty program?
131. What is audit logging, and what events should always be logged for a security-sensitive application?
132. What is the difference between authentication, authorization, and accounting (the AAA framework)?
133. What is a zero-trust security model, and how does it differ from a perimeter-based approach?
134. What is data masking vs. data tokenization, and when would you use each to protect PII?
135. How do you handle a situation where a production secret has been accidentally committed to a public Git repository?
136. What is RBAC (Role-Based Access Control) vs. ABAC (Attribute-Based Access Control)?
137. What is a DDoS attack, and what infrastructure-level defenses mitigate it?
138. How do you conduct a security review of a pull request? What are the red flags you look for?
139. What is input validation vs. output encoding, and why do you need both?
140. What is a timing attack, and what kinds of comparisons are vulnerable to it?

---

## 5. NoSQL & Specialized Databases (35 Questions)

141. What are the main categories of NoSQL databases (document, key-value, wide-column, graph), and what is each suited for?
142. When would you choose MongoDB over a relational database? What are the trade-offs?
143. What is a document in MongoDB, and how does embedding vs. referencing affect query patterns?
144. What is sharding in MongoDB, and what is a shard key? How does a poor shard key choice cause problems?
145. What is Redis, and what data structures does it support beyond simple string key-value pairs?
146. What are common Redis use cases in a data engineering or backend context?
147. What is Redis persistence, and what is the difference between RDB snapshots and AOF logging?
148. What is Redis Cluster, and how does it distribute data across nodes?
149. What is TTL (Time-To-Live) in a key-value store, and when is it an important design tool?
150. What is Apache Cassandra, and what use cases is it optimized for?
151. What is the Cassandra data model, and how does it differ fundamentally from a relational model?
152. How do you design a Cassandra partition key and clustering key? Why is query-driven design critical?
153. What is eventual consistency in Cassandra, and how do the consistency levels (ONE, QUORUM, ALL) trade off consistency vs. availability?
154. What is a wide-column store, and how does it differ from a column-oriented relational database?
155. What is DynamoDB, and what makes its pricing and performance model unique among NoSQL databases?
156. What are DynamoDB partition keys and sort keys, and how do access patterns drive table design?
157. What is a DynamoDB Global Secondary Index (GSI), and when do you need one?
158. What is the difference between a graph database and a relational database for modeling relationship-heavy data?
159. What is Neo4j, and what query language does it use? Give a real-world use case for a graph database.
160. What is the CAP theorem's practical implication when choosing between Cassandra, MongoDB, and a relational DB?
161. What is an in-memory database, and how does it differ from a traditional disk-based database in terms of durability?
162. What is InfluxDB or a time-series database, and what makes it better suited for metrics and events than a general-purpose DB?
163. What is ElasticSearch, and what types of queries is it optimized for that relational databases handle poorly?
164. What is an inverted index, and how does ElasticSearch use it for full-text search?
165. What is a vector database, and why has it become important in the context of ML and semantic search?
166. How do you choose between a managed NoSQL service and a self-hosted one? What operational factors matter?
167. What is multi-model database design, and what are the risks of using a single database for multiple access patterns?
168. What is read-your-writes consistency, and why does it matter for user-facing applications?
169. What is the difference between a hot partition and a hot key, and how do you resolve each?
170. How does data modeling in NoSQL require you to think differently than in a normalized relational schema?
171. What is a counter in Cassandra, and why are counters a special case in distributed systems?
172. What is a bloom filter, and how does Cassandra use it to avoid unnecessary disk reads?
173. What is compaction in Cassandra or LSM-tree-based stores, and why is it necessary?
174. What is an ACID-compliant NoSQL database, and what trade-offs does it make compared to eventually consistent NoSQL?
175. What is the difference between optimistic and pessimistic concurrency control in a NoSQL context?

---

## 6. Git & Version Control Workflows (35 Questions)

176. What is the difference between `git merge` and `git rebase`? When would you use each, and what are the risks of rebase on shared branches?
177. What is a fast-forward merge, and when does Git perform one automatically?
178. What is the difference between `git pull` and `git fetch`?
179. What is a detached HEAD state in Git, and how do you recover from it?
180. What is `git stash`, and when is it useful compared to creating a temporary branch?
181. What is `git cherry-pick`, and what are the risks of using it frequently?
182. What is `git bisect`, and how do you use it to find the commit that introduced a bug?
183. What is `git reflog`, and why is it useful when recovering from mistakes?
184. What is the difference between `git reset` (soft, mixed, hard) and `git revert`? When should you use each?
185. What is trunk-based development, and how does it differ from Gitflow?
186. What is Gitflow, and what are its advantages and criticisms in modern CI/CD environments?
187. What are the trade-offs between short-lived feature branches and long-lived feature branches?
188. What is a merge conflict, and what is your process for resolving one carefully?
189. What is a pull request (or merge request), and what makes a good PR description?
190. How do you keep a feature branch up to date with the main branch? What are the two approaches?
191. What is a commit squash, and when does squashing commits improve or harm the repository's history?
192. What is an interactive rebase (`git rebase -i`), and what can you do with it?
193. What is a `.gitignore` file, and what are common categories of files that should always be ignored?
194. What is a Git tag, and how does it differ from a branch? How are annotated tags different from lightweight tags?
195. How do you handle a situation where sensitive data (e.g., an API key) is accidentally committed to a repository?
196. What is a Git hook, and what are pre-commit and pre-push hooks useful for?
197. What is the difference between a fork and a branch in the context of open-source contribution workflows?
198. How does `git blame` help in a code review or debugging session? What are its limitations?
199. What is a monorepo, and what specific Git strategies (sparse checkout, shallow clone) help manage it at scale?
200. What is a shallow clone, and when would you use it in a CI/CD pipeline?
201. How do you enforce commit message standards across a team?
202. What is a submodule in Git, and what are the operational challenges of using them?
203. How do you handle a hotfix that needs to be applied to multiple release branches simultaneously?
204. What is the difference between `origin` and `upstream` remote names in a forked repository workflow?
205. What is branch protection, and what rules should you enforce on the main branch in a production repository?
206. How would you audit what changed between two releases using Git commands?
207. What is `git worktree`, and when is it more useful than switching branches?
208. What is a rebase conflict, and how does it differ from a merge conflict in how you resolve it?
209. How do you write a useful commit message? What structure would you recommend for a team standard?
210. What is conventional commits, and how does it enable automated changelog generation and semantic versioning?

---

## 7. DevOps / SRE Concepts (35 Questions)

211. What is the difference between an SLI, SLO, and SLA? Give an example of each for a data pipeline.
212. What is an error budget, and how does it help balance reliability work with feature development?
213. What are the four golden signals of monitoring, and what does each tell you about a service's health?
214. What is the difference between monitoring and observability? Why is observability more important for distributed systems?
215. What are the three pillars of observability (metrics, logs, traces), and what does each capture?
216. What is distributed tracing, and how does a trace ID propagate through a microservices call chain?
217. What is a canary deployment, and how does it reduce the risk of releasing a new version?
218. What is a blue-green deployment, and how does it differ from a canary deployment?
219. What is a rolling deployment, and what are the risks if your new and old versions are not backward compatible?
220. What is a feature flag, and how does it decouple deployment from release?
221. What is a runbook, and what makes a runbook effective during an incident?
222. What is an incident, and how do you distinguish a P1 from a P2 from a P3?
223. Walk through the phases of an incident response: detection, triage, mitigation, resolution, and post-mortem.
224. What is a blameless post-mortem, and why is psychological safety important for running one effectively?
225. What is a chaos engineering experiment, and what is it designed to reveal about a system?
226. What is the difference between MTTR (Mean Time to Restore) and MTBF (Mean Time Between Failures)?
227. What is a circuit breaker pattern, and how does it prevent cascading failures in a distributed system?
228. What is the difference between a health check endpoint and a readiness/liveness probe in Kubernetes?
229. What is the difference between horizontal pod autoscaling and vertical pod autoscaling in Kubernetes?
230. What is infrastructure drift, and how do you detect and remediate it in an IaC-managed environment?
231. What is a deployment pipeline, and what stages should a production pipeline include?
232. What is the difference between continuous integration, continuous delivery, and continuous deployment?
233. What is shift-left testing, and how does it change where quality assurance happens in a development cycle?
234. What is a DORA metric, and what are the four key metrics it tracks for engineering team performance?
235. What is toil in SRE terminology, and what is the recommended approach to reducing it?
236. How do you design an alerting strategy that minimizes alert fatigue while still catching real incidents?
237. What is the difference between a proactive and a reactive capacity planning approach?
238. What is log aggregation, and what tools are commonly used to centralize logs from distributed services?
239. What is a service mesh (e.g., Istio), and what problems does it solve for inter-service communication?
240. What is immutable infrastructure, and why is it considered a best practice in modern DevOps?
241. What is GitOps, and how does it use a Git repository as the source of truth for infrastructure state?
242. What is the difference between a deployment and a release? Why is the distinction important for risk management?
243. How do you handle database migrations as part of a zero-downtime deployment?
244. What is on-call rotation, and what practices make on-call sustainable for engineering teams?
245. What is the strangler fig pattern in the context of migrating from a monolith to microservices?

---

## 8. ML Engineering Concepts (35 Questions)

246. What is the difference between a data scientist and an ML engineer? Where does their work overlap and diverge?
247. What is the train-serve skew problem, and what causes it?
248. What is a feature store, and what problems does it solve for both training and serving?
249. What is the difference between online features and offline features in a feature store?
250. What is a feature pipeline, and how does it differ from a model training pipeline?
251. What is model serving, and what is the difference between online inference and batch inference?
252. What is the difference between a model endpoint and a batch prediction job? When would you use each?
253. What is model latency, and what techniques can you use to reduce it in a real-time serving environment?
254. What is model quantization, and how does it reduce inference cost?
255. What is model distillation, and when would you use it over quantization?
256. What is A/B testing for ML models, and how does it differ from A/B testing a UI change?
257. What is a shadow deployment for a model, and what does it tell you before you go live?
258. What is model monitoring, and what is the difference between data drift and concept drift?
259. How do you detect data drift in a production model, and what statistical tests are used?
260. What is concept drift, and how do you design a retraining strategy to respond to it?
261. What is a model card, and what information should it contain?
262. What is responsible AI, and what checks would you apply before deploying a model to production?
263. What is label leakage (target leakage), and why is it one of the most dangerous ML pipeline bugs?
264. What is cross-validation, and why is it important to ensure the validation set reflects real-world conditions?
265. What is a training-validation-test split, and what mistake do teams commonly make with test sets?
266. What is the difference between precision and recall, and in what business contexts do you optimize for each?
267. What is an ROC curve and AUC score, and what do they tell you about a classifier?
268. What is class imbalance, and what techniques (oversampling, undersampling, class weighting) address it?
269. What is a confusion matrix, and what can you derive from it beyond accuracy?
270. What is hyperparameter tuning, and what are the trade-offs between grid search, random search, and Bayesian optimization?
271. What is the difference between a batch training pipeline and an online learning system?
272. What is a champion-challenger model pattern, and how do you manage the transition between them in production?
273. How do you version a dataset for reproducibility in an ML experiment?
274. What is the role of a metadata store in an ML platform, and how does it complement a feature store?
275. What is an ML pipeline orchestrator (e.g., Kubeflow, Vertex AI Pipelines), and how does it differ from Airflow?
276. How do you evaluate a regression model vs. a classification model? What metrics apply to each?
277. What is the difference between model explainability and model interpretability?
278. What is SHAP, and how does it help explain individual model predictions?
279. What is an embedding, and how is it used in production ML systems for similarity search?
280. What is a vector index (e.g., FAISS, Pinecone), and how does approximate nearest neighbor search work?

---

## 9. Cost & Performance Engineering (35 Questions)

281. What is query cost in a cloud data warehouse (e.g., BigQuery, Snowflake), and what factors drive it?
282. How does BigQuery's on-demand pricing differ from its capacity-based (flat-rate) pricing? When does each make sense?
283. What is partition pruning in a cloud data warehouse, and how do you design tables to take advantage of it?
284. What is clustering in BigQuery or Snowflake, and how does it improve query performance and reduce cost?
285. What is the difference between a full table scan and a partition scan, and why does it matter for cost?
286. How do you analyze query execution in Snowflake using query profile and what bottlenecks does it expose?
287. What is a virtual warehouse in Snowflake, and what are the cost implications of auto-suspend and auto-resume?
288. What is result caching in BigQuery and Snowflake, and how do you design queries to take advantage of it?
289. What is a materialized view in a cloud warehouse, and when does materializing a query reduce cost vs. increase it?
290. How do you right-size a Spark cluster for a workload? What happens if you over-provision vs. under-provision?
291. What is the cost of a Spark shuffle, and what optimizations reduce shuffle volume?
292. What is spill to disk in Spark, and how do you prevent it through memory tuning?
293. What is the difference between wide and narrow transformations in Spark from a cost perspective?
294. How do you estimate the cost of an EMR or Dataproc cluster job before running it?
295. What is auto-scaling in cloud compute, and what are the cost risks of scaling policies that are too aggressive?
296. What is the cost impact of data egress in cloud architectures, and how do you design to minimize it?
297. What is columnar storage, and why does it reduce both storage costs and query costs compared to row-based storage?
298. What is compression in cloud storage and data warehouses, and what trade-offs does it introduce for compute?
299. How do you monitor cloud spending in real time, and what tools give you per-job cost visibility?
300. What is reserved capacity vs. on-demand pricing in cloud compute, and what commitment is required?
301. What is a TCO (Total Cost of Ownership) analysis, and when would you conduct one for a data platform decision?
302. How do you reduce costs in a data lake without degrading query performance?
303. What is the performance impact of small files in a distributed storage system, and how do you solve the small-file problem?
304. What is late materialization in a columnar query engine, and why does it improve performance?
305. How do you profile memory usage in a Python-based data pipeline?
306. What is vectorized execution in a query engine, and how does it improve CPU efficiency?
307. What is the difference between I/O-bound and CPU-bound workloads, and how do you optimize each?
308. How do you benchmark a data pipeline to establish a performance baseline before making changes?
309. What is network I/O cost in a distributed system, and what data locality strategies reduce it?
310. What is a hot spot in a distributed database, and what re-partitioning strategies resolve it?
311. How do you use query tagging and cost attribution to charge back cloud spending to individual teams?
312. What is the performance impact of broadcasting a large table in a join, and when does it become harmful?
313. How do you evaluate the trade-off between compute cost and storage cost when choosing between pre-aggregation and on-demand query?
314. What is connection pooling from a cost and performance perspective in a data warehouse context?
315. How do you identify and eliminate redundant data processing in a complex pipeline DAG?

---

## 10. Soft Skills & Behavioral Engineering Scenarios (35 Questions)

316. Walk me through a time you caused or contributed to a production outage. What happened, how did you respond, and what did you change afterward?
317. How do you communicate a production incident to non-technical stakeholders in real time?
318. Describe a time you had to push back on a feature request or deadline. How did you frame the conversation?
319. How do you onboard yourself to a codebase you have never seen before? Walk through your approach.
320. You join a team and discover that a critical pipeline has no tests and no documentation. How do you prioritize improving it without stopping feature work?
321. Describe a situation where you disagreed with a technical decision made by a senior engineer or architect. How did you handle it?
322. How do you decide what to do when you are blocked and your team member is unavailable?
323. Describe a time you had to explain a complex technical concept to a non-technical audience. What was your approach?
324. How do you handle receiving critical feedback on your code in a code review?
325. Describe a time you had to make a technical decision with incomplete information. How did you approach it?
326. How do you manage your own workload when multiple high-priority tasks arrive simultaneously?
327. Describe a time you mentored a junior engineer. What approach did you take, and what was the outcome?
328. How do you stay up to date with new technologies without getting overwhelmed by the pace of change?
329. Describe a situation where you identified a risk or problem that others had not noticed. What did you do?
330. How do you approach estimating the time required for a task you have never done before?
331. Describe a project that failed or significantly underdelivered. What did you learn from it?
332. How do you build trust with a new team when you join as the most experienced engineer?
333. Describe a time you had to advocate for investing in infrastructure, reliability, or tooling over new features. How did you make the case?
334. How do you handle a situation where a stakeholder is asking for something you believe is technically the wrong solution?
335. What is your approach to giving feedback to a peer whose code quality or work habits are affecting the team?
336. Describe a time when you had to learn a new technology quickly to deliver a project. What was your process?
337. How do you balance writing perfect code against shipping something that works on time?
338. How would you handle discovering that a colleague has been cutting corners on security or data quality to meet a deadline?
339. Describe a time you successfully reduced technical debt in a meaningful way. What was the impact?
340. How do you approach documentation? What do you document, and what do you leave out?
341. Describe a situation where two engineers on your team had a conflict about a technical approach. How did you help resolve it?
342. How do you ensure that knowledge is not siloed to one person on your team?
343. What do you do when a project's requirements keep changing mid-sprint?
344. Describe a time you used data or metrics to change a technical or product decision.
345. How do you approach performance reviews or goal-setting for yourself? What do you optimize for?
346. Describe a time you had to deprecate or remove a system or feature that others depended on. How did you manage the transition?
347. How do you approach working with a team in a different time zone on a shared codebase?
348. What is your framework for deciding whether to build a tool in-house or adopt an open-source or commercial solution?
349. Describe the most technically complex project you have worked on. What made it complex, and how did you navigate that complexity?
350. How do you know when you are done? What does "done" mean to you in the context of a software or data engineering task?
