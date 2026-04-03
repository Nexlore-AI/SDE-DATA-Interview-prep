==============================
FILE: Computer Networks
==============================

### HIGH PRIORITY

---

Q1. Explain the OSI model layers. What happens at each layer?

A1.
Seven layers, bottom to top:

**Layer 1 — Physical**: Raw bits over a medium — cables, radio waves, voltage levels. Ethernet cables, Wi-Fi signals.

**Layer 2 — Data Link**: Frames. MAC addresses, error detection (CRC), flow control within a local network. Switches operate here. Ethernet, Wi-Fi (802.11).

**Layer 3 — Network**: Packets. IP addressing, routing between different networks. Routers operate here. IP, ICMP.

**Layer 4 — Transport**: Segments. End-to-end communication, reliability, flow control. TCP (reliable, ordered) and UDP (fast, no guarantees). Port numbers live here.

**Layer 5 — Session**: Manages sessions between applications. Authentication, reconnection. In practice, this is often handled at layer 7.

**Layer 6 — Presentation**: Data format translation, encryption, compression. SSL/TLS technically lives here, though in practice it's bundled with layer 7.

**Layer 7 — Application**: What users and developers interact with. HTTP, DNS, SMTP, FTP.

In reality, the TCP/IP model (4 layers: Link, Internet, Transport, Application) is what's actually implemented. OSI is the conceptual reference model used for discussion.

---

Q2. What is the difference between TCP and UDP? When would you choose each?

A2.
**TCP**: Connection-oriented, reliable delivery, ordered, flow control, congestion control. Three-way handshake to establish connection. Guarantees every byte arrives in order. Slower due to overhead.

**UDP**: Connectionless, no guarantees on delivery, ordering, or duplication. Fire-and-forget. Faster, lower overhead.

**Choose TCP when**: Data integrity matters — web pages (HTTP), file transfers (FTP), emails (SMTP), database connections. You can't afford to lose or reorder data.

**Choose UDP when**: Speed matters more than reliability — real-time video/audio (WebRTC, VoIP), online gaming (player positions — if one update is lost, the next one supersedes it), DNS queries (small, single-request-response), IoT sensor data.

**Modern nuance**: QUIC (used by HTTP/3) is built on UDP but implements its own reliability and flow control at the application level — getting TCP-like reliability with UDP's speed (no head-of-line blocking, faster connection setup).

---

Q3. What is DNS, and how does DNS resolution work step by step?

A3.
DNS translates domain names (google.com) to IP addresses (142.250.x.x).

**Resolution steps** (iterative):
1. Browser checks its local cache.
2. OS checks its resolver cache (`/etc/hosts`, system DNS cache).
3. Query goes to the recursive resolver (usually your ISP's or 8.8.8.8).
4. Resolver checks its cache. If not cached:
5. Asks a root nameserver (". servers") → "I don't know google.com, but here's the .com TLD server."
6. Asks the .com TLD server → "I don't know google.com, but here's Google's authoritative nameserver."
7. Asks Google's authoritative nameserver → "google.com is 142.250.x.x" with a TTL.
8. Resolver caches the result (for TTL duration) and returns it.

**TTL** (Time to Live) controls cache duration. Short TTL = faster DNS changes but more queries. Long TTL = fewer queries but slower propagation of changes. For services behind a load balancer, TTL of 60-300 seconds is common.

---

Q4. What is the TCP three-way handshake? Why three steps?

A4.
**SYN**: Client sends a SYN packet with a random sequence number (say, seq=100). "I want to connect, starting at sequence 100."

**SYN-ACK**: Server responds with its own SYN (seq=300) and acknowledges the client's (ack=101). "Got it, I'm starting at sequence 300."

**ACK**: Client acknowledges the server's sequence (ack=301). "Got it. Connection established."

**Why three steps?** Two steps aren't enough because the server wouldn't know if the client received its SYN-ACK. The third step confirms both sides have synchronized their sequence numbers and are ready.

Two steps would also allow **SYN flood attacks** to be even more devastating — the server would consider connections established with just one packet. The three-way handshake also prevents old duplicate SYN packets from accidentally establishing ghost connections (the sequence numbers wouldn't match).

Connection teardown uses a four-way handshake (FIN, ACK, FIN, ACK) because each direction closes independently — half-close is possible.

---

Q5. What happens when you type a URL in the browser and press Enter?

A5.
1. **URL parsing**: Browser parses the URL — protocol (https), domain (example.com), path (/page), port (443).
2. **DNS resolution**: Resolves domain to IP (as described above — cache → resolver → root → TLD → authoritative).
3. **TCP connection**: Three-way handshake with the server's IP on port 443.
4. **TLS handshake**: For HTTPS — client hello, server hello, certificate exchange, key exchange. Establishes encrypted channel. Adds 1-2 round trips.
5. **HTTP request**: Browser sends `GET /page HTTP/1.1` with headers (Host, User-Agent, cookies, Accept-Encoding).
6. **Server processing**: Server receives request, runs application logic, queries database, renders response.
7. **HTTP response**: Server returns status code (200), headers (Content-Type, Cache-Control), and body (HTML).
8. **Rendering**: Browser parses HTML → builds DOM. Fetches CSS, JS, images (parallel requests). Builds CSSOM → renders layout → paints pixels.
9. **Subsequent requests**: CSS and JS may trigger additional requests. HTTP/2 multiplexes these over the same connection.

---

Q6. What is HTTPS? How does TLS ensure security?

A6.
HTTPS is HTTP over TLS. TLS provides three guarantees:

**Confidentiality**: Data is encrypted — an eavesdropper on the network can't read it. Uses symmetric encryption (AES) for data, with keys exchanged via asymmetric encryption (RSA/ECDHE) during the handshake.

**Integrity**: Data can't be tampered with in transit. MACs (Message Authentication Codes) detect any modification.

**Authentication**: The server proves its identity via a certificate signed by a trusted Certificate Authority (CA). The browser verifies the certificate chain: server cert → intermediate CA → root CA (pre-installed in the browser/OS).

**TLS 1.3 handshake** (simplified): Client sends supported cipher suites + key share. Server responds with chosen suite + its key share + certificate. One round trip. Both derive the session key and start encrypting.

Why it matters: without HTTPS, anyone on the same Wi-Fi can read passwords, session cookies, and modify page content (inject ads, malware). Modern browsers actively warn users about non-HTTPS sites.

---

Q7. What is the difference between HTTP/1.1, HTTP/2, and HTTP/3?

A7.
**HTTP/1.1**: Text-based protocol. One request per TCP connection at a time (or pipelining, which was poorly implemented). Browsers open 6-8 parallel connections per domain to work around this. Headers are uncompressed and repeated on every request.

**HTTP/2**: Binary protocol, multiplexing — multiple requests and responses over a single TCP connection simultaneously. Header compression (HPACK). Server push (server can send resources before the client asks). But: still built on TCP, so a single lost packet blocks all streams (head-of-line blocking at the TCP level).

**HTTP/3**: Built on QUIC (which runs on UDP). Eliminates TCP head-of-line blocking — a lost packet in one stream doesn't block others. Faster connection setup (0-RTT for repeat connections). Built-in encryption (TLS 1.3 is part of QUIC, not a separate handshake).

Migration path: most major sites support HTTP/2. Google, Cloudflare, and Meta actively use HTTP/3. The protocol negotiation is automatic — clients try HTTP/3, fall back to HTTP/2 or HTTP/1.1.

---

### MEDIUM PRIORITY

---

Q8. What are the common HTTP methods, and what is idempotency in the context of HTTP?

A8.
**GET**: Retrieve a resource. Should be safe (no side effects) and idempotent.
**POST**: Create a resource or trigger an action. NOT idempotent — calling it twice may create duplicates.
**PUT**: Replace a resource entirely. Idempotent — calling it twice with the same data has the same result.
**PATCH**: Partially update a resource. May or may not be idempotent depending on implementation.
**DELETE**: Remove a resource. Idempotent — deleting the same resource twice results in the same state (it's gone).

**Idempotency** means making the same request multiple times produces the same result as making it once. GET, PUT, DELETE are idempotent by specification. POST is not.

Why it matters: network failures. If a POST times out, the client doesn't know if it succeeded — retrying might create a duplicate order. Solutions: idempotency keys (client sends a unique key with the request; server deduplicates), or design POST endpoints to be idempotent where possible.

---

Q9. What is a WebSocket? How does it differ from HTTP?

A9.
HTTP is request-response: client asks, server answers, done. For real-time updates (chat, live scores, stock tickers), the client would need to repeatedly poll the server.

WebSocket starts as an HTTP request (upgrade handshake), then switches to a persistent, full-duplex connection. Both client and server can send messages at any time without the overhead of HTTP headers on every message.

**Use cases**: Chat applications, live dashboards, multiplayer games, collaborative editing, real-time notifications.

**vs. Server-Sent Events (SSE)**: SSE is simpler — server pushes events to the client over a regular HTTP connection. Unidirectional (server → client only). Good for live feeds and notifications. WebSocket is bidirectional — necessary when the client also sends frequent messages.

**vs. Long Polling**: Client makes a request, server holds it open until data is available. Works everywhere but wastes connections and has higher latency than WebSocket.

---

Q10. What is a CDN, and how does it improve performance?

A10.
A Content Delivery Network is a distributed network of servers (edge nodes / PoPs) that caches content close to users geographically. Instead of every user hitting your origin server in Virginia, users in Tokyo hit a Tokyo edge node.

**What it caches**: Static assets (images, CSS, JS, videos), and sometimes dynamic content (with shorter TTLs or edge computing).

**How it works**: First request from a region goes to the edge node → cache miss → edge fetches from origin → caches the response → serves it. Subsequent requests from that region hit the cache directly.

**Benefits**: Lower latency (shorter round trip), reduced origin server load, DDoS mitigation (absorb traffic at the edge), SSL termination at the edge.

**Cache invalidation**: The hard part. Strategies: TTL-based expiration, versioned URLs (`style.v2.css`), manual purge APIs. Most CDNs support cache tags — invalidate all resources with a specific tag.

Providers: Cloudflare, AWS CloudFront, Akamai, Fastly. For most web applications, putting a CDN in front costs almost nothing and dramatically improves global performance.

---

Q11. What is NAT (Network Address Translation)? Why is it used?

A11.
NAT translates private IP addresses (192.168.x.x) to a public IP address and vice versa. Your home router uses NAT — all your devices share one public IP.

**Why**: IPv4 has only ~4 billion addresses (exhausted). NAT lets thousands of devices share one public IP. It also provides a level of security — devices behind NAT aren't directly reachable from the internet.

**How it works**: When your laptop (192.168.1.10:5000) sends a request to google.com, the router rewrites the source to its public IP (203.0.113.1:12345), mapping the external port to your internal IP:port. When the response arrives at port 12345, the router maps it back to 192.168.1.10:5000.

**Downsides**: Breaks end-to-end connectivity (peer-to-peer requires NAT traversal techniques like STUN/TURN/ICE). Complicates hosting servers at home. IPv6 was designed to eliminate the need for NAT by providing enough addresses for every device.

---

Q12. What is the difference between a switch, a router, and a gateway?

A12.
**Switch** (Layer 2): Forwards frames within the same network (LAN) based on MAC addresses. It learns which MAC is on which port. Doesn't know about IP addresses.

**Router** (Layer 3): Forwards packets between different networks based on IP addresses. Uses routing tables to decide the next hop. Your home router connects your LAN to your ISP's network.

**Gateway**: A broader term — any device that serves as an entry/exit point between two different networks or protocols. Your home router is a gateway. An API gateway translates between external HTTP requests and internal microservice protocols. A mail gateway converts between different email formats.

In practice, your home "router" is actually a router + switch + wireless access point + NAT + DHCP server + firewall all in one device.

---

Q13. What is a load balancer, and how does it distribute traffic?

A13.
A load balancer distributes incoming traffic across multiple backend servers to prevent any single server from being overwhelmed.

**Algorithms**:
- **Round Robin**: Each request goes to the next server in sequence. Simple, works when servers are identical.
- **Weighted Round Robin**: Servers with more capacity get more requests.
- **Least Connections**: Send to the server with fewest active connections.
- **IP Hash**: Hash the client's IP to consistently route to the same server (sticky sessions).
- **Random**: Surprisingly effective at large scale with low overhead.

**L4 (Transport layer)**: Routes based on IP and port. Doesn't inspect the request content. Faster. HAProxy, AWS NLB.

**L7 (Application layer)**: Inspects HTTP headers, URLs, cookies. Can route `/api` to API servers and `/static` to CDN. Nginx, AWS ALB, Envoy.

Health checks: the load balancer pings servers periodically and removes unhealthy ones from the pool.

---

Q14. What is ARP (Address Resolution Protocol)?

A14.
ARP maps IP addresses to MAC addresses on a local network. When your computer wants to send a packet to 192.168.1.1, it needs the MAC address of that device to construct the Ethernet frame.

**Process**: The sender broadcasts an ARP request: "Who has 192.168.1.1? Tell me your MAC." The device with that IP responds with its MAC address. The sender caches this in its ARP table for future use.

If the destination is on a different network, ARP resolves the gateway's (router's) MAC address instead. The packet is sent to the router at layer 2, and the router handles layer 3 forwarding.

**Security concern**: ARP spoofing — an attacker sends fake ARP replies to redirect traffic through their machine (man-in-the-middle). Mitigations: static ARP entries, DHCP snooping, 802.1X authentication.

---

### LOW PRIORITY

---

Q15. What is subnetting, and how do you calculate a subnet mask?

A15.
Subnetting divides a network into smaller sub-networks. It controls which IP addresses can communicate directly (same subnet) vs. which need a router.

A subnet mask defines the boundary: `255.255.255.0` (or `/24` in CIDR notation) means the first 24 bits are the network portion, the last 8 bits are host addresses. This gives 256 addresses (254 usable — minus network address and broadcast).

**Example**: `192.168.1.0/24` → hosts from 192.168.1.1 to 192.168.1.254.

**Splitting**: `192.168.1.0/25` splits into two subnets: `.0-.127` and `.128-.255`, each with 126 usable hosts.

**Why subnet**: Reduce broadcast domains (broadcasts stay within the subnet), improve security (isolate departments/environments), efficient IP allocation (don't waste a /24 on a 5-device network).

Quick math: hosts = 2^(32 - prefix) - 2. So /24 = 254 hosts, /27 = 30 hosts, /30 = 2 hosts (point-to-point links).

---

Q16. What are cookies, and how do they work in HTTP?

A16.
HTTP is stateless — the server doesn't inherently know if two requests come from the same user. Cookies maintain state.

**How they work**: Server sends a `Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict` header. The browser stores the cookie and sends it with every subsequent request to that domain via the `Cookie` header.

**Types**:
- **Session cookies**: No expiry — deleted when browser closes.
- **Persistent cookies**: Have an `Expires` or `Max-Age` date.
- **HttpOnly**: Not accessible via JavaScript — protects against XSS stealing cookies.
- **Secure**: Only sent over HTTPS.
- **SameSite**: Controls cross-origin behavior — prevents CSRF attacks.

**Use cases**: Session management (authentication), user preferences, tracking (analytics, ads).

**Limitations**: 4KB per cookie, limited number per domain. For larger state, use server-side sessions with a cookie holding just the session ID.

---

Q17. What is the difference between symmetric and asymmetric encryption?

A17.
**Symmetric**: One key for both encryption and decryption. AES, ChaCha20. Fast — used for bulk data encryption. Problem: how do you securely share the key with the other party?

**Asymmetric**: Two keys — public key encrypts, private key decrypts (or: private key signs, public key verifies). RSA, ECDSA, Ed25519. Slow — used for key exchange and digital signatures, not bulk data.

**TLS combines both**: Asymmetric encryption during the handshake to securely exchange a symmetric session key. Then symmetric encryption for the actual data transfer. Best of both worlds — secure key exchange from asymmetric, performance from symmetric.

Digital signatures work the inverse way: the sender signs with their private key, anyone can verify with the public key. This proves the sender's identity and data integrity.

---

Q18. What is a firewall, and what are the different types?

A18.
A firewall filters network traffic based on predefined rules — deciding what to allow and block.

**Packet filtering** (Layer 3/4): Inspects individual packets — source/destination IP, port, protocol. Fast but stateless — doesn't understand connections. `iptables` rules.

**Stateful inspection**: Tracks connection state. Knows that an incoming packet is part of an established outbound connection. More intelligent than packet filtering. Most modern firewalls.

**Application-layer firewall** (Layer 7): Inspects the actual content — HTTP requests, SQL queries. Can detect and block SQL injection, XSS, malicious payloads. Web Application Firewalls (WAFs) like Cloudflare WAF, AWS WAF.

**Network segmentation**: Place firewalls between zones — public internet → DMZ → internal network → database zone. Each boundary has specific rules. Defense in depth.

Cloud equivalents: Security Groups (AWS/Azure) are stateful firewalls per instance. NACLs are stateless per subnet.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q19. What are the key differences between IPv4 and IPv6?

A19.
**IPv4**: 32-bit addresses (4.3 billion total). Written as four octets: `192.168.1.1`. We've basically run out — NAT (Network Address Translation) lets many devices share one public IP, but it adds complexity.

**IPv6**: 128-bit addresses (3.4 × 10^38 — practically unlimited). Written as eight groups of hex: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`. Leading zeros can be omitted, and consecutive all-zero groups collapse to `::`.

Key differences:
- **Address space**: IPv4 = 4.3 billion. IPv6 = billions of addresses per person on Earth
- **NAT**: IPv4 relies heavily on NAT. IPv6 restores end-to-end connectivity — every device gets a globally unique address
- **Header**: IPv6 has a simpler fixed-size header (40 bytes). IPv4 has variable-length headers with optional fields
- **IPsec**: Optional in IPv4, built into IPv6 spec (though not always enforced)
- **Broadcasting**: IPv4 has broadcast. IPv6 replaces it with multicast and anycast
- **DHCP**: IPv6 supports SLAAC (Stateless Address Autoconfiguration) — devices auto-generate their address from the network prefix + their MAC address

Adoption: Dual-stack (supporting both) is the transition strategy. Google reports ~45% of traffic over IPv6. Major cloud providers and CDNs support IPv6 natively.

---

Q20. What's the difference between a forward proxy and a reverse proxy?

A20.
**Forward proxy**: Sits between clients and the internet. The client knows it's using a proxy. Requests go: Client → Proxy → Server. The server sees the proxy's IP, not the client's.

Use cases: Corporate networks filtering employee internet access, caching frequently accessed content, bypassing geo-restrictions, anonymizing client identity.

**Reverse proxy**: Sits in front of servers. The client doesn't know it's hitting a proxy — it thinks it's talking to the actual server. Requests go: Client → Reverse Proxy → Backend Server.

Use cases: Load balancing across multiple servers, SSL termination (handle HTTPS at the proxy, HTTP to backends), caching responses, protecting backend servers from direct exposure, rate limiting, WAF functionality.

Examples: **Nginx** and **HAProxy** are the most common reverse proxies. **Squid** is a traditional forward proxy. **Cloudflare** acts as a reverse proxy for DDoS protection and CDN.

Key distinction: Forward proxy — client side, protects clients. Reverse proxy — server side, protects servers. In an interview, if someone says "proxy," ask which kind — they have very different purposes.

---

Q21. How does DHCP work?

A21.
DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to devices on a network. The process is called **DORA**:

1. **Discover**: Client broadcasts "I need an IP address" (UDP, destination 255.255.255.255, source 0.0.0.0). It has no IP yet.
2. **Offer**: DHCP server responds with an available IP, subnet mask, gateway, DNS servers, and a lease time.
3. **Request**: Client broadcasts acceptance of the offer (broadcast because multiple DHCP servers might have offered, and the others need to know).
4. **Acknowledge**: Server confirms; client configures its network interface.

**Lease**: The IP assignment isn't permanent — it expires after the lease time (often 24 hours). The client tries to renew at 50% of the lease time, then again at 87.5%. If renewal fails, the client must restart the DORA process.

**DHCP relay**: In large networks with multiple subnets, you don't put a DHCP server on every subnet. A relay agent on each subnet forwards DHCP broadcasts to the central DHCP server as unicast packets.

Common issue: DHCP exhaustion — if the pool runs out of addresses, new devices can't connect. Monitor pool utilization and size appropriately.

---

### IMPORTANT

---

Q22. What is a VPN, and what are the main types?

A22.
A VPN (Virtual Private Network) creates an encrypted tunnel between a client and a network, making it appear as if the client is on that network directly.

**Site-to-Site VPN**: Connects two networks (e.g., two office locations). Runs between routers/firewalls. Uses IPsec typically. Always-on.

**Remote Access VPN**: Individual users connect to a corporate network from anywhere. The client encrypts all traffic and sends it through the tunnel.

**Protocols**:
- **IPsec**: Network layer. Two modes — tunnel (encrypts entire packet, new IP header) and transport (encrypts only payload). Widely supported.
- **OpenVPN**: Uses TLS/SSL over UDP or TCP. Open source, highly configurable. Runs in user space.
- **WireGuard**: Modern, minimal codebase (~4000 lines), fast, uses state-of-the-art cryptography. In the Linux kernel since 5.6.
- **SSL/TLS VPN**: Works through the browser or a lightweight client. Good for accessing specific web applications without full network access.

Split tunneling: Only route corporate traffic through the VPN; regular internet traffic goes directly. Reduces VPN bandwidth but means internet traffic isn't protected by the corporate firewall.

---

Q23. How does TCP congestion control work?

A23.
TCP congestion control prevents senders from overwhelming the network. It uses a **congestion window (cwnd)** that limits how much unacknowledged data can be in transit.

**Slow Start**: Start with cwnd = 1 MSS (Maximum Segment Size). Double it every RTT (round-trip time) — exponential growth. Despite the name, it ramps up quickly.

**Congestion Avoidance**: Once cwnd reaches the **ssthresh** (slow start threshold), switch to linear growth — increase by 1 MSS per RTT. Cautious expansion.

**Loss Detection**:
- **Timeout**: No ACK received. Assume severe congestion. Set ssthresh = cwnd/2, reset cwnd to 1. Start slow start again. (TCP Tahoe)
- **3 duplicate ACKs**: Fast retransmit — just one packet was lost, not severe congestion. Set ssthresh = cwnd/2, cwnd = ssthresh (skip slow start). (TCP Reno — Fast Recovery)

**Modern algorithms**:
- **CUBIC** (Linux default): Uses a cubic function for window growth — aggressive recovery after loss. Optimized for high-bandwidth, high-latency networks.
- **BBR** (Google): Doesn't rely on packet loss. Instead, measures bottleneck bandwidth and minimum RTT, maintains the optimal sending rate. Better for lossy networks (wireless, long-haul).

This is why initial page loads feel slow — TCP is in slow start, sending just a few packets. Connection reuse (HTTP keep-alive, HTTP/2 multiplexing) avoids restarting slow start.

---

Q24. What are sockets and how does socket programming work?

A24.
A socket is an endpoint for communication — identified by an IP address and port number. It's the OS-level API for network communication.

**TCP socket lifecycle** (server):
1. `socket()` — create a socket
2. `bind()` — associate with an IP and port
3. `listen()` — mark as passive, start accepting connections
4. `accept()` — block until a client connects; returns a new socket for that connection
5. `read()`/`write()` — send and receive data
6. `close()` — terminate the connection

**Client**:
1. `socket()` — create
2. `connect()` — initiate TCP handshake with server
3. `read()`/`write()` — communicate
4. `close()` — disconnect

**UDP sockets** are simpler: no `listen()`/`accept()`/`connect()`. Just `sendto()` and `recvfrom()` with addresses.

**File descriptors**: Each socket is a file descriptor (everything is a file in Unix). This is why `select()`/`epoll()` work on sockets — they're just FDs being monitored for readability/writability.

Connection state: A TCP connection is identified by the 4-tuple: (source IP, source port, destination IP, destination port). A server listening on port 80 can handle thousands of clients — each gets a unique socket from different source IPs/ports.

---

Q25. What is the QUIC protocol and how does it improve on TCP+TLS?

A25.
QUIC is a transport protocol built on UDP, developed by Google and standardized as HTTP/3's transport layer. It solves several TCP problems:

**0-RTT connection setup**: TCP needs 1 RTT for handshake + 1-2 RTT for TLS = 2-3 RTTs before data flows. QUIC combines transport and crypto handshake into 1 RTT. For resumed connections, 0 RTT — data is sent with the first packet.

**No head-of-line blocking**: TCP treats everything as one byte stream. If packet 3 is lost, packets 4, 5, 6 must wait even if they're for different resources. QUIC has independent streams — loss on one stream doesn't block others.

**Connection migration**: TCP connections are tied to the 4-tuple (IPs + ports). Switch Wi-Fi to cellular? Connection breaks. QUIC uses a Connection ID — survives network changes.

**Built-in encryption**: TLS 1.3 is mandatory and integrated into the protocol, not a layer on top.

**Improved loss recovery**: Each packet has a unique, strictly increasing packet number (no ambiguity like TCP's retransmission). Better RTT estimation and loss detection.

Adoption: Google services, Facebook, Cloudflare all use QUIC. HTTP/3 = HTTP over QUIC. Most modern browsers support it. It's the future of web transport.

---

### GOOD-TO-HAVE

---

Q26. What is BGP, and why is it important to the Internet?

A26.
**BGP** (Border Gateway Protocol) is the routing protocol that makes the Internet work. It's how autonomous systems (ASes) — large networks like ISPs, cloud providers, and enterprises — exchange routing information to figure out how to reach each other.

Each AS has a unique ASN (Autonomous System Number). BGP routers advertise which IP prefixes they can reach. When AS1 announces "I can reach 203.0.113.0/24," neighboring ASes learn that path and propagate it further.

**How routing decisions happen**: BGP uses path attributes — most importantly the AS_PATH (list of ASes a route traverses). Shorter paths are preferred. Operators also use **local preference**, **MED** (Multi-Exit Discriminator), and **communities** to influence routing policy.

**Why it matters**: A BGP misconfiguration can take down parts of the Internet. In 2021, Facebook's BGP routes were withdrawn, making their DNS unreachable — a 6-hour global outage. BGP hijacking (announcing someone else's prefixes) is a real security concern — RPKI (Resource Public Key Infrastructure) helps validate route origins.

BGP is often called "the protocol that runs the Internet" — it's the glue connecting ~70,000+ autonomous systems globally.

---

Q27. What's the difference between unicast, broadcast, multicast, and anycast?

A27.
**Unicast**: One sender → one receiver. Standard point-to-point communication. When you visit google.com, your packets go to one specific server.

**Broadcast**: One sender → all devices on the network. The packet reaches every device in the broadcast domain. Used in ARP ("who has IP 192.168.1.1?") and DHCP discovery. Only works within a LAN — routers don't forward broadcasts. IPv6 replaced broadcast with multicast.

**Multicast**: One sender → a group of interested receivers. Devices subscribe to a multicast group (IP range 224.0.0.0/4). Only subscribers receive the traffic. Used for live video streaming, stock market data feeds, and service discovery (mDNS uses 224.0.0.251). More efficient than sending individual unicast streams to each viewer.

**Anycast**: One sender → the nearest (by routing metric) receiver among a group sharing the same IP. Multiple servers around the world advertise the same IP address. BGP routes each client to the closest one. Used extensively by DNS root servers and CDNs. Cloudflare's `1.1.1.1` DNS uses anycast — your query goes to the nearest Cloudflare data center, not one specific server.

Key insight: Anycast is about routing (network layer) — the same IP resolves to different physical servers. Multicast is about delivery (transport layer) — one packet reaches multiple specific subscribers.