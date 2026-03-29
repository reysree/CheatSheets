# Load Balancer

## What is a Load Balancer?
A load balancer is a server or hardware appliance that sits in front of your backend servers and distributes incoming requests across them. It acts as a traffic cop — no single server gets overwhelmed.

**Use it when:** You have more traffic than one server can handle, or you need high availability.

---

## When Should You Consider It?
Trigger words in an interview or problem statement:
- "Millions of users" or "high traffic"
- "Server is overloaded"
- "High availability" or "no single point of failure"
- "Scale horizontally"
- "Distribute requests"

---

## Load Balancing Strategies

### Round Robin
- Requests distributed one after another: S0 → S1 → S2 → S0 → ...
- Simple, fast, assumes all servers are equal
- Use when: Servers are identical, requests are roughly equal in weight
- Avoid when: Requests have wildly different processing times

### Weighted Round Robin
- Assign weights to servers based on capacity
- Higher-capacity servers get proportionally more requests
- Use when: Servers have different hardware specs

### Random
- Picks a random server for each request
- Works well at scale but has no intelligence
- Use when: Servers are identical and you want minimal overhead

### Least Connections
- Routes to the server with fewest active connections
- Use when: Connections are long-lived (e.g. WebSockets, persistent connections)
- Smarter than round robin for uneven workloads

### Least Response Time
- Routes to the server that's responding fastest
- Use when: You want to optimize for latency and servers have varying loads

### IP Hashing
- Hash the client's IP to determine which server handles all their requests
- Same client always goes to same server = sticky sessions
- Use when: You need session persistence without a shared session store
- Downside: If that server dies, session is lost

### Consistent Hashing
- Servers and requests are placed on a virtual ring using a hash function
- Each request maps to the nearest server clockwise on the ring
- When a server is added or removed, only a small fraction of requests remap
- Far less disruptive than modulo-based hashing
- Use when: You need stickiness AND graceful server addition/removal (caches, databases, session stores)

#### How Consistent Hashing Works (Implementation Detail)
1. One hash function H(x) is used for everything
2. Each physical server S creates log(m) virtual nodes by hashing different inputs: H("S0_0"), H("S0_1"), ..., H("S0_logm")
3. Each virtual node gets a different position on the ring (different input → different hash output)
4. Each request key is hashed ONCE: H(request_key) → placed on ring → routed to nearest server clockwise
5. Servers get multiple positions (virtual nodes) to prevent skewed distribution
6. Requests get hashed only once — they're transient and don't need replicas

**Key insight:** You're not hashing S0 repeatedly — you're hashing "S0_0", "S0_1" etc. Different inputs produce different outputs. Same hash function, different inputs.

**Round Robin vs Consistent Hashing:**
| | Round Robin | Consistent Hashing |
|---|---|---|
| Stickiness | No | Yes |
| Server removal impact | All requests remap | Only affected requests remap |
| Complexity | Simple | More complex |
| Use case | Stateless servers | Caches, session stores, DBs |

---

## OSI Model — Where Load Balancing Lives

| Layer | Name | What it handles |
|---|---|---|
| 1 | Physical | Cables, hardware |
| 2 | Data Link | MAC addresses, switches |
| 3 | Network | IP addresses, routers |
| 4 | Transport | TCP/UDP, ports |
| 5 | Session | Connection management |
| 6 | Presentation | Encryption, compression |
| 7 | Application | HTTP, SMTP, DNS |

**Load balancing is practically used at Layer 4 and Layer 7 only.**

---

## Layer 4 vs Layer 7 Load Balancing

### Layer 4 (Transport Layer)
- Sees: Source IP, destination IP, port numbers only
- Does NOT inspect request content
- Routes at connection level — once TCP connection goes to Server A, all traffic stays there
- Fast, low CPU overhead
- Think: postal worker who reads only the envelope address

**Use when:**
- All backend servers are identical/interchangeable
- You don't need to route based on content
- Optimizing for raw throughput and speed

**Real-world examples:**
- DNS servers (UDP port 53)
- Video streaming origin servers
- Gaming servers
- Redis/cache clusters
- VoIP/video conferencing

### Layer 7 (Application Layer)
- Sees: Full HTTP request — URL, headers, body, cookies
- Makes intelligent routing decisions based on content
- Routes at request level — different requests on same connection can go to different servers
- Slower than L4 due to deeper inspection
- Think: mail sorter who opens envelopes and reads content

**Use when:**
- You have specialized server pools for different request types
- You need to route based on URL path, headers, or content type
- You're running microservices with different endpoints

**Real-world examples:**
- Social media: /feed → lightweight servers, /upload → heavy servers
- E-commerce: /products → product service, /payments → payment service
- API gateway routing to different microservices

---

## Two-Tier Load Balancing (L4 + L7 Combined)

Used by large-scale systems (Google, Netflix, Amazon):

```
Client → L4 Load Balancer → L7 Load Balancers (pool) → Backend Servers
```

- L4 at front: handles massive raw traffic volume, distributes connections to L7 pool
- L7 behind it: inspects content and routes intelligently to specialized backends
- Best of both worlds: speed at the front, intelligence at the back

---

## CDN vs Load Balancer — Key Distinction

| | CDN | Load Balancer |
|---|---|---|
| Solves | Geography problem | Capacity problem |
| How | Caches content at edge locations near users | Distributes traffic across servers |
| Example | Edge server in New York serves NYC users | 5 origin servers share traffic equally |

**In practice (e.g. YouTube/Netflix):**
1. User request → CDN edge server (nearest geography) → cache hit? Serve immediately
2. Cache miss → CDN fetches from origin → Load balancer distributes to origin servers
3. L4 or L7 at origin handles routing to specific servers

---

## Health Checks
- Load balancer periodically pings each backend server
- If server stops responding → stop sending traffic to it
- Active health check: sends test requests to verify server is alive
- Passive health check: observes real traffic for failures

---

## Load Balancer Failover
- Single load balancer = single point of failure
- Solution: Two load balancers
  - Active-Passive: One handles traffic, other takes over if primary dies
  - Active-Active: Both handle traffic simultaneously

---

## Rate Limiting at the Load Balancer
- The load balancer is the first line of defence against traffic abuse
- It can reject or queue requests if a single client sends too many too fast
- Protects backend servers from being overwhelmed by one bad actor

**Common strategies:**
- Token bucket: each client gets a bucket of tokens; each request costs one token; bucket refills over time
- Leaky bucket: requests enter a queue and are processed at a fixed rate regardless of burst
- Fixed window: allow X requests per client per time window (e.g. 100 req/min)
- Sliding window: more accurate version of fixed window — tracks requests over a rolling time period

**Trigger words:** "prevent abuse", "DDoS protection", "API rate limits", "quota per user"

---

## SSL Termination
- HTTPS requires an expensive TLS handshake for every new connection
- If every backend server handles SSL, they all waste CPU on encryption/decryption
- Solution: load balancer handles the TLS handshake (SSL termination)
  - Client talks HTTPS to the load balancer
  - Load balancer decrypts and forwards plain HTTP to backend servers
  - Backend servers are now free from encryption overhead
- The internal network (load balancer → backends) is typically trusted/private so plain HTTP is acceptable

**Benefit:** Backend servers are simpler, cheaper, and faster
**Trigger words:** "HTTPS at scale", "TLS overhead", "certificate management"

---

## Global Server Load Balancing (GSLB)
- Regular load balancing distributes traffic across servers within ONE data centre
- GSLB distributes traffic across MULTIPLE data centres in different geographic regions
- Uses DNS to route users to the nearest or healthiest data centre

**How it works:**
1. User queries DNS for your domain
2. GSLB-aware DNS responds with the IP of the nearest/healthiest data centre
3. User connects to that data centre
4. Regular load balancer inside that data centre handles the rest

**Use when:**
- You have users in multiple continents
- You need disaster recovery across regions
- One data centre going down should not take down the whole system

**Trigger words:** "global users", "multi-region", "disaster recovery", "data centre failover"

**GSLB vs CDN:**
- CDN caches static content at edge locations
- GSLB routes users to the right data centre for dynamic/compute-heavy requests
- In practice, large systems use both together

---

## Client-Side vs Server-Side Load Balancing (Microservices)

### Server-Side Load Balancing (traditional)
- A dedicated load balancer sits between clients and services
- Client talks to load balancer, load balancer picks the server
- Client has no knowledge of backend servers
- Examples: Nginx, HAProxy, AWS ALB

### Client-Side Load Balancing
- The client itself decides which backend server to call
- Client holds a list of available servers (from a service registry like Consul or Eureka)
- Client applies its own load balancing algorithm (round robin, random, etc.)
- Examples: Netflix Ribbon, gRPC client-side LB, Kubernetes service meshes (Istio, Linkerd)

**Use when:** You're in a microservices environment and want to remove the load balancer as a bottleneck or single point of failure

**Trigger words:** "microservices", "service mesh", "Kubernetes", "service discovery"

| | Server-Side LB | Client-Side LB |
|---|---|---|
| Complexity | Low (centralised) | Higher (distributed logic) |
| Single point of failure | Yes (mitigated by redundancy) | No |
| Client awareness | Client is dumb | Client is smart |
| Common in | Traditional apps | Microservices, Kubernetes |

---

## Session Draining (Connection Draining)
- Problem: You want to remove a server from the pool (for maintenance or scaling down) but it has active connections
- Hard removal = active users get disconnected mid-session
- Solution: session draining (also called connection draining)

**How it works:**
1. Mark the server as "draining" — stop sending NEW requests to it
2. Let existing connections finish naturally
3. After all connections close (or after a timeout), safely remove the server
4. Timeout is typically 30–300 seconds depending on your use case

**Use when:** Deploying new code (rolling deployments), scaling down, server maintenance
**Trigger words:** "zero downtime deployment", "graceful shutdown", "rolling update", "remove server safely"

---

## Quick Decision Guide

| Scenario | Strategy |
|---|---|
| Identical servers, simple distribution | Round Robin |
| Servers with different capacity | Weighted Round Robin |
| Long-lived connections (WebSocket) | Least Connections |
| Need session stickiness, occasional server changes | Consistent Hashing |
| Route by URL or request type | Layer 7 |
| Raw speed, identical servers | Layer 4 |
| Mixed lightweight + heavyweight requests | L7 with separate server pools |
| Millions of users globally | CDN + GSLB + L4 + L7 combined |
| Prevent abuse / DDoS | Rate Limiting at LB |
| HTTPS at scale | SSL Termination at LB |
| Zero downtime deployments | Session Draining |
| Microservices / Kubernetes | Client-Side LB or Service Mesh |