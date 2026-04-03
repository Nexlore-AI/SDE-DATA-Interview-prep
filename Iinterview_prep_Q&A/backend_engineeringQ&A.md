==============================
FILE: Backend Engineering
==============================

### HIGH PRIORITY

---

Q1. What is a REST API? What makes an API truly RESTful vs. just using HTTP?

A1.
REST (Representational State Transfer) is an architectural style for designing networked APIs. A lot of people call any HTTP API "RESTful," but that's not accurate.

A truly RESTful API follows specific constraints: it uses resources identified by URIs (`/users/123`), manipulates them through HTTP methods (GET to read, POST to create, PUT/PATCH to update, DELETE to remove), is stateless (each request carries all needed context), and supports HATEOAS (responses include links to related resources).

In reality, most production APIs follow REST loosely — they use resources and HTTP methods, but few implement HATEOAS. That's fine — what matters is consistency: predictable URL patterns, proper use of HTTP verbs and status codes, and stateless request handling.

The real question an interviewer is testing: do you understand the principles behind the design, or do you just think "HTTP endpoint = REST"?

---

Q2. What is the difference between synchronous and asynchronous request handling in a backend service?

A2.
Synchronous handling means the server thread processes the request start to finish — it blocks while waiting for database queries, external API calls, etc. The thread is tied up until the response is sent.

Asynchronous handling means the thread kicks off the work (e.g., a database query) and is free to handle other requests while waiting for the result. When the result comes back, a callback or coroutine picks up where it left off.

The practical impact: a synchronous server with 200 threads can handle at most 200 concurrent requests. If each request takes 500ms (mostly waiting for I/O), throughput is limited. An async server can handle thousands of concurrent requests with far fewer threads because threads aren't blocked on I/O.

Node.js is inherently async (event loop). Python has `asyncio` and frameworks like FastAPI. Java has Project Loom (virtual threads). For I/O-bound services (most web APIs), async gives you massive throughput improvements. For CPU-bound work, async doesn't help — you need actual parallelism.

---

Q3. How do you design an API for idempotency? Why is it important for payment or order endpoints?

A3.
An idempotent API produces the same result whether you call it once or multiple times with the same input. GET, PUT, and DELETE are naturally idempotent. POST is not — two identical POST requests might create two orders.

For payments, this is critical. If a client sends a payment request, the network times out, and the client retries — without idempotency, you charge the customer twice. That's a real business problem.

The standard approach is an **idempotency key**. The client generates a unique key (UUID) and sends it with the request. The server checks: "Have I seen this key before?" If yes, return the stored response without reprocessing. If no, process the request and store the result keyed by that ID.

Implementation-wise, you'd store idempotency keys in Redis or a database with a TTL (24-48 hours typically). Stripe, PayPal, and most payment APIs use this exact pattern.

---

Q4. What are HTTP status codes? When would you return 200, 201, 204, 400, 401, 403, 404, 409, 429, 500, 502, 503?

A4.
Status codes communicate what happened with the request. Using them correctly makes APIs self-documenting.

- **200 OK**: Request succeeded, here's the data.
- **201 Created**: Resource was created (after a POST). Include `Location` header pointing to the new resource.
- **204 No Content**: Success but nothing to return (common for DELETE).
- **400 Bad Request**: Client sent invalid data — missing fields, wrong format.
- **401 Unauthorized**: Authentication required or token is invalid/expired.
- **403 Forbidden**: Authenticated but not authorized for this action.
- **404 Not Found**: Resource doesn't exist.
- **409 Conflict**: Request conflicts with current state — like trying to create a user with an email that already exists.
- **429 Too Many Requests**: Rate limit exceeded. Include `Retry-After` header.
- **500 Internal Server Error**: Something broke on the server — unhandled exception.
- **502 Bad Gateway**: The server, acting as a proxy, got an invalid response from upstream.
- **503 Service Unavailable**: Server is overloaded or under maintenance. Temporary.

The key is: 4xx means the client messed up, 5xx means the server messed up. Getting this right helps clients distinguish between "retry" and "fix the request."

---

Q5. What is the difference between authentication and authorization? Describe at least two auth mechanisms.

A5.
Authentication is "who are you?" — verifying identity. Authorization is "what can you do?" — verifying permissions. You always authenticate first, then authorize.

Two common mechanisms:

**JWT (JSON Web Token)**: Stateless. After login, the server issues a signed token containing user identity and claims. The client sends it with every request. The server verifies the signature without hitting a database. Trade-off: you can't easily revoke a JWT before expiry.

**Session-based auth**: Stateful. After login, the server creates a session stored server-side (in Redis, typically) and sends a session ID as a cookie. Each request includes the cookie, and the server looks up the session. Trade-off: requires server-side storage but allows easy revocation.

OAuth 2.0 is a framework for delegated authorization — "let this app access my Google Calendar" without sharing your password. It's authorization, not authentication (OpenID Connect adds the authentication layer on top of OAuth).

---

Q6. What is JWT, and how does it work for stateless authentication? What are its security limitations?

A6.
JWT is a signed token with three parts: header (algorithm), payload (claims like user ID, roles, expiration), and signature (HMAC or RSA signed). The server signs it on login, the client stores it (usually in memory or httpOnly cookie), and sends it with each request.

The server verifies the signature on each request — if it's valid and not expired, the user is authenticated. No database lookup needed — that's the "stateless" part.

Security limitations:
- **Can't revoke easily**: Once issued, a JWT is valid until it expires. If a user's account is compromised, you can't invalidate their token without maintaining a blacklist (which defeats the stateless benefit).
- **Payload is readable**: The payload is Base64-encoded, not encrypted. Anyone can read it. Never put sensitive data in a JWT.
- **Token theft**: If someone steals the JWT, they have full access until expiry. Use short expiration times + refresh tokens to limit exposure.
- **Algorithm confusion attacks**: If the server accepts `none` as an algorithm, attackers can forge tokens. Always validate the algorithm on the server side.

In practice, most teams use JWTs with short expiry (15 min) plus a refresh token flow, and store JWTs in httpOnly secure cookies rather than localStorage.

---

Q7. What is rate limiting, and how would you implement it in a backend service?

A7.
Rate limiting restricts how many requests a client can make in a given time window. It protects your service from abuse, DDoS, and noisy neighbors hogging resources.

Common algorithms:
- **Fixed window**: Count requests in fixed time slots (e.g., 100 requests per minute). Simple but has the burst problem at window boundaries — a client could send 100 at :59 and 100 at :00, effectively 200 in 2 seconds.
- **Sliding window**: Smooths out the boundary issue by using a weighted combination of current and previous windows.
- **Token bucket**: Tokens are added at a fixed rate. Each request consumes a token. If no tokens are available, the request is rejected. Allows bursts up to the bucket capacity.
- **Leaky bucket**: Requests are queued and processed at a fixed rate. Smooths traffic but adds latency.

Implementation: typically use Redis with atomic INCR and EXPIRE commands. The key is usually `rate_limit:{user_id}:{window}`. In a distributed system, all servers need a shared store (Redis) so limits are enforced globally.

Return 429 with a `Retry-After` header when limits are exceeded.

---

Q8. What is the difference between a monolithic architecture and a microservices architecture? What are the trade-offs?

A8.
A monolith is a single deployable unit — all features live in one codebase, one process. Microservices split the application into independently deployable services, each owning a specific domain.

Monolith advantages: simpler to develop, test, and deploy initially. No network calls between features — just function calls. No distributed system complexities. A startup with 5 engineers should almost always start monolithic.

Microservices advantages: independent scaling (scale the payment service without scaling the user profile service), independent deployments (ship a fix to one service without redeploying everything), technology flexibility (each service can use a different language or database), and fault isolation (one service crashing doesn't take everything down).

Microservices costs: network latency between services, distributed transactions are hard, debugging across services requires distributed tracing, data consistency challenges, and significant operational overhead (you need CI/CD per service, service discovery, monitoring).

The common pattern: start monolithic, split into microservices only when specific pain points justify the complexity — like a team that can't deploy independently because of coupling, or a component that needs to scale independently.

---

Q9. What is an API gateway, and what problems does it solve in a microservices environment?

A9.
An API gateway sits between clients and your microservices, acting as a single entry point. Instead of the client knowing about 20 different service URLs, it talks to one gateway.

It handles cross-cutting concerns:
- **Routing**: maps `/api/users/*` to the user service, `/api/orders/*` to the order service.
- **Authentication**: validates tokens once at the gateway, so individual services don't each need auth logic.
- **Rate limiting**: applies limits before requests reach services.
- **Load balancing**: distributes traffic across service instances.
- **Request aggregation**: combines responses from multiple services into one response for the client (BFF pattern — Backend For Frontend).

Examples: Kong, AWS API Gateway, NGINX, Zuul.

The trade-off: the gateway becomes a single point of failure and a potential bottleneck. It needs to be highly available and fast. Also, it adds latency — every request goes through an extra hop.

---

Q10. What is a message queue (e.g., RabbitMQ, SQS), and when should you use asynchronous messaging instead of synchronous HTTP?

A10.
A message queue is a buffer between producers and consumers. The producer sends a message to the queue and moves on. The consumer picks it up and processes it independently. If the consumer is down, messages wait in the queue.

Use async messaging when:
- **The work is slow**: Sending an email, generating a PDF, processing a video — don't make the user wait for this. Accept the request, queue the work, return immediately.
- **You need durability**: If the downstream service is temporarily down, messages persist in the queue. With synchronous HTTP, the request fails.
- **You need to decouple services**: The order service shouldn't know or care about the notification service. It just publishes an "order placed" event.
- **You need to smooth traffic spikes**: A queue absorbs bursts — instead of 10,000 simultaneous requests hitting a service, they're processed at a sustainable rate.

Stick with synchronous HTTP when you need an immediate response and the operation is fast — like looking up a user profile.

---

Q11. How would you design a retry mechanism with exponential backoff and jitter?

A11.
Retry with exponential backoff means increasing the wait time between retries exponentially — 1s, 2s, 4s, 8s, etc. This avoids hammering a struggling service.

Jitter adds randomness to the wait time — instead of all retrying clients waiting exactly 4 seconds and then all hitting the service simultaneously (thundering herd), each waits a random time between 0 and 4 seconds.

A practical implementation:

```
wait_time = min(base_delay * (2 ** attempt) + random(0, base_delay), max_delay)
```

Key design decisions:
- **Max retries**: Cap at 3-5 attempts. Infinite retries can pile up.
- **Max delay**: Cap the exponential growth (e.g., 30s max). You don't want a 16-minute wait.
- **Retry only on transient errors**: Retry 500, 502, 503. Don't retry 400, 401, 404 — those aren't going to change.
- **Idempotency**: Only retry operations that are safe to repeat (GET, PUT) or that have idempotency keys.

AWS SDKs, gRPC, and most HTTP client libraries have this built in.

---

Q12. What is the circuit breaker pattern, and why is it critical in distributed systems?

A12.
The circuit breaker prevents cascading failures. When a downstream service is failing, the circuit breaker stops sending it requests, letting it recover instead of overwhelming it.

It has three states:
- **Closed**: Everything normal. Requests pass through. Failures are counted.
- **Open**: Failure threshold exceeded. All requests are immediately rejected (fail fast) without hitting the downstream service. Returns a fallback or error.
- **Half-open**: After a timeout, the circuit lets a few test requests through. If they succeed, it moves back to Closed. If they fail, it goes back to Open.

Without a circuit breaker: Service A calls Service B, which is dying. Service A's threads pile up waiting for B's timeouts. Now Service A becomes slow. Service C calls Service A, and now C is slow too. This cascades through your entire system.

With a circuit breaker: Service A detects B is failing, fails fast, and returns a degraded response or cached data. The rest of the system stays healthy.

Libraries like Hystrix (Java, now deprecated but conceptually important), resilience4j, and Polly (.NET) implement this.

---

### MEDIUM PRIORITY

---

Q13. What is the difference between GraphQL and REST? When would you choose one over the other?

A13.
REST has multiple endpoints — one per resource. `/users`, `/users/123/orders`, etc. The server decides what data to return. GraphQL has a single endpoint — the client specifies exactly what data it needs in the query.

GraphQL solves the over-fetching and under-fetching problems. With REST, `/users/123` might return 50 fields when you only need `name` and `email`. Or you might need data from `/users/123` AND `/users/123/orders`, requiring two round trips. With GraphQL, one query gets exactly what you need.

Choose REST when: you have a simple CRUD API, your clients are mostly server-side (less concerned about payload size), you want strong HTTP caching, or your team is more familiar with REST.

Choose GraphQL when: you have multiple clients (mobile, web, third-party) that need different views of the same data, you want to reduce network requests (especially on mobile), or you have deeply interconnected data.

Trade-offs of GraphQL: more complex server implementation, harder to cache (POST requests to a single endpoint), potential for expensive deeply-nested queries (need query depth limiting), and N+1 problems if your resolvers aren't optimized.

---

Q14. What is gRPC, and what advantages does it have over REST for service-to-service communication?

A14.
gRPC is a high-performance RPC framework by Google. It uses Protocol Buffers (Protobuf) for serialization and HTTP/2 for transport.

Advantages over REST for internal communication:
- **Performance**: Protobuf binary serialization is 5-10x smaller and faster than JSON. HTTP/2 enables multiplexing (multiple requests over one connection) and header compression.
- **Strong typing**: You define your API in a `.proto` file, and code is generated for any language. The contract is explicit and enforced.
- **Streaming**: gRPC supports server streaming, client streaming, and bidirectional streaming natively. REST doesn't — you'd need WebSockets or SSE.
- **Code generation**: Client and server stubs are auto-generated. No manually writing HTTP clients.

Downsides: not browser-friendly (you need gRPC-Web proxy), harder to debug (binary payloads aren't human-readable like JSON), and less tooling support compared to REST.

Use REST for public APIs. Use gRPC for internal microservice-to-microservice communication where performance and type safety matter.

---

Q15. What is the SAGA pattern, and how does it handle distributed transactions across microservices?

A15.
In a monolith, you wrap operations in a database transaction — all succeed or all roll back. In microservices, each service has its own database, so you can't use a single transaction across services.

The SAGA pattern breaks a distributed transaction into a sequence of local transactions. Each service executes its local transaction and publishes an event. If one step fails, the saga executes compensating transactions to undo previous steps.

Two styles:
- **Choreography**: Services react to events autonomously. Order service emits "OrderCreated," payment service listens and processes payment, inventory service listens and reserves stock. If payment fails, it emits "PaymentFailed," and inventory releases the stock.
- **Orchestration**: A central orchestrator (saga manager) tells each service what to do and handles compensations. More control, clearer flow, but the orchestrator can become a bottleneck.

The hard part is designing compensating actions. You can't un-send an email or un-charge a credit card easily. You have to design for reversal from the start — like marking an order as "cancelled" instead of deleting it, or issuing a refund instead of reversing the charge.

---

Q16. What is CQRS (Command Query Responsibility Segregation), and when would you apply it?

A16.
CQRS separates read operations (queries) from write operations (commands) into different models — potentially different databases. Instead of one model doing both reads and writes, you optimize each independently.

The write model is normalized and optimized for consistency and validation. The read model is denormalized and optimized for query performance — maybe backed by Elasticsearch for search, or a materialized view for dashboards.

Apply it when: read and write patterns are dramatically different (e.g., 95% reads, highly complex queries vs. simple writes), you need to scale reads and writes independently, or your read and write models need fundamentally different shapes.

Don't apply it when: your application is simple CRUD. CQRS adds significant complexity — eventual consistency between the write and read stores, synchronization logic, and more infrastructure. For most applications, a single well-indexed database is the right answer.

---

Q17. What is event sourcing, and how does it differ from traditional CRUD persistence?

A17.
In traditional CRUD, you store the current state. Update a user's email? You overwrite the old email with the new one. The history is gone.

In event sourcing, you store every state change as an immutable event — "UserCreated," "EmailChanged," "AccountSuspended." The current state is derived by replaying all events from the beginning. You never update or delete events.

Benefits: complete audit trail (you know exactly what happened and when), ability to rebuild state at any point in time, and events naturally integrate with event-driven architectures.

Costs: replaying events for current state can be slow (snapshots help), querying current state is awkward (you often pair it with CQRS), eventual consistency, and the event schema needs careful versioning.

Use it when: you need a full audit log (financial systems, compliance), temporal queries ("what was the account balance on March 15?"), or event-driven microservices where events are already your communication mechanism.

---

Q18. How do you handle versioning of APIs without breaking existing clients?

A18.
There are several approaches, each with trade-offs:

**URL versioning**: `/api/v1/users`, `/api/v2/users`. Simple, explicit, easy to route. But purists argue it violates REST (the resource hasn't changed, the representation has).

**Header versioning**: `Accept: application/vnd.myapi.v2+json` or custom `API-Version: 2` header. Cleaner URLs but harder to test in a browser.

**Query parameter**: `/api/users?version=2`. Easy to implement but feels hacky.

My preference: URL versioning for major breaking changes, and design APIs to be backward compatible as much as possible. Adding new fields to a response is backward compatible. Removing or renaming fields is not — that's a new version.

Key practices: give clients a generous deprecation timeline (6-12 months), monitor which versions are still in use, and document migration guides. Never introduce a new version without a strong reason.

---

Q19. How would you design a webhook system? What reliability guarantees would you provide?

A19.
A webhook is a "don't call us, we'll call you" pattern. Instead of the consumer polling for updates, your system sends an HTTP POST to the consumer's URL when an event occurs.

Design considerations:
- **Retry with backoff**: If the delivery fails (timeout, 5xx), retry with exponential backoff — 1 min, 5 min, 30 min, 1 hour. Cap at ~5 attempts.
- **Idempotency**: Include an event ID so consumers can deduplicate. Deliver at-least-once.
- **Signature verification**: Sign the payload with HMAC-SHA256 using a shared secret. The consumer verifies the signature to ensure the webhook is authentic and wasn't tampered with.
- **Timeout**: Don't wait forever for the consumer's response. Set a 5-10 second timeout.
- **Dead letter queue**: After max retries, move the event to a DLQ for manual inspection.
- **Event log**: Let consumers query a `/events` endpoint to fetch missed events — webhooks are a convenience, not the only way to get data.

Stripe's webhook system is a great reference for this pattern.

---

Q20. What is a sidecar pattern, and how is it used in service meshes like Istio?

A20.
The sidecar pattern deploys a helper process alongside your main application — they run in the same pod (in Kubernetes) but in separate containers. The sidecar handles cross-cutting concerns so your application doesn't have to.

In a service mesh like Istio, the sidecar is Envoy proxy. It intercepts all incoming and outgoing network traffic from your service and handles: mTLS encryption, load balancing, circuit breaking, retry policies, metrics collection, distributed tracing, and traffic routing.

The benefit: your application code is purely business logic. It doesn't need libraries for retries, circuit breakers, or TLS. The sidecar handles it transparently. And the mesh provides a consistent policy layer across all services regardless of language.

The cost: added latency (every request goes through the proxy), increased resource consumption (each pod now has an extra container), and operational complexity of managing the mesh itself.

---

Q21. What is structured logging, and why is it preferable to unstructured log lines?

A21.
Unstructured logging: `"User 123 placed order 456 for $99.50"` — human-readable but hard to parse programmatically.

Structured logging: `{"user_id": 123, "order_id": 456, "amount": 99.50, "event": "order_placed"}` — machine-parseable, queryable, and filterable.

With structured logs, you can:
- Query all logs for `user_id=123` across all services.
- Alert on `event=order_failed AND amount > 1000`.
- Build dashboards aggregating by any field.
- Feed logs into Elasticsearch/Splunk and run complex analysis.

With unstructured logs, you'd need regex parsing, which is fragile and slow. When you have hundreds of services generating millions of log lines, structured logging is the difference between debugging in minutes vs. hours.

Use logging libraries that support structured output: Python's `structlog`, Java's Logback with JSON encoder, or Go's `zap`.

---

Q22. What is distributed tracing, and how do tools like Jaeger or OpenTelemetry help debug latency issues?

A22.
In a microservices world, a single user request might touch 10 services. If the response is slow, which service is the bottleneck? Distributed tracing answers this.

Each request gets a unique trace ID. As it flows through services, each service creates a span — a record of the work it did, how long it took, and which service it called next. The spans form a tree that shows the complete request lifecycle.

OpenTelemetry is the standard for instrumenting your code — it generates traces, metrics, and logs. Jaeger, Zipkin, and cloud-native tools (AWS X-Ray, Datadog APM) are backends that collect, store, and visualize traces.

When debugging latency, you look at the trace waterfall: "The total request took 2 seconds. The user service took 50ms, the inventory service took 100ms, but the payment service took 1.8s — that's the bottleneck." You can even see if the payment service was waiting on its own downstream dependency.

Without distributed tracing, debugging latency across microservices is essentially guesswork.

---

### LOW PRIORITY

---

Q23. What is the bulkhead pattern, and how does it isolate failures in a microservices system?

A23.
Named after watertight compartments in a ship — if one compartment floods, the others remain sealed. In software, it means isolating resources so that one failing component doesn't consume all resources and bring everything down.

For example: your service calls three external APIs. Without bulkheads, all calls share the same thread pool. If one external API becomes very slow, it eats all threads, and now calls to the other two healthy APIs can't get threads either. Everything fails.

With bulkheads: each external API gets its own thread pool (or connection pool). If one API's pool is exhausted, the others are unaffected. Alternatively, you can use separate processes or containers for different concerns.

It's about limiting the blast radius. Netflix popularized this pattern through their Hystrix library — thread pool isolation per dependency.

---

Q24. What is the outbox pattern, and how does it ensure reliable event publishing alongside database writes?

A24.
The problem: you want to save data to your database AND publish an event to a message queue, atomically. If the DB write succeeds but publishing fails, your system is inconsistent. If publishing succeeds but the DB write fails, other services react to something that didn't happen.

The outbox pattern: instead of publishing directly, you write the event to an "outbox" table in the same database, in the same transaction as your business data. A separate process (CDC-based or polling) reads the outbox table and publishes events to the message queue.

Since the business data and the outbox entry are in the same transaction, they either both commit or both roll back. The publisher process guarantees at-least-once delivery by tracking what's been published.

Debezium (CDC-based) is the most popular tool for this pattern with relational databases.

---

Q25. How do you handle request deduplication in an event-driven backend?

A25.
In at-least-once delivery systems (which is most message queues), the same event can be delivered multiple times. You need to make your consumers idempotent.

The standard approach: each event has a unique ID. The consumer stores processed event IDs (in Redis or a database table). Before processing, check: "Have I seen this ID?" If yes, skip it.

For database operations, you can also use upserts or conditional inserts (`INSERT ... ON CONFLICT DO NOTHING`) so processing the same event twice produces the same result.

The event ID storage needs a retention policy — you can't keep IDs forever. 24-72 hours covers most retry scenarios. If your system has longer replay needs, you might need a different deduplication strategy.

---

Q26. What is a service registry, and how does service discovery work in a container-orchestrated environment?

A26.
A service registry is a directory where services register themselves with their network locations (IP, port). Other services query the registry to find where to send requests.

In Kubernetes, this is built-in via DNS-based service discovery. When you create a Service object, Kubernetes assigns it a stable DNS name (e.g., `payment-service.default.svc.cluster.local`). Any pod can resolve this name to reach the service. Kube-proxy handles the actual load balancing.

Without Kubernetes, tools like Consul, Eureka, or etcd serve as registries. Services register on startup and de-register on shutdown. Health checks ensure dead instances are removed.

Client-side discovery: the client queries the registry and decides which instance to call. Server-side discovery: the client sends to a load balancer, which queries the registry. Kubernetes uses server-side discovery.

---

Q27. What is blue-green deployment vs. canary deployment? When would you use each?

A27.
**Blue-green**: You maintain two identical production environments. Blue is live, green is idle. Deploy the new version to green, test it, then switch all traffic from blue to green. If something's wrong, switch back instantly.

**Canary**: Roll out the new version to a small percentage of traffic (1%, 5%, 10%...) and gradually increase. Monitor metrics at each stage. If something breaks, only a fraction of users are affected, and you roll back.

Use blue-green when: you want instant, all-or-nothing cutover with instant rollback. It's simpler but requires double the infrastructure.

Use canary when: you want to limit blast radius and catch issues that only appear at scale or with real traffic patterns. It's more sophisticated and requires good monitoring and traffic splitting (feature flags or load balancer configuration).

In practice, canary is more common for large-scale services because it catches issues that testing can't — performance regressions, edge cases in specific regions, etc.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q28. What is Docker, and how does containerization differ from virtualization? Why is it essential for modern backends?

A28.
Docker packages an application and ALL its dependencies (runtime, libraries, system tools, config) into a container — a lightweight, standalone, executable unit. The key insight: it works the same everywhere — your laptop, staging, production.

**Containers vs VMs**: A VM emulates an entire operating system — each VM has its own kernel, OS, and full stack. Heavy (GBs), slow to start (minutes). A container shares the host OS kernel and isolates only the application layer — using Linux namespaces (process isolation) and cgroups (resource limits). Light (MBs), starts in seconds.

**Why Docker matters for backends**:
- **Consistency**: "Works on my machine" is eliminated. The container IS the deployment artifact.
- **Isolation**: Each service runs in its own container with its own dependencies. No version conflicts.
- **Scaling**: Spin up 100 containers in seconds. Kubernetes orchestrates this automatically.
- **CI/CD**: Build once, run anywhere. The same container image goes through test → staging → production.

**Dockerfile basics**: `FROM python:3.11 → COPY . /app → RUN pip install -r requirements.txt → CMD ["python", "app.py"]`. Each instruction creates a layer. Layers are cached — rebuilds are fast.

---

Q29. What is Kubernetes at a high level? What problems does it solve for backend services?

A29.
Kubernetes (K8s) is a container orchestration platform. You tell it "I want 5 replicas of this service, always running" and it makes that happen — deploying containers, restarting crashed ones, scaling up/down, and load balancing.

**Core concepts**:
- **Pod**: The smallest deployable unit — one or more containers that share network and storage. Usually one container per pod.
- **Deployment**: Declares desired state — "run 3 replicas of my API server using this container image." K8s ensures the actual state matches.
- **Service**: A stable network endpoint. Pods are ephemeral (IP changes on restart). A Service provides a stable DNS name and load balances across healthy pods.
- **Ingress**: Routes external HTTP traffic to internal Services based on URL paths or hostnames.

**Problems it solves**: Automated rollouts and rollbacks, self-healing (restarts failed containers), horizontal autoscaling (scale based on CPU/memory/custom metrics), secret management, service discovery, resource quotas and limits.

**When NOT to use K8s**: Simple applications with 1-3 services. The operational overhead of K8s is significant — consider managed services (AWS ECS, Cloud Run, Heroku) for simpler deployments.

---

Q30. What is a connection pool, and why is it critical for database-backed services?

A30.
A connection pool maintains a set of pre-established database connections that are reused across requests, rather than opening and closing a connection for each query.

**Why it's critical**: Opening a database connection is expensive — TCP handshake, TLS negotiation, authentication, allocating memory on the database server. A typical connection takes 20-50ms to establish. At 1000 requests/second, creating a new connection per request would mean 1000 simultaneous connection setups — the database would be overwhelmed.

**How it works**: The pool maintains N idle connections (e.g., 10-50). When application code needs a connection, it borrows one from the pool. When done, it returns it. If all connections are in use, the request waits (with a timeout) or a new connection is created (up to a maximum).

**Key settings**: `min_idle` (minimum warm connections), `max_pool_size` (maximum total), `connection_timeout` (how long to wait for a connection), `idle_timeout` (close connections idle too long), `max_lifetime` (recycle connections to handle server-side limits).

**Tools**: HikariCP (Java — fastest, default in Spring Boot), pgBouncer (PostgreSQL external pooler), SQLAlchemy pool (Python), Prisma (Node.js).

**Common mistake**: Setting pool size too high. Each connection uses ~10MB on the database server. 100 app instances × 50 connections = 5000 connections. PostgreSQL defaults to max 100 connections. Use pgBouncer for connection multiplexing.

---

Q31. What is Redis, and what are its common usage patterns beyond caching?

A31.
Redis is an in-memory data structure store — it's not just a key-value cache, it's a toolkit for building distributed systems.

**Beyond basic caching**:
- **Rate limiting**: Use `INCR` + `EXPIRE` to count requests per time window. O(1) per request.
- **Distributed locks**: `SET key value NX EX 30` — set only if not exists, with 30s expiry. Used for ensuring exactly one worker processes a job. Redlock algorithm for multi-node setups.
- **Pub/Sub**: Publish messages to channels, subscribers receive them in real-time. Lightweight alternative to Kafka for simple real-time notifications.
- **Leaderboards**: Sorted sets (`ZADD`, `ZRANGEBYSCORE`). Add a user's score, get top-N, get rank — all O(log n).
- **Session storage**: Fast reads, automatic TTL-based expiry. Better than database sessions for high-traffic apps.
- **Queue**: Lists as queues (`LPUSH`/`BRPOP`). Simple job queue without the complexity of RabbitMQ/Kafka.
- **Geospatial**: `GEOADD`, `GEORADIUS` — find nearby locations within a radius.

**Persistence options**: RDB (periodic snapshots) or AOF (append-only log of every write). Or both. Redis is not purely ephemeral.

**Trade-off**: Everything in memory — fast but expensive. A 100GB Redis instance costs significantly more than a 100GB PostgreSQL instance. Use Redis for hot data, database for cold data.

---

### IMPORTANT

---

Q32. What is a reverse proxy (e.g., Nginx)? How does it differ from a forward proxy?

A32.
**Forward proxy**: Sits between clients and the internet. The client sends requests to the proxy, which forwards them to the target server. Used for: privacy (hide client IP), content filtering (block sites), caching. The client knows it's using a proxy.

**Reverse proxy**: Sits between the internet and your backend servers. The client sends requests to the reverse proxy (thinks it's the real server), which forwards them to the appropriate backend. Used for: load balancing, SSL termination, caching, compression, security (hide backend topology).

**Nginx as reverse proxy**: Clients hit `api.example.com` → Nginx receives the request → forwards to one of several backend servers (localhost:8001, :8002, :8003). Nginx handles SSL, serves static files, compresses responses, and distributes load.

**Key difference**: Forward proxy acts on behalf of the client. Reverse proxy acts on behalf of the server. The client doesn't know a reverse proxy exists.

---

Q33. What is CI/CD? How does a typical deployment pipeline look for a backend service?

A33.
**CI (Continuous Integration)**: Developers merge code to the main branch frequently. Each merge triggers automated builds and tests. Catches integration issues early — no more "it works on my machine."

**CD (Continuous Delivery/Deployment)**: Delivery = automated pipeline to a staging environment, manual approval for production. Deployment = fully automated to production — every passing commit goes live.

**Typical pipeline**:
1. **Commit**: Developer pushes to a branch, opens a PR.
2. **Build**: Compile code, build Docker image, resolve dependencies.
3. **Test**: Unit tests → integration tests → end-to-end tests. If any fail, block the merge.
4. **Static analysis**: Linting, security scanning (SAST), dependency vulnerability checking.
5. **Artifact publish**: Push Docker image to registry (ECR, Docker Hub).
6. **Deploy to staging**: Automatically deploy the image. Run smoke tests.
7. **Deploy to production**: Canary deployment (5% → 25% → 100%). Monitor error rates, latency, and business metrics at each stage.
8. **Rollback if needed**: Automatic rollback if error rate exceeds threshold.

**Tools**: GitHub Actions, GitLab CI, Jenkins, CircleCI, ArgoCD (for GitOps-based K8s deployments).

---

Q34. How do you handle database migrations in a zero-downtime deployment?

A34.
Zero-downtime migrations require that old code and new code can run simultaneously against the database during the rollout.

**Safe migration strategy (expand and contract)**:
1. **Expand**: Add new columns/tables. Don't remove or rename anything. Both old and new code work. Deploy new code that writes to both old and new columns.
2. **Migrate data**: Backfill existing data to the new columns.
3. **Contract**: Once all code uses the new schema, remove old columns in a later deploy.

**Rules for zero-downtime migrations**:
- ✅ Add a nullable column (old code ignores it).
- ✅ Add a new table (old code doesn't know about it).
- ❌ Remove a column (old code that references it will crash).
- ❌ Rename a column (effectively remove + add).
- ❌ Add a NOT NULL column without a default (inserts from old code fail).

**Large table migrations**: `ALTER TABLE` can lock a table for minutes/hours. Use online schema migration tools — gh-ost (GitHub), pt-online-schema-change (Percona) — they create a shadow table, copy data, and swap atomically.

**Tools**: Flyway, Liquibase (Java), Alembic (Python), Django Migrations, Prisma Migrate.

---

Q35. What is the difference between thread-pool and event-driven architectures for handling concurrent requests?

A35.
**Thread-per-request (thread pool)**: Each incoming request gets its own thread. The thread handles the request start to finish, including blocking I/O (waiting for DB, external API). Simple programming model — synchronous code. But each thread uses ~1MB of memory. 1000 concurrent requests = 1000 threads = 1GB just for stacks. Beyond ~10K concurrent connections, thread overhead becomes a bottleneck.

**Event-driven (async/non-blocking)**: A small number of threads (often 1-4) handle all requests via an event loop. When a request needs I/O, it registers a callback and the thread moves to the next request. When I/O completes, the callback fires. Handles 100K+ concurrent connections efficiently.

**Examples**: Thread pool — Java Spring (traditional), Python Django/Flask with Gunicorn workers. Event-driven — Node.js, Nginx, Python asyncio/FastAPI, Java Netty/Spring WebFlux.

**Trade-off**: Thread pool is simpler to code and debug (sequential logic). Event-driven handles more connections with less memory but is harder to reason about — callback hell, unhandled promise rejections, harder debugging.

**Modern hybrid**: Go's goroutines — lightweight threads (2KB initial stack) managed by Go's runtime scheduler. You write synchronous-looking code but get event-loop-level concurrency. Handles millions of concurrent operations.

---

Q36. What are health checks (liveness vs readiness probes)? How are they used in container orchestration?

A36.
**Liveness probe**: "Is the process alive and not stuck?" If it fails, K8s restarts the container. Catches deadlocks, infinite loops, or memory corruption that leaves the process running but non-functional.

**Readiness probe**: "Is the service ready to accept traffic?" If it fails, K8s removes the pod from the load balancer — no traffic is sent. Catches situations like: the app is starting up and hasn't loaded its cache, or it's temporarily overloaded and needs time to recover.

**Implementation**: Usually an HTTP endpoint (`GET /health` → 200 OK, `GET /ready` → 200 OK). Liveness checks should be simple (process is alive). Readiness checks can verify dependencies (database connection OK, cache loaded).

**Common mistake**: Making liveness checks depend on external services. If your database goes down, your liveness check fails, K8s restarts your pods — but they still can't connect to the DB, so they fail again. Cascading restart loop. Liveness = "am I functioning?" Readiness = "can I serve requests?"

---

### GOOD-TO-HAVE

---

Q37. What is the 12-Factor App methodology? Which factors are most impactful?

A37.
12-Factor is a set of principles for building SaaS applications that are portable, scalable, and maintainable. Written by Heroku engineers.

**Most impactful factors**:
- **Config in environment variables**: No hardcoded config. Database URLs, API keys, feature flags — all in env vars. Different environments (dev/staging/prod) differ only in config.
- **Stateless processes**: Application instances should be stateless. Sessions, caches, and state go in external stores (Redis, database). Any instance can handle any request — makes horizontal scaling trivial.
- **Port binding**: The app exports HTTP by binding to a port. No external web server needed as a dependency.
- **Disposability**: Fast startup, graceful shutdown. Processes can be killed and restarted at any time. Enables scaling, deploys, and crash recovery.
- **Dev/prod parity**: Keep environments as similar as possible. Use the same database, message queue, and tools in dev and production. Reduces "works on my machine" bugs.
- **Logs as event streams**: Write logs to stdout. Let the platform (K8s, Cloud Run) collect and route them. Don't manage log files in your app.

---

Q38. What is feature flagging, and how do you use it for safe deployments?

A38.
Feature flags are runtime toggles that enable or disable features without deploying new code. You deploy the code to production with the feature behind a flag, then gradually enable it.

**Use cases**: Canary releases (enable for 1% of users, monitor, increase). A/B testing (variant A vs variant B). Kill switch (disable a problematic feature instantly without a rollback). Dark launches (deploy code without enabling the feature for anyone — test that it doesn't break existing functionality).

**Implementation**: Simple flag — `if feature_enabled("new_checkout"): show_new_checkout() else: show_old_checkout()`. Backed by a flag service (LaunchDarkly, Unleash, Flagsmith) or a config file/database.

**Important**: Clean up flags after rollout. Tech debt accumulates if you leave flags in code forever — you end up with `if flag_A and not flag_B or (flag_C and user.is_premium)` branching everywhere.

---

Q39. What is backpressure, and how do you handle it in high-throughput systems?

A39.
Backpressure occurs when a component produces data faster than the downstream consumer can process it. Without handling it, the buffer between them grows unbounded → out of memory → crash.

**Strategies**:
- **Buffering with limits**: Accept data into a bounded buffer. When full, reject new data (HTTP 429, Kafka produces error). The producer slows down.
- **Dropping**: Drop oldest or newest messages when overwhelmed. Acceptable for metrics/logging — you can lose some data points.
- **Flow control**: Signal the producer to slow down. TCP does this natively (window size). Reactive Streams (Java) and gRPC flow control implement this for application-level protocols.
- **Throttling**: Rate-limit the producer — only accept N messages per second.
- **Scaling consumers**: Auto-scale consumers based on queue depth. If the Kafka consumer group lag grows, add more consumers.

**Real-world example**: A logging pipeline. App servers produce 100K log events/second. Logstash can process 50K/second. Without backpressure, the buffer fills and either crashes Logstash or drops logs silently. Solution: bounded queue in Logstash + producers handle rejections gracefully.