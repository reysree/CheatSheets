# Comprehensive System Design: Caching Guide

## Table of Contents
1. What is Caching?
2. Cache Hierarchy Layers
3. When to Use Each Layer (Practical Scenarios)
4. Cache Invalidation Strategies
5. Cache Eviction Policies
6. Cache Warming & Stampede Prevention
7. Distributed Caching Challenges
8. Decision Framework for Interviews

---

## 1. What is Caching?

**Definition:** Caching is about storing frequently accessed data closer to where you need it, avoiding expensive calls to your database or external services every time. It trades a bit of storage for massive speed gains.

**Core Principle:** Each cache layer trades off speed versus consistency.

---

## 2. Cache Hierarchy Layers

### Understanding "Edge"
- **Edge** = geographically distributed locations closer to your users
- The outer perimeter of your infrastructure
- Examples: London, Singapore, Sydney, São Paulo servers instead of just Virginia

### Layer 1: Browser Cache
- **Location:** User's local machine
- **What it caches:** CSS files, images, JavaScript, HTML
- **Control:** HTTP headers (cache-control)
- **Example:** When you revisit a website, images load instantly because they're cached locally
- **Use case:** Static assets that don't change often

### Layer 2: CDN Cache (Content Delivery Network)
- **Location:** Edge servers worldwide (Cloudflare, Akamai)
- **What it caches:** Static assets, sometimes HTML
- **Control:** Cache headers you set
- **Example:** User in Singapore requests image → hits Singapore CDN server instead of Virginia origin
- **Use case:** Global user bases, static content

### Layer 3: Application-Level Cache
- **Location:** In-process memory on your web server
- **What it caches:** Frequently accessed data (user profiles, config)
- **Control:** You write the caching logic
- **Example:** Dictionary in memory: "user 123's profile is this data"
- **Use case:** Read-heavy data that updates rarely
- **Limitation:** Not shared across multiple servers

### Layer 4: Distributed Cache (Redis/Memcached)
- **Location:** Separate service between app servers and database
- **What it caches:** Shared state across servers
- **Control:** Fully controllable
- **Example:** User sessions, shopping carts, leaderboards
- **Use case:** Multiple servers need to share cached data

### Layer 5: Database Cache
- **Location:** Database engine internal memory
- **What it caches:** Frequently accessed data pages
- **Control:** Database manages automatically
- **Use case:** Always present, transparent to developers

**Speed Hierarchy (fastest to slowest):**
Browser → In-process → Distributed → Database → Disk

---

## 3. When to Use Each Layer: Practical Scenarios

### Scenario 1: Simple Blog
- **Data:** Articles that change weekly
- **Solution:** Browser cache + CDN cache
- **Settings:** Cache for 1 week
- **Why:** Database barely gets hit, simple and fast
- **Skip:** Redis, application caching (unnecessary)

### Scenario 2: Social Media Feed (Twitter-like)
- **Data:** Constantly changing feeds, but user profiles stable
- **Solution:** Minimal browser/CDN + Redis for user profiles
- **Settings:** Profile cache TTL = 5 minutes
- **Why:** Feed is personalized/fresh, but profiles accessed thousands of times
- **Skip:** CDN caching for feeds (personalized data)

### Scenario 3: E-commerce (Amazon-like)
- **Data:** Product pages + dynamic inventory + shopping carts
- **Solution:** 
  - Browser/CDN: Product images, descriptions (cache for hours)
  - Redis: Shopping carts (shared across devices)
  - No cache: Real-time inventory
- **Why:** Static content cacheable, carts need persistence, inventory too dynamic

### Scenario 4: Real-time Game Leaderboard
- **Data:** Scores updating every second
- **Solution:** Redis only
- **Settings:** Sub-millisecond lookups required
- **Why:** Can't query database for top 100 players on every request
- **Skip:** Browser/CDN (data changes constantly)

---

## 4. Cache Invalidation Strategies

### Strategy 1: Time-Based Invalidation (TTL - Time To Live)
- **How:** Set expiration timer, data auto-deletes after time
- **Example:** User profile cached for 5 minutes
- **Pros:** Simple to implement
- **Cons:** Potential staleness (up to TTL duration)
- **Use case:** Non-critical data, read-heavy scenarios
- **Implementation:** Redis TTL, HTTP cache-control headers

### Strategy 2: Event-Based Invalidation
- **How:** Delete cached data immediately when source data changes
- **Example:** Price change → immediately delete product cache
- **Pros:** Always fresh data
- **Cons:** Complex implementation, must hook into all update logic
- **Use case:** Critical data where staleness costs money
- **Implementation:** Cache deletion in update workflows

### Strategy 3: Write-Through Caching
- **How:** Update cache and database simultaneously
- **Example:** Player scores 100 points → update both database and Redis atomically
- **Pros:** Cache and database always in sync
- **Cons:** Complex error handling, slower writes
- **Use case:** Real-time data requiring consistency
- **Variation:** Write-behind (cache first, database async)

### Choosing Strategy:
- **Non-critical, read-heavy:** Time-based (TTL)
- **Critical, changes often:** Event-based
- **Real-time, consistency critical:** Write-through

---

## 5. Cache Eviction Policies

### The Problem
Redis has finite memory (e.g., 10GB). When full, which data gets deleted?

### LRU (Least Recently Used) - Most Common
- **Logic:** Delete data not accessed in longest time
- **Example:** User 123 (last accessed 2 hours ago) vs User 456 (10 seconds ago) → delete User 123
- **Use case:** Most traffic patterns (hot data stays hot)

### LFU (Least Frequently Used)
- **Logic:** Delete data with lowest access count
- **Example:** User 123 (2 total accesses) vs User 456 (100 accesses) → delete User 123
- **Use case:** When frequency matters more than recency

### FIFO (First In, First Out)
- **Logic:** Delete oldest cached data regardless of access patterns
- **Pros:** Simple to implement
- **Cons:** May evict hot data just because it was cached early
- **Use case:** Simple systems with predictable access patterns

### Combining with Invalidation
- **TTL + LRU:** Data expires after 5 minutes OR gets evicted if memory full
- **Event-based + LRU:** Manual deletion + automatic LRU safety net
- **Write-through + LRU:** Keep hot data, evict cold data when needed

---

## 6. Cache Warming & Stampede Prevention

### Cache Warming
- **Problem:** Service restarts → empty cache → database hammered
- **Solution:** Proactively load hot data before serving traffic
- **Example:** On startup, cache top 100 products, popular user profiles
- **Implementation:** Startup hooks, background jobs

### Cache Stampede (Thundering Herd)
- **Problem:** Popular cache expires → 1000 simultaneous requests hit database
- **Example:** Product cached for 1 hour → expires → 1000 users request simultaneously
- **Solutions:**
  1. **Locking:** First request fetches, others wait for lock
  2. **Refresh-ahead:** Refresh at 55 minutes instead of waiting for 60-minute expiration
  3. **Probabilistic expiration:** Slightly randomize expiration times

---

## 7. Distributed Caching Challenges

### Single Redis Limitations
- Single point of failure
- Limited throughput
- Geographic latency

### Replication Solution
- **Setup:** Master Redis + Multiple Replicas
- **Pattern:** Writes to master, reads from replicas
- **Benefits:** Distributed read load

### Consistency Challenges
- **Problem:** Master-replica sync lag
- **Example:** Write succeeds on master, hasn't replicated yet → different servers see different data
- **Solutions:**
  - **Strong consistency:** All replicas sync before confirming write (slow)
  - **Eventual consistency:** Replicas catch up over time (fast but risky)

### Failover Handling
- **Problem:** Master crashes → which replica becomes new master?
- **Solutions:** Redis Sentinel, Redis Cluster for automatic failover
- **Monitoring:** Track sync lag, set appropriate TTLs

---

## 8. Decision Framework for System Design Interviews

### Step 1: Identify Data Types
- **Static content:** Images, CSS, JS → Browser + CDN cache
- **User-specific data:** Profiles, preferences → Distributed cache (Redis)
- **Real-time data:** Live scores, inventory → Minimal caching or none
- **Shared state:** Sessions, carts → Distributed cache

### Step 2: Choose Invalidation Strategy
- **How often does data change?**
  - Rarely: TTL-based
  - Frequently: Event-based
  - Real-time: Write-through
- **Cost of staleness?**
  - Low: TTL-based
  - High: Event-based or write-through

### Step 3: Plan for Scale
- **Small app (<1000 users):** Browser + CDN cache only
- **Medium app:** Add Redis for shared state
- **Large app:** Consider Redis replication, cache warming, stampede prevention

### Step 4: Address Consistency
- **Can you tolerate brief staleness?** Eventual consistency
- **Need immediate consistency?** Strong consistency (slower)
- **Monitor sync lag and tune TTLs accordingly**

### Common Interview Questions & Answers

**Q: "Design a cache for a social media platform"**
**A:** Browser/CDN for static assets (images, CSS), Redis for user profiles and friend lists (TTL-based, 5-minute expiration), no caching for real-time feeds (too personalized), event-based invalidation when users update profiles.

**Q: "How do you handle cache consistency?"**
**A:** Choose appropriate invalidation strategy based on staleness tolerance. For critical data, use event-based invalidation. For non-critical data, TTL-based is sufficient. Monitor distributed cache sync lag and set TTLs accordingly.

**Q: "What if your cache goes down?"**
**A:** Circuit breaker pattern - when cache fails, serve directly from database. Implement cache warming for faster recovery. Use Redis replication for high availability.

---

## Key Takeaways

1. **Not all layers needed:** Start simple, add complexity as you scale
2. **Invalidation is hard:** Choose strategy based on consistency requirements
3. **Monitor everything:** Cache hit rates, sync lag, memory usage
4. **Plan for failures:** Cache warming, circuit breakers, failover strategies
5. **Trade-offs everywhere:** Speed vs consistency, simplicity vs reliability

Remember: Caching isn't just about speed - it's about managing consistency and handling edge cases at scale.