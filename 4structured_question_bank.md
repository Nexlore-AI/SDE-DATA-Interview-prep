# Structured Interview Question Bank — Backend + Data + AI/ML

> **Total: ~330 questions** across 17 topics, organized by priority.

---

==============================
FILE: Programming Fundamentals
==============================

### HIGH PRIORITY
1. What is the difference between pass-by-value and pass-by-reference? How does your primary language handle it?
2. What are the four pillars of object-oriented programming, and why does each one matter?
3. What is the difference between an abstract class and an interface? When do you use each?
4. What is polymorphism? How does compile-time polymorphism differ from runtime polymorphism?
5. What is the difference between composition and inheritance? Why do experienced engineers often prefer composition?
6. What is a closure, and how does it capture variables from its enclosing scope?
7. What is the difference between mutable and immutable objects? Why does immutability matter in concurrent code?
8. What is the difference between static typing and dynamic typing? What are the practical trade-offs?
9. What is exception handling? How do you decide between throwing an exception, returning an error, and using a result type?
10. What is a lambda expression, and where would you use one over a named function?

### MEDIUM PRIORITY
11. What is type coercion, and what class of bugs does it introduce?
12. What is the difference between a shallow copy and a deep copy? When does it matter?
13. What are generics, and why are they important for writing reusable, type-safe code?
14. What is a generator, and how does it differ from a function that returns a list?
15. What is the difference between eager evaluation and lazy evaluation? When does lazy evaluation produce better performance?
16. What is method overloading vs. method overriding? Can you have both in the same class?
17. What is dependency injection, and how does it improve testability and decoupling?
18. What is serialization and deserialization? Where is it used in distributed systems?

### LOW PRIORITY
19. What is a coroutine, and how does it differ from a thread?
20. What is tail recursion, and which languages optimize for it?
21. What is operator overloading, and what are the risks of misusing it?
22. What is the difference between a struct and a class in languages that support both?
23. What are first-class functions, and how do they enable higher-order programming patterns?

---

==============================
FILE: Data Structures & Algorithms (DSA)
==============================

### HIGH PRIORITY
1. What is Big-O notation, and why is it more useful than benchmarking for comparing algorithms?
2. What is the difference between an array and a linked list at the memory level? How does this affect access, insertion, and deletion?
3. How does a hash map work internally? What happens during a hash collision, and what strategies resolve it?
4. What is a stack vs. a queue? Give a real-world use case for each.
5. What is the difference between a binary search tree and a balanced BST (e.g., AVL, Red-Black)? Why does balancing matter?
6. When would you use a heap (priority queue), and what is its time complexity for insert and extract-min?
7. What is the difference between BFS and DFS in terms of use cases and space complexity?
8. What is the difference between a tree and a graph? How does this affect traversal strategies?
9. Why is sorting important as a preprocessing step? Compare the trade-offs between merge sort, quick sort, and heap sort.
10. What is hashing, and what makes a good hash function for a hash table?

### MEDIUM PRIORITY
11. What is amortized analysis? Give an example of a data structure whose worst-case per-operation cost is misleading without it.
12. What is the difference between a trie and a hash map for string lookups? When would you prefer one?
13. What is a bloom filter, and where is it used in practice?
14. What is the difference between a directed and an undirected graph? How does this affect shortest-path algorithms?
15. When would you use a disjoint set (union-find) data structure?
16. What is the difference between time complexity and space complexity? When would you trade one for the other?
17. What is a monotonic stack, and what class of problems does it solve efficiently?
18. What is topological sorting, and where is it used in real systems?

### LOW PRIORITY
19. What is a skip list, and how does it compare to a balanced BST?
20. What is an LRU cache, and what data structures power an O(1) implementation?
21. What is the difference between a B-tree and a B+ tree? Why are they used in databases?
22. What is consistent hashing, and why is it important in distributed systems?

---

==============================
FILE: Backend Engineering
==============================

### HIGH PRIORITY
1. What is a REST API? What makes an API truly RESTful vs. just using HTTP?
2. What is the difference between synchronous and asynchronous request handling in a backend service?
3. How do you design an API for idempotency? Why is it important for payment or order endpoints?
4. What are HTTP status codes? When would you return 200, 201, 204, 400, 401, 403, 404, 409, 429, 500, 502, 503?
5. What is the difference between authentication and authorization? Describe at least two auth mechanisms.
6. What is JWT, and how does it work for stateless authentication? What are its security limitations?
7. What is rate limiting, and how would you implement it in a backend service?
8. What is the difference between a monolithic architecture and a microservices architecture? What are the trade-offs?
9. What is an API gateway, and what problems does it solve in a microservices environment?
10. What is a message queue (e.g., RabbitMQ, SQS), and when should you use asynchronous messaging instead of synchronous HTTP?
11. How would you design a retry mechanism with exponential backoff and jitter?
12. What is the circuit breaker pattern, and why is it critical in distributed systems?

### MEDIUM PRIORITY
13. What is the difference between GraphQL and REST? When would you choose one over the other?
14. What is gRPC, and what advantages does it have over REST for service-to-service communication?
15. What is the SAGA pattern, and how does it handle distributed transactions across microservices?
16. What is CQRS (Command Query Responsibility Segregation), and when would you apply it?
17. What is event sourcing, and how does it differ from traditional CRUD persistence?
18. How do you handle versioning of APIs without breaking existing clients?
19. How would you design a webhook system? What reliability guarantees would you provide?
20. What is a sidecar pattern, and how is it used in service meshes like Istio?
21. What is structured logging, and why is it preferable to unstructured log lines?
22. What is distributed tracing, and how do tools like Jaeger or OpenTelemetry help debug latency issues?

### LOW PRIORITY
23. What is the bulkhead pattern, and how does it isolate failures in a microservices system?
24. What is the outbox pattern, and how does it ensure reliable event publishing alongside database writes?
25. How do you handle request deduplication in an event-driven backend?
26. What is a service registry, and how does service discovery work in a container-orchestrated environment?
27. What is blue-green deployment vs. canary deployment? When would you use each?

---

==============================
FILE: System Design (HLD)
==============================

### HIGH PRIORITY
1. How would you design a URL shortener? What are the key decisions around ID generation, storage, and redirection?
2. How would you design a rate limiter that works across multiple servers?
3. How would you design a notification system (push, email, SMS) that handles millions of users?
4. What is horizontal scaling vs. vertical scaling? When do you apply each?
5. What is the CAP theorem? How does it influence your choice of database in a distributed system?
6. What is eventual consistency, and how does it differ from strong consistency? Give an example of where each is appropriate.
7. What is database sharding, and what are the strategies for choosing a shard key?
8. What is caching, and where do you place caches in a system (client, CDN, application, database)?
9. What are cache invalidation strategies (TTL, write-through, write-behind, cache-aside)?
10. What is a CDN, and how does it improve system performance and availability?
11. What is a load balancer? What are the differences between L4 and L7 load balancing?
12. How would you design a chat/messaging system that supports real-time and offline delivery?

### MEDIUM PRIORITY
13. How would you design a distributed job scheduler that handles millions of scheduled tasks?
14. How would you design a search autocomplete/typeahead system?
15. What is consistent hashing, and how does it help when adding/removing nodes in a distributed cache?
16. What is a write-ahead log (WAL), and why is it fundamental to database durability?
17. How would you design an analytics event collection pipeline that ingests billions of events daily?
18. What is the difference between a push model and a pull model in news feed/timeline design?
19. How would you design a file storage service (e.g., Google Drive, Dropbox)?
20. What is leader election, and when is it necessary in a distributed system?
21. How would you handle data replication across geographically distributed regions?

### LOW PRIORITY
22. How would you design a distributed unique ID generator (e.g., Snowflake IDs)?
23. What is the Raft or Paxos consensus algorithm at a high level? Why is consensus hard?
24. How would you design a real-time leaderboard for a gaming platform?
25. What is a conflict-free replicated data type (CRDT), and where would you use one?
26. How would you design a multi-tenant SaaS platform where tenants have different data isolation requirements?

---

==============================
FILE: Low-Level Design (LLD)
==============================

### HIGH PRIORITY
1. What are the SOLID principles? Explain each with a concrete example.
2. What is the Strategy pattern, and when would you use it instead of a chain of if-else?
3. What is the Factory pattern, and how does it differ from the Abstract Factory pattern?
4. What is the Observer pattern? How is it used in event-driven systems?
5. What is the Singleton pattern? What are its drawbacks, and why do some consider it an anti-pattern?
6. How would you design a parking lot system? What classes and relationships would you define?
7. How would you design an elevator system? What states and transitions does it need?

### MEDIUM PRIORITY
8. What is the Decorator pattern, and how does it differ from inheritance for extending behavior?
9. What is the Builder pattern, and when is it preferable to a constructor with many parameters?
10. What is the Adapter pattern? Give a scenario where you would use it.
11. What is the Command pattern, and how does it enable undo functionality?
12. How would you design a library management system? What entities and methods are needed?
13. How would you design a ride-sharing fare calculator that supports different pricing strategies?
14. What is the Template Method pattern, and when does it enforce a skeleton algorithm with customizable steps?

### LOW PRIORITY
15. What is the Flyweight pattern, and where does it save memory?
16. What is the Chain of Responsibility pattern, and how is it used in middleware pipelines?
17. What is the Proxy pattern, and how does it differ from the Decorator pattern?
18. How would you design a vending machine using state-based transitions?

---

==============================
FILE: Databases (DBMS)
==============================

### HIGH PRIORITY
1. What is the difference between a relational database and a NoSQL database? When do you choose one over the other?
2. What are the ACID properties? Why are they essential for transactional systems?
3. What is database normalization? Explain 1NF, 2NF, 3NF, and BCNF with examples.
4. When is denormalization the right trade-off? What problems does it introduce?
5. What is an index? How does a B+ tree index work under the hood?
6. What is the difference between a clustered index and a non-clustered index?
7. What is a transaction, and what happens when two transactions conflict?
8. What are the SQL isolation levels? What anomalies (dirty read, non-repeatable read, phantom read) does each prevent?
9. What is the difference between optimistic locking and pessimistic locking? When do you use each?
10. What is sharding? How do you choose a shard key, and what problems arise from a bad choice?

### MEDIUM PRIORITY
11. What is a write-ahead log (WAL), and how does it ensure durability and crash recovery?
12. What is MVCC (Multi-Version Concurrency Control), and how does it avoid locking for reads?
13. What is the difference between a primary key, a unique key, and a foreign key?
14. What is referential integrity, and what happens when you violate it?
15. What is database replication? What is the difference between synchronous and asynchronous replication?
16. What are the trade-offs between a document database (MongoDB) and a relational database (PostgreSQL)?
17. What is a connection pool, and why does it matter for database performance?
18. What is the difference between OLTP and OLAP systems? How does this affect schema design?
19. What is a materialized view, and when would you use one?

### LOW PRIORITY
20. What is a columnar database, and why is it efficient for analytical queries?
21. What is the difference between eventual consistency and strong consistency in a distributed database?
22. What is a time-series database (e.g., InfluxDB, TimescaleDB)? When would you use one?
23. What is a graph database, and what kinds of queries is it optimized for?
24. What is the CAP theorem in practice? How do databases like Cassandra and PostgreSQL make different trade-offs?

---

==============================
FILE: SQL (Practical + Optimization)
==============================

### HIGH PRIORITY
1. What is the difference between INNER JOIN, LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN?
2. What is the difference between WHERE and HAVING? When must you use HAVING?
3. What are window functions? Explain ROW_NUMBER(), RANK(), DENSE_RANK(), and LAG()/LEAD() with examples.
4. What is a CTE (Common Table Expression), and when is it better than a subquery?
5. What is the difference between UNION and UNION ALL?
6. How do you use EXPLAIN / EXPLAIN ANALYZE to diagnose a slow query?
7. What is a composite index, and how does column order in the index affect query performance?
8. What is the N+1 query problem, and how do you fix it?
9. How do you find and remove duplicate rows in a table?
10. How do you write an efficient pagination query for large datasets?

### MEDIUM PRIORITY
11. What is a correlated subquery, and how does it differ from a non-correlated subquery in execution?
12. What is a recursive CTE? Give a practical use case (e.g., org hierarchy, tree traversal).
13. How do you calculate running totals or cumulative sums using window functions?
14. What is index selectivity, and why does a low-selectivity index sometimes hurt performance?
15. Under what circumstances can an index make a query slower?
16. What is a covering index, and how does it eliminate table lookups?
17. How do NULL values behave in comparisons, aggregations, and joins? What pitfalls should you avoid?
18. What is the difference between EXISTS and IN? When does each perform better?

### LOW PRIORITY
19. How would you pivot rows into columns without using a built-in PIVOT function?
20. What is a self-join? Give a scenario where it is necessary.
21. What is the difference between a natural join and a join with an explicit ON clause?
22. What is a trigger, and when should you avoid using one?
23. What is the difference between a stored procedure and a function in SQL?

---

==============================
FILE: Operating Systems
==============================

### HIGH PRIORITY
1. What is the difference between a process and a thread? When would you use one over the other?
2. What is a context switch, and why does it have a performance cost?
3. What is a deadlock? What are the four necessary conditions, and how do you prevent them?
4. What is a race condition, and how do you prevent it?
5. What is the difference between a mutex and a semaphore?
6. How does virtual memory work? What roles do the page table and TLB play?
7. What is the difference between user space and kernel space?
8. What is a system call, and how does it differ from a regular function call?

### MEDIUM PRIORITY
9. What is the difference between preemptive and cooperative scheduling?
10. What is a page fault, and what happens when the OS handles one?
11. What is thrashing, and what causes it?
12. What is the difference between a hard link and a symbolic link?
13. What is copy-on-write, and where is it used in operating systems?
14. How does inter-process communication (IPC) work? Compare pipes, shared memory, and message queues.
15. What is a file descriptor, and how does the OS manage open files?

### LOW PRIORITY
16. What is the difference between a monolithic kernel and a microkernel?
17. How does the Linux OOM killer decide which process to terminate?
18. What is a spin lock, and when is it preferable to a blocking mutex?
19. What is NUMA architecture, and how does it affect application performance?
20. How does epoll differ from select/poll for handling concurrent connections?

---

==============================
FILE: Computer Networks
==============================

### HIGH PRIORITY
1. What happens step by step when you type a URL into a browser and press Enter?
2. What is the difference between TCP and UDP? Give a real-world use case for each.
3. What is the TCP three-way handshake, and what does each step accomplish?
4. What is DNS resolution, and what are the different types of DNS records (A, CNAME, MX, etc.)?
5. What is HTTP vs. HTTPS? What role does TLS play?
6. What is the difference between HTTP/1.1, HTTP/2, and HTTP/3?
7. What is a load balancer, and what algorithms can it use to distribute traffic?

### MEDIUM PRIORITY
8. What is the OSI model, and how does it help you debug network issues layer by layer?
9. What is the difference between latency, bandwidth, and throughput?
10. What is a reverse proxy, and how does it differ from a forward proxy?
11. What is a CDN, and how does it reduce latency for geographically distributed users?
12. What is the difference between a WebSocket and HTTP long polling? When would you use each?
13. What is NAT (Network Address Translation), and why is it used?
14. What is the TIME_WAIT state in TCP, and why does it exist?

### LOW PRIORITY
15. How does ARP work, and what role does it play in local network communication?
16. What is BGP, and why is it important for internet routing?
17. What is the difference between a VLAN and a subnet?
18. What is MTU, and what happens when a packet exceeds it?

---

==============================
FILE: Data Engineering
==============================

### HIGH PRIORITY
1. What is the difference between ETL and ELT? When would you choose each approach?
2. What is the difference between batch processing and stream processing? Give a real scenario for each.
3. What is a data warehouse, and how does it differ from a data lake?
4. What is a data lakehouse, and what problem does it solve?
5. What is Apache Spark, and when would you choose it over a SQL engine?
6. What is a shuffle in Spark, and why is it one of the most expensive operations?
7. What is data skew, and how do you diagnose and handle it in a distributed job?
8. What is Apache Kafka, and how does it differ from a traditional message queue?
9. What is a Kafka topic, partition, and consumer group? How do they relate to throughput and ordering?
10. What is Apache Airflow, and how does it orchestrate data pipelines better than cron?
11. What is idempotency in a data pipeline, and how do you design tasks to be idempotent?
12. What is the difference between Parquet, Avro, and ORC? When do you choose each format?

### MEDIUM PRIORITY
13. What is the medallion architecture (bronze, silver, gold)? What purpose does each layer serve?
14. What is schema evolution, and how do you handle it without breaking downstream consumers?
15. What is change data capture (CDC), and what tools or approaches implement it?
16. What is a slowly changing dimension (SCD)? Explain SCD Type 1, 2, and 3.
17. What is data lineage, and why is it important for governance and debugging?
18. What is a star schema vs. a snowflake schema? When do you use each?
19. What is backfilling in a data pipeline, and what challenges does it introduce?
20. What is exactly-once semantics in stream processing, and why is it hard to achieve?
21. What is a watermark in stream processing? How does it handle late-arriving events?
22. How do you perform data quality checks in a production pipeline? What frameworks exist?
23. What is a data contract, and how does it prevent breaking changes between producers and consumers?

### LOW PRIORITY
24. What is the difference between a Lambda architecture and a Kappa architecture?
25. What is Delta Lake, and what guarantees does it provide over plain Parquet?
26. What is a surrogate key, and why is it preferred over a natural key in data warehousing?
27. What is data cataloging, and what role does it play in data governance?
28. How do you secure PII data in a data pipeline? What techniques do you use (masking, tokenization, encryption)?
29. What are the Spark deployment modes (client, cluster), and when would you choose each?

---

==============================
FILE: Machine Learning Fundamentals
==============================

### HIGH PRIORITY
1. What is the bias-variance trade-off, and how does it relate to underfitting and overfitting?
2. What is overfitting, and what techniques do you use to prevent it?
3. What is the difference between supervised, unsupervised, and semi-supervised learning?
4. What is cross-validation, and why is it better than a single train-test split?
5. What is regularization? Explain the difference between L1 (Lasso) and L2 (Ridge) regularization.
6. What evaluation metrics would you use for a classification problem? When is accuracy misleading?
7. What is the confusion matrix, and how do you derive precision, recall, F1-score from it?
8. What is the ROC curve, and what does AUC represent?
9. What is feature engineering? Give examples of how domain knowledge improves model performance.
10. How do you handle missing data in a dataset? What are the trade-offs of each approach?

### MEDIUM PRIORITY
11. What is the difference between bagging and boosting? Give an example algorithm for each.
12. What is the curse of dimensionality, and how does it affect model performance?
13. How do you handle class imbalance in a classification task?
14. What is feature scaling, and when is it necessary? What is the difference between normalization and standardization?
15. What is multicollinearity, and how does it affect linear regression?
16. What is the difference between generative and discriminative models?
17. What is the purpose of a validation set vs. a test set?
18. How do you select features? Describe filter, wrapper, and embedded methods.

### LOW PRIORITY
19. What is the difference between parametric and non-parametric models?
20. What is a learning curve, and how do you use it to diagnose model problems?
21. What is Bayesian inference, and how does it differ from frequentist statistics in ML?
22. What is the VC dimension, and what does it tell you about model complexity?

---

==============================
FILE: Advanced ML Algorithms
==============================

### HIGH PRIORITY
1. How does a decision tree work? What are its splitting criteria (Gini impurity, information gain)?
2. What is a Random Forest, and why does it perform better than a single decision tree?
3. What is Gradient Boosting? How do XGBoost, LightGBM, and CatBoost differ?
4. How does Logistic Regression work, and why is it called "regression" when it's used for classification?
5. What is the kernel trick in SVM? When would you use an SVM over a simpler model?
6. What is K-Means clustering? What are its limitations and assumptions?
7. What is Principal Component Analysis (PCA), and when would you use it?

### MEDIUM PRIORITY
8. What is DBSCAN, and how does it differ from K-Means for clustering?
9. What is a recommendation system? Explain collaborative filtering vs. content-based filtering.
10. What is the difference between hard clustering and soft clustering? Give an example of each.
11. How does Naive Bayes work, and what is the "naive" assumption? When is it still useful?
12. What is the difference between online learning and batch learning?
13. What is an ensemble method? Why does combining weak learners often produce a strong learner?
14. What is the difference between model interpretability and model accuracy? When do you prioritize one?

### LOW PRIORITY
15. What is a Gaussian Mixture Model (GMM), and how does it relate to K-Means?
16. What is isolation forest, and how does it detect anomalies?
17. What is a Hidden Markov Model, and where is it used in practice?
18. What is the EM (Expectation-Maximization) algorithm at a high level?

---

==============================
FILE: ML Systems Design
==============================

### HIGH PRIORITY
1. How would you design an end-to-end ML system from data collection to model serving?
2. What is the difference between online (real-time) and batch (offline) model inference? When do you use each?
3. How do you monitor a model in production? What metrics would you track?
4. What is model drift (data drift and concept drift)? How do you detect and respond to it?
5. What is a feature store, and why is it important for ML systems at scale?
6. How would you design an A/B testing framework for evaluating ML model changes?
7. How do you version data, models, and pipelines in an ML system?

### MEDIUM PRIORITY
8. What is the difference between shadow deployment, canary deployment, and blue-green deployment for ML models?
9. How would you design a real-time prediction service with latency constraints?
10. What is the feedback loop problem in ML systems, and how can it lead to bias amplification?
11. How do you handle training-serving skew (discrepancies between training features and serving features)?
12. What is model compression? Describe techniques like quantization, pruning, and distillation.
13. How would you design a recommendation system that serves millions of users with personalized results?
14. What is the role of an orchestrator (e.g., Kubeflow, Airflow) in an ML pipeline?

### LOW PRIORITY
15. How would you design an ML system that needs to comply with GDPR (right to be forgotten, explainability)?
16. What is a champion-challenger pattern in model deployment?
17. How do you handle cold-start problems in a recommendation or personalization system?
18. What is federated learning, and when is it the right approach?

---

==============================
FILE: MLflow & MLOps Pipelines
==============================

### HIGH PRIORITY
1. What is MLflow, and what are its four core components?
2. How does MLflow Tracking work? What information does it log for each run?
3. What is the MLflow Model Registry, and what problem does it solve?
4. How do you log parameters, metrics, and artifacts in an MLflow run?
5. How do you compare multiple experiment runs in MLflow to select the best model?
6. What are the model lifecycle stages (Staging, Production, Archived), and how do you transition between them?
7. How do you integrate MLflow into a CI/CD pipeline for automated training and deployment?

### MEDIUM PRIORITY
8. What is an MLflow Project, and how does it ensure reproducibility?
9. What is the MLflow Model Signature, and why is it important for input validation?
10. How do you serve an MLflow model as a REST API endpoint?
11. How do you automate model promotion from Staging to Production using MLflow APIs?
12. What is the difference between a logged model and a registered model in MLflow?
13. How does MLflow handle nested runs? When would you use them?
14. How do you log and version datasets alongside models in MLflow?

### LOW PRIORITY
15. How do you set up MLflow in a team environment with a shared tracking server and remote artifact store?
16. What are the security considerations when exposing an MLflow tracking server?
17. How does MLflow integrate with Databricks and cloud ML platforms?
18. What are the limitations of MLflow, and when might you choose an alternative (e.g., Weights & Biases, Neptune)?
19. How do you manage artifact storage costs at scale in MLflow?

---

==============================
FILE: Deep Learning
==============================

### HIGH PRIORITY
1. What is a neural network? Explain forward propagation and backpropagation at a high level.
2. What is the vanishing gradient problem, and what techniques mitigate it?
3. What is the difference between a CNN and an RNN? What types of data is each designed for?
4. What is a convolutional layer, and what is the role of filters, stride, and padding?
5. What are activation functions (ReLU, sigmoid, tanh, softmax)? Why is ReLU preferred in hidden layers?
6. What is dropout, and how does it act as a regularizer during training?
7. What is batch normalization, and why does it help training stability and speed?

### MEDIUM PRIORITY
8. What is the difference between SGD, Adam, and RMSProp optimizers? When would you choose each?
9. What is transfer learning, and when does it significantly outperform training from scratch?
10. What is an LSTM, and what problem does it solve over a vanilla RNN?
11. What is a learning rate schedule, and why does it matter for convergence?
12. What is the difference between a generative model and a discriminative model in deep learning?
13. What is data augmentation, and how does it improve generalization?
14. What is the role of a loss function? Compare cross-entropy loss and mean squared error.

### LOW PRIORITY
15. What is a GAN (Generative Adversarial Network), and how do the generator and discriminator interact?
16. What is a variational autoencoder (VAE), and how does it differ from a standard autoencoder?
17. What is mixed-precision training, and how does it speed up deep learning workloads?
18. What is gradient clipping, and when is it necessary?
19. What is the difference between model parallelism and data parallelism for distributed training?

---

==============================
FILE: Transformers & LLMs
==============================

### HIGH PRIORITY
1. What is the Transformer architecture? Explain the key components (self-attention, positional encoding, feed-forward layers).
2. What is the self-attention mechanism, and why is it more powerful than recurrence for sequence modeling?
3. What is the difference between an encoder-only (BERT), decoder-only (GPT), and encoder-decoder (T5) architecture?
4. What are embeddings in the context of LLMs? How do word embeddings capture semantic meaning?
5. What is fine-tuning a pre-trained LLM, and when would you fine-tune vs. use prompting alone?
6. What is RAG (Retrieval-Augmented Generation), and how does it reduce hallucinations?
7. What is prompt engineering? What techniques (few-shot, chain-of-thought, system prompts) improve LLM output?
8. What are hallucinations in LLMs, and what strategies mitigate them?
9. What is the difference between zero-shot, few-shot, and fine-tuned model performance?

### MEDIUM PRIORITY
10. What is LoRA (Low-Rank Adaptation), and why is it popular for efficient fine-tuning?
11. What is RLHF (Reinforcement Learning from Human Feedback), and how does it align LLMs with human preferences?
12. What is a vector database, and how is it used in RAG pipelines?
13. What is tokenization, and how do subword tokenizers (BPE, WordPiece) handle out-of-vocabulary words?
14. What is the context window of an LLM, and what challenges arise when inputs exceed it?
15. What is the difference between causal (autoregressive) attention and bidirectional attention?
16. How do you evaluate the quality of an LLM's output? What metrics or frameworks exist?
17. What is knowledge distillation, and how do you create a smaller model from a larger one?

### LOW PRIORITY
18. What is multi-head attention, and why does the model use multiple attention heads instead of one?
19. What are positional encodings, and what happens if you remove them?
20. What is the difference between pre-training and instruction tuning?
21. What is quantization (e.g., 4-bit, 8-bit), and how does it enable running LLMs on smaller hardware?
22. What is a mixture-of-experts (MoE) architecture, and how does it achieve efficient scaling?
23. What are the ethical and safety concerns of deploying LLMs in production? How do you mitigate them?

---

==============================
FILE: Web Development (Backend-Focused)
==============================

### HIGH PRIORITY
1. What is the difference between a cookie, a session, and a token? When do you use each?
2. What is CORS (Cross-Origin Resource Sharing), and why does the browser enforce it?
3. What is the difference between GET, POST, PUT, PATCH, and DELETE HTTP methods?
4. What is CSRF (Cross-Site Request Forgery), and how do you prevent it?
5. What is XSS (Cross-Site Scripting)? What are the different types, and how do you defend against them?
6. What is HTTPS, and how does TLS secure data in transit?
7. What is OAuth 2.0, and how does the authorization code flow work?

### MEDIUM PRIORITY
8. What is the difference between server-side rendering (SSR) and client-side rendering (CSR)?
9. What are HTTP headers, and which ones are critical for security (e.g., Content-Security-Policy, Strict-Transport-Security)?
10. What is caching in HTTP? Explain Cache-Control, ETag, and Last-Modified headers.
11. What is the difference between a forward proxy and a reverse proxy in a web infrastructure?
12. What is REST vs. RPC? When is each paradigm appropriate?
13. What is content negotiation in HTTP, and how does the Accept header work?

### LOW PRIORITY
14. What is HTTP/2 server push, and why was it deprecated in some browsers?
15. What is a Content Delivery Network (CDN) edge function (e.g., Cloudflare Workers), and when would you use it?
16. What is the SameSite cookie attribute, and how does it protect against CSRF?
