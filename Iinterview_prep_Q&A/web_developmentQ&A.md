==============================
FILE: Web Development (Backend-Focused)
==============================

### HIGH PRIORITY

---

Q1. What is the difference between a Single Page Application (SPA) and a Multi-Page Application (MPA)?

A1.
**SPA**: The browser loads a single HTML page, and JavaScript dynamically updates the content without full page reloads. Navigating between "pages" just swaps out components — the URL changes via the History API, but no new HTML is fetched from the server.

**MPA**: Every navigation request goes to the server, which returns a full new HTML page. Each page is a separate server response.

**SPA advantages**: Fast, app-like feel after initial load. Smooth transitions. The frontend handles routing (React Router, Vue Router). Good for interactive applications (Gmail, Slack).

**SPA disadvantages**: Larger initial bundle (all JavaScript upfront). SEO is harder — search engines see an empty HTML page until JS executes. Initial load is slower.

**MPA advantages**: Better SEO (each page is a complete HTML document). Simpler architecture. Faster first paint — the server sends ready-to-display HTML.

**Modern compromise**: Server-Side Rendering (SSR) — render the SPA on the server for the first load (good SEO, fast first paint), then hydrate on the client for SPA-like interactivity. Next.js, Nuxt.js do this. Or Static Site Generation (SSG) — pre-render pages at build time.

---

Q2. What is CORS (Cross-Origin Resource Sharing)? Why does it exist?

A2.
CORS is a browser security mechanism that restricts web pages from making requests to a different origin (domain, port, or protocol) than the one that served the page.

**Why it exists**: Without CORS, a malicious site could make API requests to your bank's API using your cookies — the Same-Origin Policy prevents this. If `evil.com` tries to fetch `yourbank.com/api/balance`, the browser blocks it.

**How CORS works**: The browser sends an `Origin` header with the request. The server responds with `Access-Control-Allow-Origin` headers specifying which origins are allowed. If the requesting origin isn't allowed, the browser blocks the response.

**Preflight requests**: For "non-simple" requests (PUT, DELETE, custom headers), the browser first sends an OPTIONS request asking "is this allowed?" The server responds with allowed methods and headers. If approved, the actual request proceeds.

**Common configuration** (backend): Set `Access-Control-Allow-Origin: https://yourdomain.com` (specific origin) or `*` (any origin — use only for public APIs). Also set `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and `Access-Control-Allow-Credentials` as needed.

**Common mistake**: Setting `Allow-Origin: *` with `Allow-Credentials: true` — browsers reject this combination to prevent credential leaking to arbitrary origins.

---

Q3. How does web authentication work? Compare cookies/sessions vs. JWT.

A3.
**Cookie/Session-based auth**:
1. User logs in → server creates a session (stored in memory or database), generates a session ID.
2. Server sends the session ID in a `Set-Cookie` header.
3. Browser automatically includes the cookie in every subsequent request to that domain.
4. Server looks up the session ID → finds the user.

**Pros**: Server controls sessions — can invalidate instantly (logout). Secure when using `HttpOnly`, `Secure`, `SameSite` cookie flags.
**Cons**: Server must store sessions (memory/DB). Harder to scale horizontally (sticky sessions or shared session store needed).

**JWT (JSON Web Token) auth**:
1. User logs in → server creates a JWT containing user claims, signs it with a secret key.
2. Server sends the JWT to the client (stored in localStorage or HttpOnly cookie).
3. Client includes the JWT in the `Authorization: Bearer <token>` header.
4. Server verifies the signature — no session lookup needed.

**Pros**: Stateless — any server can verify the token. Scales easily.
**Cons**: Can't easily revoke (no central sessions to delete). Token size is larger. If stolen, valid until expiration.

**Best practice**: Use short-lived JWTs (15 min) + refresh tokens (stored securely, longer-lived). Refresh tokens can be revoked. Store JWTs in HttpOnly cookies, not localStorage (XSS protection).

---

Q4. What makes a good REST API design?

A4.
**Resources as nouns**: URLs represent resources — `/users`, `/orders/123`, not `/getUser` or `/createOrder`. HTTP methods define the action.

**HTTP methods correctly**:
- GET: Read (idempotent, safe).
- POST: Create (not idempotent).
- PUT: Full replace (idempotent).
- PATCH: Partial update (idempotent).
- DELETE: Remove (idempotent).

**Status codes correctly**: 200 (OK), 201 (Created), 204 (No Content — successful delete), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict), 422 (Unprocessable Entity), 429 (Too Many Requests), 500 (Server Error).

**Versioning**: `/api/v1/users` — allows breaking changes without affecting existing clients.

**Pagination**: Don't return thousands of records. Use `?page=2&limit=20` or cursor-based pagination (`?cursor=abc123`) for large datasets. Include total count and next/prev links.

**Filtering and sorting**: `?status=active&sort=-created_at` (- prefix for descending).

**Error responses**: Consistent format — `{ "error": { "code": "VALIDATION_ERROR", "message": "Email is required", "details": [...] } }`.

**HATEOAS** (optional): Include links to related resources in responses — `{ "user": { "id": 1, "orders_url": "/users/1/orders" } }`.

---

Q5. What is WebSocket? How does it differ from HTTP?

A5.
**HTTP**: Request-response model. The client sends a request, the server responds, the connection closes (or stays open for keep-alive, but is still request-response). The server cannot push data to the client unprompted.

**WebSocket**: Full-duplex, persistent connection. After an HTTP handshake (upgrade request), the connection switches to the WebSocket protocol. Both client and server can send messages at any time without waiting for a request.

**When to use WebSocket**:
- Real-time chat applications.
- Live dashboards with streaming data.
- Multiplayer games.
- Collaborative editing (Google Docs-style).
- Stock price tickers, live sports scores.

**When HTTP is sufficient**:
- CRUD operations.
- Form submissions.
- REST APIs where the client initiates all interactions.
- When polling every few seconds is acceptable.

**Alternatives to WebSocket**:
- **Server-Sent Events (SSE)**: Server pushes to client (one-way). Simpler than WebSocket. Good for notifications, live feeds. Uses HTTP, so works through proxies easily.
- **Long polling**: Client makes a request; server holds it until new data is available. Compatible everywhere, but resource-intensive.

**WebSocket at scale**: Need sticky sessions (or a pub/sub layer like Redis). More complex to load-balance than HTTP. Handle reconnection logic client-side.

---

Q6. What are CSRF and XSS attacks? How do you prevent them?

A6.
**XSS (Cross-Site Scripting)**: An attacker injects malicious JavaScript into your website. When other users load the page, the script executes in their browser — stealing cookies, session tokens, or performing actions as the user.

Types: Stored XSS (script saved in DB, served to all users), Reflected XSS (script in URL parameter, reflected in response), DOM-based XSS (client-side JS unsafely manipulates the DOM).

**Prevention**: Sanitize/escape all user input before rendering (HTML entity encoding). Use Content Security Policy (CSP) headers. Use HttpOnly cookies (JavaScript can't access them). React/Vue auto-escape by default — but `dangerouslySetInnerHTML` (React) bypasses this.

**CSRF (Cross-Site Request Forgery)**: An attacker tricks an authenticated user into making an unwanted request to a site where they're logged in. Example: A hidden form on `evil.com` submits a money transfer request to `yourbank.com` — the browser includes the user's session cookies automatically.

**Prevention**: CSRF tokens — include a random token in forms; the server verifies it matches. SameSite cookie attribute (`Strict` or `Lax`) — browser won't send cookies with cross-origin requests. Check the `Origin` or `Referer` header. For APIs, require custom headers (like `X-Requested-With`) — browsers don't allow cross-origin custom headers without CORS approval.

---

Q7. What is OAuth 2.0? Explain the authorization code flow.

A7.
OAuth 2.0 is an authorization framework that lets a third-party application access a user's resources without getting their password. "Log in with Google" uses OAuth.

**Authorization Code Flow** (most secure, for server-side apps):
1. User clicks "Login with Google" → app redirects to Google's authorization endpoint with `client_id`, `redirect_uri`, `scope`, and `response_type=code`.
2. User authenticates with Google and grants permission.
3. Google redirects back to your app's `redirect_uri` with an authorization code.
4. Your server exchanges the code for tokens by calling Google's token endpoint with the code + `client_secret` (server-to-server — the secret never reaches the browser).
5. Google returns an access token (and optionally a refresh token).
6. Your app uses the access token to call Google's APIs on behalf of the user.

**Key distinction**: OAuth is for authorization (access to resources), not authentication (proving identity). OpenID Connect (OIDC) builds on OAuth 2.0 to add authentication — it returns an ID token with user identity information.

**PKCE** (Proof Key for Code Exchange): For SPAs and mobile apps (no client secret). Generates a code_verifier + code_challenge pair. Prevents authorization code interception attacks. Required for public clients.

---

### MEDIUM PRIORITY

---

Q8. What are HTTP caching strategies?

A8.
HTTP caching reduces load and latency by storing responses and reusing them for subsequent identical requests.

**Cache-Control header** (primary mechanism):
- `max-age=3600`: Cache for 1 hour. The browser won't even contact the server during this time.
- `no-cache`: Can cache, but must revalidate with the server before using (sends a conditional request).
- `no-store`: Don't cache at all. For sensitive data.
- `public`: Any cache (CDN, proxy) can store it.
- `private`: Only the browser can cache it (not CDN/proxies). For user-specific data.

**Conditional requests** (revalidation):
- `ETag` + `If-None-Match`: Server returns a hash (ETag) with the response. Next request includes the hash. If unchanged, server returns 304 (Not Modified) — no body, use cached version.
- `Last-Modified` + `If-Modified-Since`: Similar but uses timestamps.

**Strategies by content type**:
- Static assets (JS, CSS, images): Long `max-age` (1 year) + fingerprinted filenames (`app.a1b2c3.js`). When content changes, the filename changes, busting the cache.
- API responses: Short `max-age` or `no-cache` + ETag. Balance freshness with performance.
- User-specific data: `private, no-cache` + ETag.

---

Q9. What is rate limiting? How do you implement it?

A9.
Rate limiting restricts how many requests a client can make in a given time window. Protects against abuse, DoS attacks, and ensures fair resource usage.

**Algorithms**:
- **Fixed window**: Count requests per time window (e.g., 100 per minute). Simple, but burst at window boundaries — 100 at 0:59 + 100 at 1:00 = 200 in 2 seconds.
- **Sliding window**: Smooths out the fixed window problem. Weighted count based on current position in the window.
- **Token bucket**: Bucket holds tokens (capacity = burst limit). Tokens added at a fixed rate. Each request consumes one token. If empty, request is rejected. Allows bursts up to capacity.
- **Leaky bucket**: Requests enter a queue that drains at a fixed rate. Queue full → reject. Smooths traffic to a steady rate.

**Implementation**:
- In-memory (single server): Simple counter with timestamp. Limited to one server.
- Redis (distributed): `INCR` + `EXPIRE` for fixed window. Use Redis because rate limits must be shared across all app servers.
- API Gateway (Kong, AWS API Gateway): Built-in rate limiting. Simplest for production.

**Response**: 429 Too Many Requests with `Retry-After` header. Include `X-RateLimit-Remaining` and `X-RateLimit-Reset` headers for client awareness.

---

Q10. What is server-side rendering (SSR) vs. client-side rendering (CSR)?

A10.
**CSR**: The server sends an empty HTML shell + JavaScript bundle. The browser downloads JS, executes it, fetches data from APIs, and renders the page. First paint is slow (waiting for JS download + execution + API calls). SEO is poor (crawlers see empty HTML).

**SSR**: The server renders the full HTML for each request. The browser receives ready-to-display content immediately. Fast first paint. SEO-friendly (crawlers see complete content). Then the client-side JS "hydrates" — attaches event listeners and makes the page interactive.

**SSG (Static Site Generation)**: Pages are pre-rendered at build time, served as static files. Fastest possible performance (CDN-cached). Great for content that doesn't change per request (blogs, docs). But requires a rebuild for content changes.

**ISR (Incremental Static Regeneration)**: Next.js feature — serve static pages but regenerate them in the background after a set interval. Combines SSG speed with fresh content.

**Choosing**:
- SSR: User-specific content, real-time data, SEO-critical pages (e-commerce product pages).
- CSR: Dashboards, internal tools, apps behind login (SEO doesn't matter).
- SSG: Blogs, documentation, marketing sites.
- ISR: E-commerce catalogs, news sites — mostly static but needs periodic updates.

---

Q11. What are cookies, localStorage, and sessionStorage? When to use each?

A11.
**Cookies**: Sent with every HTTP request to the matching domain. Max 4KB. Has expiration, domain, path, and security flags (HttpOnly, Secure, SameSite). Server can set them. Use for: authentication tokens, session IDs, tracking.

**localStorage**: Stored in the browser, persists indefinitely (until explicitly cleared). Max ~5-10MB. Not sent with HTTP requests. JavaScript only. Use for: user preferences, draft content, non-sensitive cached data.

**sessionStorage**: Same as localStorage but scoped to the browser tab. Cleared when the tab closes. Use for: one-time form data, wizard state, per-tab state.

**Security considerations**:
- Never store sensitive tokens in localStorage — vulnerable to XSS (any JS on the page can read it).
- Use HttpOnly cookies for auth tokens — JavaScript can't access them, only the browser includes them in requests.
- SameSite cookies prevent CSRF by restricting when cookies are sent cross-origin.

**Common pattern**: Auth token in an HttpOnly cookie (secure), user preferences in localStorage (convenient), multi-step form state in sessionStorage (tab-scoped).

---

Q12. What are service workers?

A12.
Service workers are JavaScript scripts that run in the background, separate from the web page. They act as a proxy between the browser and the network, intercepting requests and enabling offline functionality.

**Key capabilities**:
- **Offline support**: Cache critical assets and API responses. When the network is unavailable, serve from cache.
- **Background sync**: Queue actions performed offline and send them when the connection is restored.
- **Push notifications**: Receive push messages from a server even when the page isn't open.
- **Request interception**: Intercept every network request and decide how to handle it — serve from cache, fetch from network, or return a custom response.

**Caching strategies**:
- Cache First: Check cache → serve if available → fall back to network. For static assets.
- Network First: Try network → fall back to cache if offline. For API data.
- Stale While Revalidate: Serve from cache immediately, fetch from network in background to update cache. Best of both — fast and fresh.

**Lifecycle**: Register → Install (pre-cache assets) → Activate (clean old caches) → Fetch (intercept requests). Updates require the old service worker to be replaced — happens on next page load.

**Use cases**: Progressive Web Apps (PWAs), offline-capable apps, performance optimization via caching.

---

Q13. What is content negotiation in HTTP?

A13.
Content negotiation allows the client and server to agree on the format of the response. The client specifies what it can accept, and the server responds in the best matching format.

**How it works**: The client sends `Accept` headers:
- `Accept: application/json` → wants JSON.
- `Accept: text/html` → wants HTML.
- `Accept: application/xml, application/json;q=0.9` → prefers XML, but JSON is acceptable (q = quality factor, 0-1).

**Other negotiation headers**:
- `Accept-Language: en-US, fr;q=0.8` → prefers English, French acceptable.
- `Accept-Encoding: gzip, br` → supports gzip and Brotli compression.
- `Accept-Charset: utf-8` → character encoding.

**Server response**: Responds with `Content-Type: application/json` (or whatever format was chosen). If it can't satisfy any accepted format, returns 406 (Not Acceptable).

**In practice**: Most modern APIs just return JSON. Content negotiation is more relevant for APIs serving multiple formats (JSON + XML), internationalized content, or when optimizing compression. GraphQL APIs skip this entirely — always JSON.

---

### LOW PRIORITY

---

Q14. What is HATEOAS?

A14.
HATEOAS (Hypermedia as the Engine of Application State) is a REST constraint where the server includes links to related actions and resources in its responses. The client discovers available actions dynamically, rather than hardcoding API URLs.

**Example response**:
```json
{
  "order": {
    "id": 42,
    "status": "pending",
    "total": 99.99,
    "links": [
      { "rel": "self", "href": "/orders/42" },
      { "rel": "cancel", "href": "/orders/42/cancel", "method": "POST" },
      { "rel": "payment", "href": "/orders/42/pay", "method": "POST" }
    ]
  }
}
```

If the order status were "shipped," the cancel and payment links wouldn't be included — the client knows those actions aren't available.

**Benefit**: Decouples client from URL structure. Server can change URLs without breaking clients. Client logic is simpler — only show actions that appear in the response.

**Reality**: Most REST APIs don't implement HATEOAS. It adds complexity and most clients are built alongside the API (not truly decoupled). It's the "most mature" level of REST (Richardson Maturity Model Level 3) but rarely worth the overhead in practice.

---

Q15. What are Progressive Web Apps (PWAs)?

A15.
PWAs are web apps that use modern web technologies to deliver app-like experiences — installable, offline-capable, and fast.

**Key technologies**:
- **Service workers**: Enable offline functionality and background sync.
- **Web manifest**: A JSON file describing the app (name, icons, start URL, display mode). Allows "Add to Home Screen" on mobile and desktop.
- **HTTPS**: Required for service workers and security.

**Capabilities**: Work offline, send push notifications, access device hardware (camera, GPS), install on home screen without an app store, auto-update.

**Advantages over native apps**: No app store approval process. Single codebase for all platforms. Instant updates (no user action needed). Smaller than native apps. Discoverable via search engines.

**Limitations**: Limited access to some native APIs (Bluetooth, NFC — improving). iOS support is behind Android. No presence in app stores (though PWABuilder bridges this gap). Performance-intensive tasks are still better native.

**Examples**: Twitter Lite, Starbucks, Pinterest — all reported significant improvements in engagement and performance after going PWA.

---

Q16. What is web accessibility (a11y)? What are the basics?

A16.
Web accessibility ensures that websites are usable by people with disabilities — visual, auditory, motor, and cognitive impairments. It's both an ethical obligation and a legal requirement in many jurisdictions (ADA, European Accessibility Act).

**Core principles (WCAG POUR)**:
- **Perceivable**: Content must be presentable in ways users can perceive. Alt text for images, captions for videos, sufficient color contrast.
- **Operable**: Interface must be navigable. Keyboard navigation (tab through all interactive elements), no time-limited interactions, no seizure-inducing flashing.
- **Understandable**: Content and UI must be understandable. Clear labels, consistent navigation, error identification with suggestions.
- **Robust**: Content must work with assistive technologies. Semantic HTML, proper ARIA attributes, valid markup.

**Quick wins**:
- Use semantic HTML (`<button>`, `<nav>`, `<main>`, `<h1>`-`<h6>`) instead of divs for everything.
- Add `alt` attributes to all images.
- Ensure 4.5:1 color contrast ratio for text.
- Make all interactive elements keyboard-focusable.
- Use ARIA labels when semantic HTML isn't sufficient: `aria-label`, `aria-describedby`, `role`.
- Test with a screen reader (VoiceOver, NVDA).

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q17. What's the difference between a reverse proxy and a load balancer?

A17.
They're related but serve different purposes, and often the same tool (Nginx, HAProxy) does both.

**Reverse proxy**: Sits in front of backend servers. Clients think they're talking to the proxy. It forwards requests to backends, handles SSL termination, caching, compression, and hides backend architecture.

**Load balancer**: Distributes incoming traffic across multiple backend servers to prevent any single server from being overwhelmed.

**Key difference**: A reverse proxy is about abstraction — hiding and protecting backends. A load balancer is about distribution — spreading traffic. A reverse proxy can serve a single backend (still useful for SSL, caching). A load balancer requires multiple backends by definition.

**Load balancing algorithms**:
- **Round-robin**: Rotate through servers sequentially. Simple, works when servers are identical.
- **Weighted round-robin**: Servers with more capacity get more traffic.
- **Least connections**: Send to the server with fewest active connections. Better for varying request durations.
- **IP hash**: Same client IP always goes to the same server (sticky sessions).

**In practice**: Nginx configured as a reverse proxy with an `upstream` block IS a load balancer. Cloud load balancers (ALB, GCP LB) combine both roles plus health checks, auto-scaling integration, and global distribution.

---

Q18. What's the difference between horizontal and vertical scaling?

A18.
**Vertical scaling (scale up)**: Add more power to an existing machine — more CPU, RAM, faster disk. Your single server becomes beefier.

**Horizontal scaling (scale out)**: Add more machines. Instead of one powerful server, run 10 smaller ones behind a load balancer.

**Vertical pros**: Simpler — no distributed systems complexity. No data synchronization issues. Works well up to a point.

**Vertical cons**: Hardware limits — you can't put 10TB of RAM in one box. Single point of failure. Expensive at the top end (the jump from 64GB to 128GB costs more than buying two 64GB machines).

**Horizontal pros**: Theoretically unlimited scaling. Fault tolerance — one machine dying doesn't take down the system. Cost-effective — use commodity hardware.

**Horizontal cons**: Distributed systems complexity — consistency, network partitions, data synchronization. Application must be designed for it (stateless services, shared-nothing architecture). Need load balancers, service discovery, distributed databases.

**What scales how**:
- **Web servers**: Horizontal (stateless, behind load balancer) — easy
- **Application servers**: Horizontal with sticky sessions or shared session store (Redis)
- **Databases**: Vertical first (easier), horizontal later (read replicas, sharding) — hardest to scale horizontally
- **Caches**: Horizontal with consistent hashing (Redis Cluster)

---

### IMPORTANT

---

Q19. How does GraphQL differ from REST, and when would you choose one over the other?

A19.
**REST**: Resources have URLs (`/users/123`). Fixed response shapes per endpoint. Multiple endpoints for different resources. Standard HTTP methods (GET, POST, PUT, DELETE).

**GraphQL**: Single endpoint (`/graphql`). Client specifies exactly what fields it wants in the query. Server returns precisely that shape — nothing more, nothing less.

**GraphQL advantages**:
- **No over-fetching**: REST's `/users/123` returns all fields even if you only need name and email. GraphQL: `{ user(id: 123) { name, email } }`
- **No under-fetching**: Getting a user's posts and comments in REST requires 3 requests. GraphQL gets it in one query with nested fields.
- **Typed schema**: Self-documenting API with introspection. Frontend knows exactly what's available.
- **Ideal for**: Mobile apps (bandwidth-constrained), complex UIs with varied data needs, API consumed by many different clients.

**REST advantages**:
- **Caching**: HTTP caching works out of the box (CDN, browser). GraphQL (POST requests) needs custom caching (Apollo cache).
- **Simplicity**: Easier to understand, implement, and debug. No query complexity to manage.
- **Rate limiting**: Easy per-endpoint. GraphQL needs query cost analysis.
- **File uploads**: Straightforward in REST, awkward in GraphQL.

**Choose GraphQL** when you have many different clients consuming varied slices of data. **Choose REST** for simple CRUD APIs, public APIs with caching needs, or when your team is smaller.

---

Q20. What is database connection pooling, and why is it critical for web applications?

A20.
A connection pool maintains a set of pre-established database connections that are reused across requests rather than creating a new connection for each request.

**Why it matters**: Creating a database connection involves TCP handshake, TLS negotiation, authentication, and session setup — ~50-100ms. If 1000 requests/second each create a new connection, the database drowns in connection overhead. Pooling reuses existing connections, reducing this to a hash table lookup (~1ms).

**How it works**:
1. Pool initializes with `min_connections` (e.g., 5) at startup
2. Request needs a DB connection → borrows one from the pool
3. Request completes → returns the connection to the pool (not closed)
4. If all connections are in use and pool hasn't reached `max_connections`, create a new one
5. If at max, the request waits (with a timeout)

**Key parameters**:
- **min/max pool size**: Too small → requests queue up. Too large → overwhelm the database. Typical: 10-20 per app instance.
- **Connection timeout**: How long to wait for a connection from the pool
- **Idle timeout**: Close connections that haven't been used (prevent stale connections)
- **Max lifetime**: Recycle connections periodically (prevents issues with database failover)

**Tools**: PgBouncer (PostgreSQL connection pooler — sits between app and DB), HikariCP (Java), SQLAlchemy pool (Python), Prisma's connection pool.

**Common pitfall**: Connection leaks — borrowing a connection without returning it. Eventually the pool is exhausted and everything hangs. Always use context managers/try-finally to ensure connections are returned.

---

Q21. How do you configure a CDN, and what are the key considerations?

A21.
A **CDN** (Content Delivery Network) caches your content at edge servers worldwide. Users are served from the nearest edge, reducing latency.

**What to cache**:
- **Static assets**: CSS, JS, images, fonts — cache aggressively (long TTL). Use content hashing in filenames (`app.a1b2c3.js`) so you can cache forever and bust cache by changing the filename.
- **API responses**: Cache GET requests for public data. Short TTL (30s-5min). Use `Cache-Control` and `Vary` headers carefully.
- **HTML pages**: Static sites → cache fully. Dynamic sites → cache edges with `stale-while-revalidate`.

**Key configuration**:
- **Cache-Control header**: `public, max-age=31536000, immutable` for hashed assets. `private, no-cache` for user-specific data.
- **Origin shield**: An intermediate cache between edge and origin. Reduces origin load when many edges request the same content.
- **Purge/invalidation**: When content changes, purge the CDN cache. Instant purge for critical updates, TTL expiry for normal updates.
- **Custom error pages**: Return a cached error page when origin is down (graceful degradation).

**Security**:
- **DDoS protection**: CDNs absorb attack traffic (Cloudflare, AWS CloudFront, Akamai)
- **WAF integration**: Filter malicious requests at the edge
- **TLS termination**: CDN handles HTTPS, reducing SSL overhead on origin

**Providers**: Cloudflare (generous free tier, easy setup), AWS CloudFront (deep AWS integration), Fastly (programmable edge with VCL/Wasm), Akamai (enterprise).

---

### GOOD-TO-HAVE

---

Q22. What are health check endpoints, and why are they important?

A22.
Health check endpoints are HTTP endpoints that report the application's status — used by load balancers, orchestrators (Kubernetes), and monitoring systems to determine if an instance is healthy.

**Types**:
- **Liveness check** (`/health/live`): "Is the process running and not deadlocked?" Returns 200 if the server can respond. If this fails, Kubernetes restarts the container.
- **Readiness check** (`/health/ready`): "Can this instance serve traffic?" Checks dependencies — database connection, cache, downstream services. If this fails, the load balancer stops sending traffic but doesn't restart the instance.
- **Startup check** (`/health/startup`): "Has the application finished initializing?" For slow-starting apps (loading ML models, warming caches). Prevents premature liveness checks.

**What to check in readiness**:
- Database connection is alive (run a simple `SELECT 1`)
- Cache (Redis) is reachable
- Required external services are available
- Disk space is sufficient

**What NOT to check**: Don't fail readiness because a non-critical dependency is down. If your notification service is down, you can still serve requests — just log a warning.

**Best practices**: Keep health checks fast (<100ms). Don't do expensive operations. Return structured JSON (`{"status": "healthy", "db": "up", "cache": "up"}`). Include version/git commit for debugging.

---

Q23. How do you implement graceful shutdown in a web server?

A23.
Graceful shutdown means the server stops accepting new requests but finishes processing in-flight requests before exiting.

**The problem**: A hard kill (`SIGKILL`, `kill -9`) terminates immediately — in-flight requests get dropped, database transactions are interrupted, users see errors.

**The solution** (respond to `SIGTERM`):
1. Receive SIGTERM signal (sent by Kubernetes, Docker, systemd before SIGKILL)
2. Stop accepting new connections (close the listening socket / deregister from load balancer)
3. Wait for in-flight requests to complete (with a timeout — typically 30s)
4. Close database connections, flush buffers, release resources
5. Exit cleanly

```python
# Python (FastAPI/Uvicorn)
import signal
async def shutdown():
    # Stop accepting new requests
    server.should_exit = True
    # Wait for in-flight requests (handled by the server)
    
# Node.js
process.on('SIGTERM', () => {
    server.close(() => {  // stops accepting new connections
        db.end();         // close DB pool
        process.exit(0);  // exit after cleanup
    });
});
```

**Kubernetes integration**: Kubernetes sends SIGTERM, waits `terminationGracePeriodSeconds` (default 30s), then sends SIGKILL. Also, add a `preStop` hook with a small sleep (5s) to allow the load balancer to deregister the pod before SIGTERM — prevents requests hitting a terminating pod.

**Key**: Set your graceful shutdown timeout slightly less than Kubernetes' `terminationGracePeriodSeconds` so the application exits cleanly before SIGKILL.