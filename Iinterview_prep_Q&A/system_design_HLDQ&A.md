==============================
FILE: System Design (HLD)
==============================

### HIGH PRIORITY

---

Q1. How would you design a URL shortener? What are the key decisions around ID generation, storage, and redirection?

A1.
The core is simple: map a short code to a long URL. But the design decisions are what interviewers care about.

**ID generation**: You need a unique short code. Options: hash the long URL (MD5/SHA and take first 7 chars — risk of collisions), use a counter-based approach (auto-increment ID encoded in base62 — predictable but has no collisions), or pre-generate random codes in batches. I'd go with base62-encoded auto-increment IDs behind a distributed ID generator like Snowflake — no collisions, short codes, and decentralized.

**Storage**: Key-value store (like DynamoDB or Redis) is ideal — it's just a lookup from short code to long URL. If you need analytics (click counts, referrers), add a separate analytics store.

**Redirection**: 301 (permanent redirect) is cacheable by browsers, reducing load but losing analytics visibility. 302 (temporary redirect) forces every click through your server — better for tracking. Most URL shorteners use 302 or 307.

**Scale considerations**: Read-heavy (1000:1 read:write ratio). Cache hot URLs in Redis. Bloom filter to quickly check if a code exists before hitting the DB. Geographically distributed caches for global latency.

---

Q2. How would you design a rate limiter that works across multiple servers?

A2.
A single-server rate limiter is trivial — keep counters in memory. The challenge is making it work across a distributed fleet where any server can receive any request.

The key is centralized state. Use Redis as the shared counter store. For a sliding window counter: the key is `rate:{user_id}:{window}`, and you use `INCR` with `EXPIRE` atomically.

For a token bucket approach: store the token count and last refill timestamp in Redis. On each request, calculate how many tokens should have been added since the last refill, update the count, and consume a token — all in a Lua script to ensure atomicity.

**Edge cases**: What if Redis is down? You have two options — fail open (allow all requests, risk abuse) or fail closed (reject all requests, risk availability). Most systems fail open with a local in-memory fallback.

**Where to put it**: At the API gateway (Kong, NGINX) for global limits, or in application middleware for per-endpoint limits. You might also have different limits per tier (free users: 100/min, paid: 1000/min).

---

Q3. How would you design a notification system (push, email, SMS) that handles millions of users?

A3.
I'd think of this as an event-driven pipeline with pluggable delivery channels.

**Architecture**: An event triggers a notification (e.g., "order shipped"). The notification service receives the event, determines which users to notify, checks their preferences (do they want email? push? SMS?), templates the message, and routes it to the appropriate delivery channel.

**Components**:
- **Event ingestion**: Kafka topic for incoming notification events — decouples producers from the notification system.
- **User preferences service**: Which channel(s) does the user want? Time zone for batching? Quiet hours?
- **Template engine**: Message templates with variable substitution. Different templates per channel.
- **Channel dispatchers**: Separate workers for email (via SES, SendGrid), push (via FCM/APNs), SMS (via Twilio). Each channel has different throughput limits and failure modes.
- **Rate limiting**: Don't bombard a user with 50 notifications in a minute. Aggregate or throttle.

**Reliability**: Use persistent queues for each channel. Retry on failure. Track delivery status. Dead letter queue for permanently failed messages. Idempotency keys to prevent duplicate sends.

**Scale**: Kafka partitioned by user ID ensures ordering. Workers auto-scale based on queue depth. Priority queue for urgent notifications (security alerts) vs. batch notifications (weekly digests).

---

Q4. What is horizontal scaling vs. vertical scaling? When do you apply each?

A4.
Vertical scaling: make the machine bigger — more CPU, more RAM, faster disk. Horizontal scaling: add more machines.

Vertical scaling is simpler — no distributed system complexity. Just upgrade the server. But it has hard limits (you can only buy so big a machine), and it's a single point of failure.

Horizontal scaling is more complex — you need load balancing, data partitioning, distributed coordination. But it's theoretically unlimited, more resilient (one machine failure doesn't take everything down), and often more cost-effective at scale.

In practice: scale vertically first because it's simple. When you hit the limits of a single machine (usually database first), then scale horizontally. Stateless application servers are easy to scale horizontally. Databases are harder — that's where sharding, read replicas, and distributed databases come in.

---

Q5. What is the CAP theorem? How does it influence your choice of database in a distributed system?

A5.
CAP says a distributed system can only guarantee two of three properties: **Consistency** (every read gets the most recent write), **Availability** (every request gets a response), and **Partition tolerance** (the system works despite network failures between nodes).

Since network partitions are inevitable in any distributed system, the real choice is between CP and AP during a partition:
- **CP (Consistency + Partition tolerance)**: During a partition, the system rejects requests rather than serve stale data. Example: PostgreSQL with synchronous replication, HBase, MongoDB (with majority write concern).
- **AP (Availability + Partition tolerance)**: During a partition, the system continues serving requests, even if some responses might be stale. Example: Cassandra, DynamoDB, CouchDB.

The choice depends on your use case. Banking transactions need consistency — you can't show the wrong balance. A social media "like count" can tolerate eventual consistency — being a few seconds behind is fine.

In practice, most modern systems are tunable — Cassandra lets you configure consistency level per query.

---

Q6. What is eventual consistency, and how does it differ from strong consistency? Give an example of where each is appropriate.

A6.
Strong consistency: after a write completes, every subsequent read sees that write. No matter which node you read from. Simple to reason about but expensive — it requires coordination between nodes.

Eventual consistency: after a write, reads *might* return stale data temporarily, but eventually (usually milliseconds to seconds) all nodes converge to the same value. Faster and more available.

Strong consistency is appropriate for: bank account balances, inventory counts (you can't sell what you don't have), user authentication state (is this session valid?).

Eventual consistency is appropriate for: social media feeds, product review counts, recommendation scores, analytics dashboards. A like count showing 1,042 instead of 1,043 for a few seconds isn't a problem.

The nuance: "eventually" isn't "whenever." In well-designed systems, convergence is usually sub-second. The real concern is: what's the worst that happens during that window? If the answer is "nothing important," eventual consistency is the right choice because it gives you better performance and availability.

---

Q7. What is database sharding, and what are the strategies for choosing a shard key?

A7.
Sharding splits your database horizontally — different rows live on different servers. Each shard holds a subset of the data. This is how you scale writes beyond what a single database can handle.

**Shard key selection** is the most critical decision:
- **Hash-based**: Hash the key and mod by number of shards. Distributes data evenly but makes range queries across shards expensive. Good for: user data sharded by user_id.
- **Range-based**: Shard by ranges (users A-M on shard 1, N-Z on shard 2). Supports range queries but risks hot spots if distribution is uneven.
- **Directory-based**: A lookup table maps keys to shards. Most flexible but the directory is a single point of failure.

**Good shard key**: evenly distributes data, avoids cross-shard queries for common access patterns, and won't need to change. Usually the primary entity ID that most queries filter by.

**Bad shard key**: anything with low cardinality (country — you'd have one huge US shard), anything that creates hotspots (timestamp — all current writes hit the latest shard), or anything that forces frequent cross-shard joins.

---

Q8. What is caching, and where do you place caches in a system (client, CDN, application, database)?

A8.
Caching stores frequently accessed data closer to where it's needed, reducing latency and load on the origin.

**Layers**:
- **Client-side**: Browser cache, mobile app cache. Reduces network requests entirely. Controlled via HTTP headers (Cache-Control, ETag).
- **CDN**: Caches static assets and even API responses at edge locations globally. Reduces latency for geographically distributed users. CloudFront, Cloudflare.
- **Application-level**: Redis or Memcached in front of your database. Cache query results, computed values, session data. This is the most impactful layer for most backends.
- **Database-level**: Query cache, buffer pool. The database caches frequently accessed pages in memory automatically.

The closer the cache is to the user, the less work your system does. But cache invalidation becomes harder the farther it is — how do you tell a CDN in Tokyo that data changed in your US database?

Each layer adds a cache invalidation challenge. The general principle: cache aggressively at each layer, but ensure you have a strategy for invalidation or acceptable staleness.

---

Q9. What are cache invalidation strategies (TTL, write-through, write-behind, cache-aside)?

A9.
**Cache-aside (lazy loading)**: Application first checks the cache. On miss, it reads from the database, writes the result to cache, and returns. On write, the application updates the database and invalidates (deletes) the cache entry. This is the most common pattern — simple and effective. Risk: the first request after invalidation is slow (cache miss).

**Write-through**: Every write goes to both the cache and the database simultaneously. Cache is always up to date. But writes are slower (two writes per operation), and you cache data that might never be read.

**Write-behind (write-back)**: Writes go to the cache immediately, and the cache asynchronously flushes to the database. Very fast writes, but you risk data loss if the cache crashes before flushing.

**TTL (Time-to-Live)**: Cache entries expire after a set duration. Simple but imprecise — data could be stale for up to the TTL duration. Good as a safety net on top of other strategies.

In practice, I use cache-aside with TTL as a fallback. For critical, frequently-changing data (like inventory), short TTLs (seconds). For slowly-changing data (user profiles), longer TTLs (minutes to hours).

---

Q10. What is a CDN, and how does it improve system performance and availability?

A10.
A CDN (Content Delivery Network) is a globally distributed network of edge servers that cache and serve content close to users. Instead of every request traveling to your origin server in Virginia, a user in Mumbai gets served from a nearby edge node.

**Performance**: Reduces latency by serving from a geographically closer location. CDN edge to user is often 5-20ms vs 200-400ms to origin. Also reduces the number of requests hitting your origin, so it can handle more unique traffic.

**Availability**: If your origin goes down, the CDN can continue serving cached content. It's an implicit fallback layer.

Beyond static assets (images, CSS, JS), modern CDNs cache API responses, run edge functions (Cloudflare Workers, Lambda@Edge), and handle DDoS protection.

The main challenge is cache invalidation — when you deploy a new version, you need to purge the CDN cache. Using versioned URLs (`style.abc123.css`) avoids this entirely.

---

Q11. What is a load balancer? What are the differences between L4 and L7 load balancing?

A11.
A load balancer distributes incoming traffic across multiple servers so no single server is overwhelmed, and if one fails, traffic routes to healthy ones.

**L4 (Transport layer)**: Makes routing decisions based on TCP/UDP headers — IP address and port. It doesn't look at the request content. Fast because it operates at lower level. Good for raw TCP traffic, database connections.

**L7 (Application layer)**: Makes routing decisions based on HTTP content — URL path, headers, cookies. It can route `/api/*` to backend servers and `/static/*` to a CDN origin. It can do SSL termination, request modification, and WAF (web application firewall) filtering.

**Algorithms**: Round robin (simplest), weighted round robin (more traffic to beefier servers), least connections (send to the server with fewest active requests), IP hash (same client always hits the same server — useful for session affinity).

In practice, you typically use L7 for HTTP traffic (NGINX, ALB) because you need content-based routing, and L4 for non-HTTP traffic like database proxies or gaming servers (NLB).

---

Q12. How would you design a chat/messaging system that supports real-time and offline delivery?

A12.
This has two distinct paths: real-time (user is online) and store-and-forward (user is offline).

**Real-time path**: WebSocket connections between clients and chat servers. When User A sends a message to User B, the server looks up B's WebSocket connection and pushes the message. If they're on different chat servers, you need a pub/sub layer (Redis Pub/Sub or Kafka) between servers.

**Offline path**: If User B isn't connected, store the message in a persistent store (Cassandra works well — write-optimized, wide-column, good for time-series-like message data). When B comes online, the server fetches undelivered messages and pushes them.

**Key components**:
- **Connection manager**: Tracks which user is connected to which server. Redis hash map: `user_id → server_id`.
- **Message queue**: For reliable delivery — messages go through a queue before being sent.
- **Message storage**: Two models — per-conversation (write once, both users reference same copy) or per-user inbox (copy per recipient — easier reads, more storage).
- **Presence service**: Who's online? Heartbeat-based with a timeout.

**Scale**: Partition by user/conversation ID. Group chats are harder — a message to a 1000-person group means 1000 deliveries. Fan-out on write (push to each inbox) vs fan-out on read (query the group's message stream on load).

---

### MEDIUM PRIORITY

---

Q13. How would you design a distributed job scheduler that handles millions of scheduled tasks?

A13.
The core challenge is efficiently finding which jobs are due at any given moment and executing them reliably.

**Storage**: Use a database with efficient range queries — like a sorted set in Redis (score = execution timestamp) or a time-partitioned table in PostgreSQL. Workers poll: "give me all jobs where execution_time <= now."

**Execution**: Workers pull jobs, execute them, and mark them complete. Use distributed locking (Redis SETNX or database row locking) to prevent two workers from picking the same job.

**Reliability**: Jobs must be at-least-once. If a worker dies mid-execution, the job should become eligible for re-pickup after a visibility timeout (like SQS). Store execution state: pending → in_progress → completed/failed.

**Scale**: Partition jobs by time buckets (minute-level partitions). Don't have every worker scan the entire job set — assign time ranges to worker groups. Use consistent hashing to distribute job ownership across workers.

**Recurring jobs**: Store the cron/interval definition separately. A "scheduler" process creates individual job instances ahead of time. This separates the schedule definition from execution.

---

Q14. How would you design a search autocomplete/typeahead system?

A14.
The user types "new y" and gets suggestions: "new york", "new year", "new york times".

**Data structure**: A trie is the classic answer — each path represents a prefix, and nodes store suggestion candidates. But at scale, you'd use a precomputed dictionary: for each prefix, store the top-k suggestions sorted by relevance/popularity.

**Architecture**: Precompute prefix-to-suggestions mappings offline. Store in a fast KV store (Redis) or a dedicated cache. When the user types "new y", look up "new y" and return cached results. No computation at query time.

**Ranking**: Rank suggestions by search frequency, recency, and personalization. Update rankings periodically (batch) rather than real-time.

**Performance**: Latency must be < 50ms — users are typing continuously. Use browser-side debouncing (wait 100ms after keystroke before querying). Cache results client-side for visited prefixes.

**Scale**: The prefix space is bounded. If you only care about prefixes up to 10 characters long, the dataset is manageable. Distribute across regions — typeahead is highly local (language, region-specific terms).

---

Q15. What is consistent hashing, and how does it help when adding/removing nodes in a distributed cache?

A15.
With regular hashing (`key % N`), adding or removing a node changes N, so almost every key remaps — causing a massive cache miss storm.

Consistent hashing places both keys and nodes on a virtual ring (0 to 2^32). Each key is assigned to the first node clockwise from its position. When you add a node, only the keys between it and the previous node remigrate — roughly 1/N of all keys. When you remove a node, its keys move to the next node. Minimal disruption.

**Virtual nodes**: Physical nodes get multiple positions on the ring to ensure even distribution. A powerful node gets more virtual nodes, handling more keys.

This is fundamental to distributed caches (Memcached), distributed databases (Cassandra, DynamoDB), and CDN edge node assignment.

---

Q16. What is a write-ahead log (WAL), and why is it fundamental to database durability?

A16.
A WAL logs every change before it's applied to the actual data files. If the system crashes, the WAL is replayed to recover committed transactions.

The key insight: writing to disk sequentially (appending to a log) is much faster than writing to random positions (updating data pages). So the database appends the change to the WAL (fast), acknowledges the transaction, and lazily applies changes to data files later.

If the machine crashes, on restart: replay the WAL from the last checkpoint. Committed transactions are re-applied. Uncommitted transactions are rolled back. Data integrity is guaranteed.

PostgreSQL, MySQL InnoDB, SQLite, and most databases use WAL. Kafka's commit log is essentially the same concept — the log IS the data.

---

Q17. How would you design an analytics event collection pipeline that ingests billions of events daily?

A17.
The key constraints: high write throughput, fault tolerance, and eventual queryability.

**Ingestion layer**: Lightweight HTTP endpoint (or SDK) that accepts events and immediately writes to Kafka. Keep the ingestion path as thin as possible — validate the schema and push to Kafka. Don't do processing at ingestion time.

**Processing layer**: Kafka consumers (Spark Streaming, Flink, or simple consumers) read events, enrich them (add geo from IP, resolve user IDs), and write to a data warehouse.

**Storage layer**: Write raw events to a data lake (S3 in Parquet format, partitioned by date and event type). Aggregated summary tables in a data warehouse (BigQuery, Redshift, ClickHouse) for fast queries.

**Scale**: Kafka handles the throughput — partition by event type or user_id for parallelism. Batch writes to the warehouse every few minutes. Use columnar storage for query efficiency.

**Reliability**: Events in Kafka are durable (replicated). If a consumer fails, it resumes from its last committed offset. The pipeline is naturally idempotent if consumers handle deduplication.

---

Q18. What is the difference between a push model and a pull model in news feed/timeline design?

A18.
**Fan-out on write (push)**: When a user posts, immediately push the post to all followers' feed caches. Each follower's feed is pre-built — fast reads (O(1) — just fetch the cache). But writes are expensive for users with millions of followers — pushing one post to 10M feeds takes time and resources.

**Fan-out on read (pull)**: Feeds are built on demand. When a user opens their feed, the system queries all users they follow and merges recent posts in real-time. Writes are cheap, but reads are slow — especially if you follow 1000 people.

**Hybrid (what Twitter/Facebook does)**: Use push for most users and pull for celebrity accounts. If a user has < 10K followers, push to all. If they have millions, let followers pull from their timeline at read time. This handles the celebrity problem without sacrificing read speed for normal users.

---

Q19. How would you design a file storage service (e.g., Google Drive, Dropbox)?

A19.
**Core components**:
- **Metadata service**: Stores file hierarchy, permissions, versions, sharing settings. Relational database (PostgreSQL) — ACID transactions for permission changes.
- **Block storage**: Split files into fixed-size blocks (4MB). Store blocks in blob storage (S3). This enables: resumable uploads, efficient sync (only upload changed blocks), and deduplication (same blocks across files stored once).
- **Sync engine**: Client tracks local file changes, computes block-level diffs, uploads changed blocks, and updates metadata. The server resolves conflicts (last-write-wins or branching).

**Sync notifications**: When User A changes a file shared with User B, User B needs to know. Long polling or WebSocket from client to a notification service. The notification triggers a sync pull.

**Versioning**: Store block diffs or keep N versions of each file's block manifest. Restoring a version = reassembling from stored blocks.

**Scale**: Block storage scales horizontally via blob storage. Metadata service sharded by user or workspace. Sync traffic is bursty — handle with queues.

---

Q20. What is leader election, and when is it necessary in a distributed system?

A20.
Leader election is the process of designating one node as the "leader" that coordinates work, while others are followers. It's needed when you need exactly one node to perform a task — running a cron job, coordinating writes to avoid conflicts, or managing cluster membership.

**Approaches**:
- **ZooKeeper/etcd**: Nodes attempt to create an ephemeral node. The first one wins and becomes leader. If the leader dies, the ephemeral node disappears, and others compete again.
- **Raft consensus**: Nodes elect a leader through a voting protocol. The leader handles all writes and replicates to followers.
- **Database-based**: Acquire a distributed lock (row-level lock with heartbeat). Simple but less robust.

**Challenges**: Split-brain — network partition makes two groups, each thinks it's the leader. Fencing tokens (monotonically increasing tokens) help — the old leader's requests are rejected because they have a stale token.

---

Q21. How would you handle data replication across geographically distributed regions?

A21.
**Synchronous replication**: Write is confirmed only after all replicas acknowledge. Strong consistency, but high latency (a write from US waits for confirmation from Europe and Asia). Good for critical data where consistency is non-negotiable.

**Asynchronous replication**: Write is confirmed after the local region persists. Replicas sync in the background. Low latency for writes, but risk of data loss if the primary fails before replicating.

**Conflict resolution**: With multi-region writes, two users might update the same record in different regions simultaneously. Strategies: last-writer-wins (simple but can lose data), vector clocks (track causality), CRDTs (conflict-free by design), and application-level resolution (show both versions to the user).

**Practical choices**: Use a single write region (primary) with read replicas in other regions — simplest, no conflicts. If you need multi-region writes, use a database designed for it (CockroachDB, Spanner, DynamoDB Global Tables).

---

### LOW PRIORITY

---

Q22. How would you design a distributed unique ID generator (e.g., Snowflake IDs)?

A22.
You can't use a single auto-incrementing counter — it's a bottleneck and single point of failure. Twitter's Snowflake approach solves this:

A 64-bit ID composed of: timestamp (41 bits, ~69 years), machine/datacenter ID (10 bits, 1024 machines), and sequence number (12 bits, 4096 IDs per millisecond per machine).

Each machine generates IDs independently — no coordination. IDs are roughly time-ordered (the timestamp is the most significant bits), which is great for database indexing. Any machine can generate 4096 unique IDs per millisecond.

Alternative approaches: UUIDs (128-bit, universally unique, but large and random — poor index performance), or database-backed ID ranges (each server pre-allocates a block of IDs from a central service). Snowflake strikes the best balance for most use cases.

---

Q23. What is the Raft or Paxos consensus algorithm at a high level? Why is consensus hard?

A23.
Consensus is getting multiple nodes to agree on a value, even when some nodes fail or messages are delayed.

**Raft** (designed to be understandable): Nodes are in one of three states: leader, follower, or candidate. A leader is elected through voting. The leader handles all writes — it appends to its log and replicates to followers. A write is committed when a majority acknowledge it. If the leader dies, a follower with the most up-to-date log starts an election.

**Why it's hard**: Network partitions can cause split votes. Nodes might have different views of what's been committed. You need to handle all combinations of node failures, message delays, and network splits — and still guarantee that committed values are never lost.

Paxos was the original algorithm (Lamport, 1989) but notoriously hard to understand and implement. Raft was designed in 2014 as a more understandable alternative. In practice: etcd uses Raft, ZooKeeper uses ZAB (similar to Paxos).

---

Q24. How would you design a real-time leaderboard for a gaming platform?

A24.
The core operations: update a player's score and get the top-K or a player's rank.

**Redis Sorted Sets** are purpose-built for this. `ZADD` updates a score in O(log n), `ZRANGE` gets top-K in O(log n + K), and `ZRANK` gets a player's rank in O(log n). Redis handles millions of players with sub-millisecond latency.

For a global leaderboard: a single Redis sorted set works well up to tens of millions of entries. Beyond that, shard by score range or use partitioned leaderboards (regional, then merge).

For time-scoped leaderboards (daily, weekly): use separate sorted sets with TTL. Create a new set each period and let old ones expire.

For relative ranking ("you're #5,432 out of 1M"): Redis sorted sets give you this natively with ZRANK.

---

Q25. What is a conflict-free replicated data type (CRDT), and where would you use one?

A25.
A CRDT is a data structure designed so that concurrent updates on different replicas always converge to the same result without coordination. No conflicts, no need for consensus.

Example: a G-Counter (grow-only counter). Each node maintains its own count. The global count is the sum. Node A increments locally, Node B increments locally — they sync later, and the sum is always correct. No conflicts possible.

More complex CRDTs: LWW-Register (last-writer-wins), OR-Set (observed-remove set), and text-editing CRDTs (used in collaborative editors like Figma, Google Docs).

Use CRDTs when: you need multi-region writes with high availability, can tolerate eventual convergence, and the data model fits a CRDT type. They're common in collaborative editing, shopping carts, and distributed counters.

Limitation: not everything fits a CRDT. Bank accounts need coordination — you can't have a grow-only balance. CRDTs work for commutative, associative operations.

---

Q26. How would you design a multi-tenant SaaS platform where tenants have different data isolation requirements?

A26.
Three main isolation models:

**Shared database, shared schema**: All tenants in the same tables with a `tenant_id` column. Cheapest, easiest to manage, but requires careful query discipline (every query must filter by tenant_id), and a bug could leak data between tenants. Good for: most B2B SaaS with moderate security needs.

**Shared database, separate schemas**: Each tenant gets their own schema (set of tables) in the same database. Better isolation, easier per-tenant backups/migrations. But schema changes must be applied to all tenant schemas. Good for: tenants that need customization.

**Separate databases**: Each tenant gets their own database instance. Strongest isolation, simplest security story, easiest compliance. But expensive and operationally heavy — managing hundreds of database instances requires automation. Good for: enterprise clients with strict compliance (healthcare, finance).

In practice, most platforms use shared schema for small tenants and separate databases for enterprise clients. The platform needs a routing layer that maps each request to the right database/schema based on the tenant.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q27. How would you design a video streaming platform (e.g., YouTube/Netflix)? Cover upload, transcoding, storage, and adaptive streaming.

A27.
This is a multi-pipeline system with distinct upload and viewing paths.

**Upload pipeline**: User uploads a video → store the raw file in object storage (S3). Put a transcoding job in a queue. Workers transcode the video into multiple resolutions (360p, 720p, 1080p, 4K) and formats (H.264, VP9, AV1). Generate thumbnails. Store transcoded segments in object storage. Update the metadata database (title, duration, resolution availability).

**Storage**: Videos are chunked into small segments (2-10 seconds). Each resolution/codec combination of each segment is a separate file. A 10-minute video might produce thousands of segment files. Use a CDN to cache popular videos near users — most views cluster around a small percentage of videos (power law).

**Adaptive streaming (HLS/DASH)**: The player starts streaming at a default quality. Based on the client's bandwidth and buffer level, it dynamically switches between resolutions segment by segment. Slow connection → drop to 480p. Fast connection → jump to 1080p. A manifest file lists available qualities and segment URLs.

**Architecture**: Upload → Object Storage → Transcoding Queue → Worker Fleet → CDN → Client. Metadata and user data in a relational database. Recommendations and search as separate services.

**Key challenges**: Transcoding is CPU-intensive — use spot/preemptible instances. CDN costs dominate. Video deduplication. Copyright detection (Content ID). Live streaming adds real-time encoding constraints.

---

Q28. How would you design a social media news feed/timeline (e.g., Twitter/Instagram)?

A28.
The core question: how do you generate a personalized feed for each user from millions of posts by the people they follow?

**Push model (fan-out on write)**: When a user posts, immediately push the post to all followers' feeds (stored as a precomputed list in Redis/Cassandra). When a follower opens their feed, it's already built — just read it. **Pros**: Fast reads (O(1)). **Cons**: Expensive writes. A celebrity with 50M followers → 50M writes per post. Wasted if most followers never check their feed.

**Pull model (fan-out on read)**: When a user opens their feed, fetch recent posts from all people they follow, then merge and rank. **Pros**: No wasted writes. **Cons**: Slow reads — fetching from 500 followed accounts and merging is expensive.

**Hybrid (what Twitter/Instagram actually does)**: Push for regular users (pre-compute feeds). Pull for celebrities (fan-out on read — fetch celebrity posts at view time and merge into the precomputed feed). Threshold: users with >500K followers use pull.

**Ranking**: Simple feeds use reverse chronological order. Modern feeds use ML-ranked feeds — predict engagement probability for each candidate post and rank accordingly.

**Infrastructure**: Write path → Message queue → Fan-out service → Feed cache (Redis). Read path → Feed service → merge cached feed + celebrity pull + ranked ads → return to client.

---

Q29. How do you do back-of-the-envelope estimation / capacity planning for a system design interview?

A29.
**Step 1: Clarify scale**: Daily active users (DAU), requests per second (RPS), data volume. Example: 100M DAU, each user makes 10 requests/day.

**Step 2: Compute RPS**: 100M × 10 / 86,400 seconds ≈ 12,000 RPS average. Peak is typically 2-3x average → ~30,000 RPS peak.

**Step 3: Storage**: If each request generates 1KB of data → 100M × 10 × 1KB = 1TB/day → 365TB/year. With replication (3x) → ~1PB/year.

**Step 4: Bandwidth**: 1TB/day ÷ 86,400 ≈ 12MB/s average. Peak → ~30MB/s. For video: 5MB average video × 10M uploads/day = 50TB/day.

**Key numbers to memorize**: 1 day ≈ 100K seconds. 1 server handles ~1K-10K QPS (depending on complexity). SSD random read ≈ 100μs. HDD ≈ 10ms. Network roundtrip within datacenter ≈ 0.5ms. Cross-region ≈ 50-150ms. Redis GET ≈ 0.1ms. MySQL query ≈ 1-10ms.

**Purpose in interviews**: Not about exact numbers — it's about demonstrating order-of-magnitude reasoning. "We need ~10K QPS, a single server handles ~5K, so 2-3 servers with a load balancer." Shows you understand the scale and can justify architectural decisions.

---

Q30. How would you design a payment processing system with exactly-once semantics?

A30.
Payments are the hardest distributed systems problem — duplicate charges, lost transactions, and partial failures are unacceptable.

**Idempotency key**: Every payment request includes a client-generated unique key. The server stores the key → result mapping. If the same key comes again (retry), return the cached result without reprocessing. This is the foundation of exactly-once payments.

**State machine**: Payment goes through states: `CREATED → AUTHORIZED → CAPTURED → SETTLED`. Each transition is idempotent. Moving from AUTHORIZED to CAPTURED twice is a no-op.

**Double-entry ledger**: Every money movement creates two entries: debit from one account, credit to another. The sum of all entries is always zero. This ensures accounting integrity even during failures.

**Architecture**: API Gateway → Payment Service (validates, creates payment record with idempotency key) → Payment Provider Integration (Stripe/Adyen) → Webhook handler (receives async confirmation) → Ledger service (records the transaction) → Notification service (receipt to user).

**Critical patterns**: Outbox pattern for reliable event publishing. Saga pattern for multi-step flows (authorize → reserve inventory → capture → send receipt). Compensating transactions for rollback (refund if inventory reservation fails after authorization).

**Reconciliation**: A separate batch job reconciles your records against the payment provider's records daily. Catches discrepancies caused by dropped webhooks or network failures.

---

### IMPORTANT

---

Q31. How would you design a web crawler that scales to billions of pages?

A31.
**Core loop**: Pick a URL from the frontier → fetch the page → parse HTML → extract links → add new links to the frontier → store the content.

**Frontier (URL queue)**: Priority queue — important/fresh URLs first. Politeness constraints — don't DDoS any single domain (per-domain rate limiting). De-duplication — don't re-crawl known URLs (Bloom filter check before enqueuing).

**Fetching**: Distributed fetcher workers. DNS resolution is a bottleneck — cache DNS results. Respect robots.txt (check before crawling). Handle redirects, timeouts, and errors gracefully.

**Content processing**: Parse HTML, extract text and links. Detect near-duplicate pages (SimHash/MinHash) — the web has many similar pages. Detect language, extract metadata.

**Storage**: URL database (what we've crawled, when, status). Content store (object storage for raw pages). Inverted index (for search — separate system).

**Scale**: Billions of URLs → distributed frontier across multiple nodes (consistent hashing by domain). Hundreds of fetcher workers. Prioritize by: PageRank, freshness requirements, content change frequency. Total state: ~100TB+ for the URL database alone.

---

Q32. How would you design an e-commerce system (product catalog, cart, checkout, inventory)?

A32.
**Core services**:
- **Product Catalog**: Products, categories, search. Read-heavy — cached aggressively. Elasticsearch for search. Database for structured product data.
- **Cart**: Per-user basket. Store in Redis (fast access, auto-expiry for abandoned carts). Sync to database periodically.
- **Inventory**: Track stock levels. Decrement on purchase. The critical challenge: preventing overselling. Use optimistic locking (`UPDATE stock SET count = count - 1 WHERE product_id = X AND count > 0`).
- **Checkout/Order**: Create order, process payment, reserve inventory. Multi-step — use the Saga pattern. If payment fails → release inventory. If inventory is out → refund payment.
- **Pricing**: Dynamic pricing, discounts, coupons. Complex rules engine. Computed server-side (never trust client-sent prices).

**Flash sale challenge**: 100K users click "buy" simultaneously for 100 items. Queue the requests — first 100 get the item, rest get "sold out." Pre-computed tokens or distributed locks to prevent overselling.

**Architecture**: API Gateway → microservices (catalog, cart, inventory, order, payment) → async events for non-critical tasks (send email, update analytics). Event sourcing for orders (full audit trail of every state change).

---

Q33. How would you design a metrics collection and monitoring system (e.g., Datadog/Prometheus)?

A33.
**Collection**: Agents on each server collect metrics (CPU, memory, disk, custom app metrics). Push to a central collection endpoint or expose via a pull endpoint (`/metrics` for Prometheus).

**Ingestion pipeline**: High throughput — thousands of servers sending metrics every 10-60 seconds. Kafka as a buffer → stream processor (aggregate, downsample) → time-series database.

**Storage**: Time-series database (InfluxDB, TimescaleDB, or Prometheus TSDB). Optimized for write-heavy workloads and range queries over time. Data retention policies: full resolution for 7 days, 1-minute aggregates for 30 days, 1-hour aggregates for 1 year.

**Querying and visualization**: Query language for aggregation over time windows (PromQL, InfluxQL). Dashboards (Grafana). "Show me P99 latency for the payment service over the last 24 hours."

**Alerting**: Define rules — "if error rate > 5% for 5 minutes, alert on-call." PagerDuty/OpsGenie for escalation. Avoid alert fatigue — alert on symptoms (high error rate), not causes (high CPU). Group related alerts.

**Scale challenge**: A 1000-server deployment generating 100 metrics each at 10-second intervals = 10K data points/second = 864M/day. Cardinality explosion (high-cardinality labels like user_id) can break time-series databases.

---

Q34. How would you design a distributed key-value store?

A34.
**Core operations**: `GET(key)`, `PUT(key, value)`, `DELETE(key)`. Must be distributed (no single server holds all data), durable, and available.

**Data distribution**: Consistent hashing to map keys to nodes. Each key is replicated to N nodes (typically 3) for durability.

**Consistency model**: Choose on the consistency spectrum. Strong consistency: reads always return the latest write (Raft/Paxos consensus — slower). Eventual consistency: writes propagate asynchronously — reads may return stale data briefly (faster, higher availability). Tunable consistency (Cassandra): `W + R > N` for strong, `W=1, R=1` for eventual.

**Write path**: Client → coordinator node → replicate to N nodes → respond after W acknowledgments. Write-ahead log on each node for durability. Memtable (in-memory) → SSTable (on disk) — LSM tree architecture.

**Read path**: Query R replicas → return the latest value (by timestamp/version). Read repair: if replicas disagree, update the stale one.

**Failure handling**: Hinted handoff — if a replica is down, store the write temporarily on another node and forward when it recovers. Anti-entropy (Merkle trees) — periodically verify replicas are in sync.

**Examples**: DynamoDB, Cassandra, Riak. Trade-offs follow CAP theorem — DynamoDB chooses AP (availability), Etcd chooses CP (consistency).

---

Q35. What is the difference between synchronous and asynchronous replication? What are the trade-offs?

A35.
**Synchronous replication**: A write isn't acknowledged until it's persisted on ALL replicas. The client waits. **Pros**: No data loss — if the primary dies, replicas have all data. Strong consistency. **Cons**: Slow — latency is the maximum of all replica write latencies. If any replica is down or slow, writes are blocked.

**Asynchronous replication**: The primary acknowledges the write immediately, replicates to followers in the background. **Pros**: Fast writes — client doesn't wait for replication. System remains available even if replicas lag. **Cons**: Data loss risk — if the primary fails before replication, uncommitted writes are lost. Replication lag causes stale reads from replicas.

**Semi-synchronous**: Write to at least one replica synchronously, rest asynchronously. Compromise — guaranteed one backup, but faster than fully synchronous. PostgreSQL's synchronous_commit setting supports this.

**In practice**: Most production databases use asynchronous replication for performance. Critical data (financial transactions) may use synchronous replication or consensus protocols (Raft). MySQL default is async. PostgreSQL supports both. Trade-off: latency vs durability.

---

### GOOD-TO-HAVE

---

Q36. How would you design a collaborative document editing system (e.g., Google Docs)?

A36.
**Core challenge**: Multiple users editing the same document simultaneously. Edits must not conflict or overwrite each other.

**Operational Transformation (OT)**: Each edit is an operation (insert char at position X, delete char at position Y). When two concurrent operations arrive, transform one against the other so both can be applied in any order and produce the same result. Used by Google Docs. Complex to implement — many edge cases.

**CRDTs (Conflict-free Replicated Data Types)**: Model the document as a CRDT — concurrent edits merge automatically without conflicts. Simpler semantics but can produce surprising results (concurrent inserts at the same position). Used by Figma, Yjs, Automerge.

**Architecture**: Client → WebSocket connection to a collaboration server → OT/CRDT engine resolves conflicts → broadcast changes to all other clients → persist to database periodically.

**Presence and cursors**: Show where other users are editing. Broadcast cursor positions via the same WebSocket. Debounce updates (every 100ms, not every keystroke).

**Offline support**: Queue local operations. When reconnecting, send all queued operations — OT/CRDT engine merges them with operations from other users.

---

Q37. How would you design a ride-sharing system (matching, ETA, pricing)?

A37.
**Core services**:
- **Matching**: Riders request a ride → find the nearest available driver. Use a geospatial index (H3 hexagonal grid or Geohash). Query drivers within a radius, rank by distance/ETA. Send ride offers to drivers — first to accept wins.
- **ETA estimation**: Graph-based routing (road network) + real-time traffic data + ML corrections. Pre-compute routes between common zones. Cache heavily.
- **Dynamic pricing (surge)**: Supply/demand model per geographic zone. High demand + low supply → surge multiplier increases. Recompute every few minutes. Anti-gaming: lock the price when the rider confirms.

**Location tracking**: Drivers send GPS updates every 3-5 seconds. Store in a streaming system (Kafka) → update the geospatial index in real-time. Historical locations in a time-series store.

**Architecture**: API servers → matching service → location service → pricing service → payment service → notification service (push notifications to drivers/riders). All async communication via event bus.

---

Q38. What are the common single points of failure in a system, and how do you eliminate them?

A38.
**Database**: Single database server → add replicas (read replicas for scaling reads, failover replica for availability). Use a managed database with automatic failover (AWS RDS Multi-AZ).

**Load balancer**: Single LB → use redundant LBs in active-passive or active-active setup. DNS-based failover (Route 53 health checks). Cloud LBs (ALB/NLB) are inherently redundant.

**DNS**: If your DNS provider goes down, nobody can resolve your domain. Use multiple DNS providers. Or use anycast DNS (Cloudflare) that's inherently distributed.

**Single service**: If one instance of a microservice crashes → run multiple replicas behind a load balancer. K8s handles this: `replicas: 3` ensures 3 instances are always running.

**Region failure**: Single data center → multi-region deployment. Active-active (both regions serve traffic) or active-passive (standby region takes over on failure). Data replication across regions adds latency.

**General principle**: Never have exactly one of anything critical. Test failures actively — chaos engineering (Netflix's Chaos Monkey randomly kills instances to verify resilience).