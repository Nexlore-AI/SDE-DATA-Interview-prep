# Cloud Services — AWS vs GCP vs Azure — Interview Q&A

> This file maps cloud services across providers with analogies. Essential for **Cloud Engineer**, **AI Infra Engineer**, and **System Design (HLD)** interviews.

---

## ☁️ AWS vs GCP vs Azure — Service Mapping

### Compute

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Virtual Machines | EC2 | Compute Engine | Virtual Machines | "The server" — your basic compute unit. When you say "we need a server" in HLD, this is it. |
| Serverless Functions | Lambda | Cloud Functions | Azure Functions | "Event-driven microservice" — runs code on triggers without managing servers. Use in HLD for lightweight async tasks (image resize, webhook processing). |
| Containers (managed) | ECS / Fargate | Cloud Run | Container Apps | "Dockerized microservice deployment" — in HLD, when you containerize a service but don't want full K8s complexity. |
| Kubernetes | EKS | GKE | AKS | "Container orchestration layer" — use in HLD when you need auto-scaling, self-healing, multi-service deployments. GKE is considered the gold standard (Google created K8s). |
| Auto Scaling | Auto Scaling Groups | Managed Instance Groups | VM Scale Sets | "Elastic compute" — in HLD, this is how you handle traffic spikes. "We'll put servers behind an auto-scaling group." |

### Storage

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Object Storage | S3 | Cloud Storage (GCS) | Blob Storage | "The file store" — images, videos, backups, static assets. In every HLD: "Store media files in S3/GCS." Virtually unlimited, cheap, durable (11 nines). |
| Block Storage | EBS | Persistent Disk | Managed Disks | "The hard drive attached to your VM" — fast, low latency, but tied to one machine. |
| File Storage (NFS) | EFS | Filestore | Azure Files | "Shared file system" — when multiple servers need to read/write the same files. |
| Archive Storage | S3 Glacier | Archive Storage | Archive Storage | "Cold storage" — rarely accessed data, very cheap, slow retrieval. Use for compliance/backup. |

### Database

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Relational DB | RDS (MySQL, Postgres) | Cloud SQL | Azure SQL | "The primary database" — your OLTP store. In HLD: "User data in RDS/Cloud SQL with read replicas." |
| Global Relational | Aurora Global | Spanner | Cosmos DB (relational) | "Globally distributed SQL" — when you need consistency + global reach. Spanner is unique: external consistency across regions. |
| NoSQL (Document) | DynamoDB | Firestore / Datastore | Cosmos DB | "The NoSQL store" — key-value or document. In HLD: "Session data in DynamoDB" or "product catalog in Firestore." DynamoDB = single-digit ms latency at any scale. |
| NoSQL (Wide Column) | DynamoDB / Keyspaces | Bigtable | Cosmos DB (Cassandra API) | "Time-series / high-throughput writes" — Bigtable powers Google Search, Maps, Gmail internals. |
| In-Memory Cache | ElastiCache (Redis/Memcached) | Memorystore | Azure Cache for Redis | "The cache layer" — in every HLD: "Put a Redis cache in front of the DB to reduce read latency." |
| Search | OpenSearch (Elasticsearch) | — | Cognitive Search | "Full-text search engine" — in HLD: "Search queries go through Elasticsearch." |

### Networking

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| CDN | CloudFront | Cloud CDN | Azure CDN / Front Door | "Edge caching" — in HLD: "Static assets served from CDN for low latency globally." |
| Load Balancer | ALB / NLB / ELB | Cloud Load Balancing | Azure Load Balancer | "Traffic distributor" — sits in front of your servers. ALB = HTTP-aware (Layer 7), NLB = TCP (Layer 4). |
| DNS | Route 53 | Cloud DNS | Azure DNS | "Name resolution" — maps domain names to IPs. Route 53 supports health checks + failover routing. |
| VPC / Networking | VPC | VPC | VNet | "Your private network in the cloud" — isolates your services. Subnets, security groups, route tables. |
| API Gateway | API Gateway | Apigee / API Gateway | API Management | "The front door for your APIs" — rate limiting, auth, throttling. In HLD: "All client requests hit the API Gateway first." |

### Messaging & Streaming

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Message Queue | SQS | — | Azure Queue Storage | "Async task queue" — decouple producers from consumers. In HLD: "Order placed → message in SQS → worker processes it." |
| Pub/Sub Messaging | SNS | Pub/Sub | Service Bus | "Fan-out notifications" — one event triggers multiple subscribers. SNS + SQS combo = fan-out pattern in HLD. |
| Event Streaming | Kinesis | Pub/Sub (streaming) | Event Hubs | "Real-time data stream" — like Kafka. In HLD: "Clickstream data flows through Kinesis → processed in real-time." |
| Managed Kafka | MSK | — | Event Hubs (Kafka) | "Kafka-as-a-service" — when you explicitly need Kafka API compatibility. |

### Data & Analytics

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Data Warehouse | Redshift | BigQuery | Synapse Analytics | "The analytics database" — OLAP, columnar storage, SQL on massive data. BigQuery is serverless and auto-scales. |
| ETL / Data Pipeline | Glue | Dataflow | Data Factory | "The data mover" — extract, transform, load. In HLD: "Raw data → Glue ETL → data warehouse." |
| Workflow Orchestration | Step Functions | Cloud Composer (Airflow) | Logic Apps | "Task orchestrator" — chain multiple steps with retries, branching. In HLD: "Multi-step pipeline orchestrated by Step Functions." |
| Data Lake | S3 + Lake Formation | GCS + BigLake | Data Lake Storage | "Raw data dump" — store everything, process later. Schema-on-read. |

### AI/ML

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| ML Platform | SageMaker | Vertex AI | Azure ML | "End-to-end ML" — training, tuning, deploying models. |
| Pre-built AI APIs | Rekognition, Comprehend | Vision AI, Natural Language | Cognitive Services | "AI-as-an-API" — image recognition, NLP, speech without building models. |
| GPU Instances | P4/P5 instances | A2/A3 (NVIDIA) | NCv4 (NVIDIA) | "Training hardware" — for deep learning. GCP has TPU access (custom Google hardware). |

### Security & Identity

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Identity & Access | IAM | IAM | Azure AD + IAM | "Who can do what" — roles, policies, permissions. In HLD: "Service-to-service auth via IAM roles." |
| Secrets Management | Secrets Manager | Secret Manager | Key Vault | "Store API keys, DB passwords securely" — never hardcode secrets. |
| DDoS Protection | Shield | Cloud Armor | DDoS Protection | "Attack mitigation" — in HLD: "CloudFront + Shield protects edge layer." |
| SSL/TLS Certificates | ACM | Managed SSL | App Service Certificates | "HTTPS for free" — auto-renewing certs for your domains. |

### Monitoring & Observability

| Purpose | AWS | GCP | Azure | HLD Analogy |
|---------|-----|-----|-------|-------------|
| Monitoring | CloudWatch | Cloud Monitoring | Azure Monitor | "System health dashboard" — metrics, alarms, dashboards. |
| Logging | CloudWatch Logs | Cloud Logging | Log Analytics | "Centralized logs" — in HLD: "All services log to CloudWatch, set alarms on error rates." |
| Tracing | X-Ray | Cloud Trace | Application Insights | "Request path tracking" — trace a request across microservices to find bottlenecks. |

---

## 🏗️ Cloud Concepts for HLD Interviews

### 1. How would you explain a 3-tier architecture using cloud services?

```
Client → CloudFront (CDN) → ALB (Load Balancer) → EC2/ECS (App Servers) → RDS (Database)
                                                        ↓
                                                   ElastiCache (Redis)
                                                        ↓
                                                      S3 (Media)
```

**GCP equivalent:**
```
Client → Cloud CDN → Cloud Load Balancing → GKE/Cloud Run → Cloud SQL
                                                  ↓
                                              Memorystore
                                                  ↓
                                                 GCS
```

### 2. How do you design for high availability?

- **Multi-AZ deployment:** RDS Multi-AZ, EC2 across availability zones
- **Auto-scaling:** Scale out behind a load balancer
- **Health checks:** ALB health checks remove unhealthy instances
- **Database:** Read replicas for read scaling, Multi-AZ for failover
- **Global:** Route 53 latency-based routing across regions

### 3. What is the difference between horizontal and vertical scaling in cloud context?

- **Vertical (scale up):** Bigger instance (t3.micro → m5.2xlarge). Has limits. Downtime required.
- **Horizontal (scale out):** More instances behind a load balancer. Preferred in cloud — theoretically unlimited. Requires stateless design.

**In HLD:** Always design for horizontal scaling. "Stateless app servers behind an ALB with auto-scaling."

### 4. When would you choose serverless vs containers vs VMs?

| Scenario | Choose | Why |
|----------|--------|-----|
| Simple API, variable traffic | Lambda / Cloud Functions | No server management, pay per invocation |
| Microservices, steady traffic | ECS/Fargate / Cloud Run | Consistent performance, easier networking |
| Complex microservices architecture | EKS / GKE | Full orchestration, service mesh, canary deploys |
| Legacy apps, full OS control | EC2 / Compute Engine | Maximum flexibility |

### 5. Explain the CAP theorem using cloud database choices.

- **CP (Consistency + Partition Tolerance):** Spanner, DynamoDB (strong consistency mode) — banking systems
- **AP (Availability + Partition Tolerance):** DynamoDB (eventual consistency), Cassandra — social media feeds
- **CA (Consistency + Availability):** Single-node RDS (no partition tolerance) — small apps

**In HLD:** "For a payment system, we choose CP (Spanner/strong consistent DynamoDB). For a news feed, AP (eventual consistency) is fine."

---

## 🔑 Key AWS Services — Deep Dive (Most Asked)

### S3 (Simple Storage Service)
- **Storage classes:** Standard, IA (Infrequent Access), Glacier, Deep Archive
- **Features:** Versioning, lifecycle policies, event notifications, static website hosting
- **HLD use:** "Store user uploads in S3, trigger Lambda on upload for processing"
- **Durability:** 99.999999999% (11 nines)

### EC2 (Elastic Compute Cloud)
- **Instance types:** General (t3, m5), Compute (c5), Memory (r5), GPU (p4), Storage (i3)
- **Pricing:** On-demand, Reserved (1-3yr commitment, 40-60% off), Spot (up to 90% off, can be interrupted)
- **HLD use:** "Application servers on c5.xlarge behind an ALB"

### DynamoDB
- **Key concepts:** Partition key, sort key, GSI (Global Secondary Index), LSI (Local Secondary Index)
- **Modes:** On-demand (pay per request) vs Provisioned (set RCU/WCU)
- **DAX:** In-memory cache for DynamoDB (microsecond reads)
- **HLD use:** "Session store, user profiles, URL shortener mappings"

### Lambda
- **Limits:** 15 min timeout, 10 GB memory, 250 MB deployment package
- **Triggers:** S3, API Gateway, SQS, DynamoDB Streams, CloudWatch Events
- **Cold start:** First invocation is slower (especially Java/C#). Mitigate with provisioned concurrency.
- **HLD use:** "Image thumbnail generation on S3 upload, webhook processing"

### SQS + SNS
- **SQS:** Queue (point-to-point), at-least-once delivery, 14-day retention
  - Standard: unlimited throughput, possible duplicates
  - FIFO: exactly-once, ordered, 300 msg/sec
- **SNS:** Topic (pub/sub), push-based, fan-out to multiple subscribers
- **Pattern:** SNS → multiple SQS queues (fan-out)
- **HLD use:** "Order service → SNS → [SQS for email, SQS for inventory, SQS for analytics]"

---

## 🔑 Key GCP Services — Deep Dive

### BigQuery
- **Serverless data warehouse** — no infrastructure to manage
- **Separation of storage and compute** — pay for queries and storage independently
- **Nested/repeated fields** — supports semi-structured data
- **HLD use:** "All analytics queries run against BigQuery. Cost-efficient for ad-hoc analysis."
- **Killer feature:** Analyze petabytes in seconds with standard SQL

### Spanner
- **Globally distributed relational database** with strong consistency
- **TrueTime API** — uses atomic clocks + GPS for global consistency
- **HLD use:** "For a global banking system that needs ACID transactions across regions, use Spanner"
- **Unique:** Only database offering external consistency at global scale

### Pub/Sub
- **Serverless messaging** — auto-scales, global
- **At-least-once delivery** with exactly-once processing capability
- **HLD use:** "Event bus for microservices. Service A publishes to topic, Services B, C, D subscribe."
- **vs Kafka:** Fully managed, no partitions to manage, but less control

### GKE (Google Kubernetes Engine)
- **Managed Kubernetes** — auto-upgrades, auto-repair, integrated logging
- **Autopilot mode:** Google manages nodes too (truly serverless K8s)
- **HLD use:** "Microservices deployed on GKE with Istio service mesh"
- **Advantage:** Google created Kubernetes → GKE is the most mature managed K8s

---

## 📝 Cloud Interview Tips

1. **Always think in terms of managed services** — don't say "we'll deploy MySQL on an EC2" when RDS exists
2. **Know the pricing model** — on-demand vs reserved vs spot for compute, storage classes for S3
3. **Multi-region ≠ Multi-AZ** — know the difference and cost implications
4. **Security** — always mention IAM, encryption at rest/transit, VPC isolation
5. **When discussing HLD:** Map abstract components to real cloud services. "The cache" → "ElastiCache Redis". "The queue" → "SQS". This shows practical knowledge.
