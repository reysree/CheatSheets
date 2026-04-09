# Databases — System Design Reference

## Overview

Databases are fundamentally about **durability** and **consistency** — they guarantee your data gets saved and stays correct, which caching and load balancing do not.

Two main categories:
- **Relational (SQL)**: PostgreSQL, MySQL — strict consistency, ACID properties, vertical scaling
- **NoSQL**: MongoDB, Cassandra, DynamoDB — horizontal scaling, flexible schema, eventual consistency

**Core decision trigger**: Does this data need to be immediately consistent, or can it eventually be consistent?
- User balances, payments → relational (strong consistency)
- Feeds, comments, logs → NoSQL (eventual consistency is fine)

---

## 1. Sharding

### What is it?
Splitting data across multiple database instances based on a **shard key** (the column used to decide which shard a row goes to).

### Example — Twitter at scale
- Shard by `user_id`
- User 42's tweets → hash(42) → shard 2
- User 100's tweets → hash(100) → shard 5
- On read: hash the user_id, go directly to that shard

### Range-based vs Hash-based sharding

**Range-based sharding**: Shard by ranges of the shard key.
- User IDs 1–1M → shard 1, 1M–2M → shard 2, and so on
- Pros: range queries are fast — "give me all users between 500k and 600k" hits one shard
- Cons: if most data falls in one range, that shard becomes a hot shard

**Hash-based sharding**: Hash the shard key, map the hash to a shard.
- hash(user_id=42) → shard 3
- Pros: distributes data evenly, avoids hot shards
- Cons: range queries scatter across all shards — can't efficiently ask for "users between 500k and 600k"

**Trade-off**: Range-based for good range query performance but uneven distribution. Hash-based for even distribution but bad range queries.

### Consistent hashing
Used to minimize data movement during resharding.

Normal resharding: add a new shard → rehash every key → redistribute most of your data. Painful.

Consistent hashing: imagine the hash space as a ring. Each shard owns a section of the ring. When you add a new shard, only the keys falling in the new shard's section move — the rest stay put. Adding one shard impacts roughly 1/N of your data instead of everything.

### Hot shard problem
When a celebrity user (e.g. 100M followers) hammers one shard with reads/writes.

**Fixes:**
1. Add a secondary shard key to that shard only (e.g. `user_id + timestamp`) — splits that one hot shard further, rest of shards unchanged
2. Add read replicas to that specific shard
3. Cache aggressively (Redis) in front of that shard

### Scatter-gather problem
When you shard by `user_id` but query by hashtag:
- Hashtag "biryani" could exist across users on all shards
- System must **scatter** query to every shard, then **gather** and merge results
- Expensive — you're hitting every database

**Fix:** Maintain a separate denormalized hashtag index table:
```
hashtag → shard_ids where it exists
```
Now query the index first → scatter only to relevant shards.

### Resharding
When data grows and you need more shards, you redistribute data. Operationally painful.
Use **consistent hashing** to minimize data movement during resharding.

### Partitioning vs Sharding
- **Partitioning** is the broader term — splitting data across storage units
- **Sharding** = horizontal partitioning across multiple machines (what we care about in system design)
- **Vertical partitioning** = splitting columns across machines (rare in interviews)

---

## 2. Replication

### What is it?
One **primary** database accepts all writes. Multiple **replicas** (exact copies) handle reads. Every write to the primary gets replicated to all replicas.

### Why it matters
Solves the **read scaling** problem. Does NOT solve the write scaling problem (all writes still go to primary).

### Synchronous vs Asynchronous replication

**Asynchronous replication** (most common):
- Primary writes data and returns success immediately, without waiting for replicas
- Replicas catch up in the background
- Pros: low write latency
- Cons: if primary crashes before replicating, that data is lost

**Synchronous replication**:
- Primary waits for at least one replica to acknowledge the write before returning success
- Pros: guaranteed durability — no data loss on primary crash
- Cons: slower writes — you're waiting for a network round trip to a replica

**Semi-synchronous** (best of both):
- Wait for one replica to acknowledge, then return success
- Most production systems use this — balances safety and speed

### Replication lag
Async replication means replicas may be milliseconds to seconds behind the primary.

**Rule of thumb:**
- Read from replica: analytics, feeds, background jobs — anything tolerating slight staleness
- Read from primary: immediately after a write, or for critical consistency (e.g. payment confirmation)

### Leader election
What happens when the primary goes down?

Replicas need to elect a new primary automatically. This is called **leader election**.

**How it works (Raft algorithm):**
1. Replicas constantly heartbeat with the primary
2. If replicas don't hear from primary within a timeout period, they trigger an election
3. Replicas vote — the replica with the most up-to-date data wins
4. Winner becomes the new primary and starts accepting writes

**Risk — split-brain**: two replicas simultaneously think they are primary and start accepting writes independently, causing data divergence. Prevented by requiring a majority quorum to elect a leader.

**In practice**: PostgreSQL uses tools like Patroni + etcd for automatic leader election.

### Multi-primary replication
Allows writes to multiple primaries simultaneously. Each primary replicates to others.

**Problem**: conflicts — if two primaries write different values to the same key at the same time, which one wins?

**Conflict resolution strategies:**
- Last-write-wins (by timestamp)
- Vector clocks (track causality to resolve conflicts)

Use cases are narrow — distributed caches, specific geo-distributed setups. Generally avoided due to complexity.

### Sharding + Replication together
This is how you scale both reads and writes:

```
Shard 1 Primary → Replica 1a, Replica 1b
Shard 2 Primary → Replica 2a, Replica 2b
Shard 3 Primary → Replica 3a, Replica 3b
```

- Writes distributed horizontally across shard primaries (based on shard key)
- Reads distributed horizontally across replicas within each shard
- Both are **horizontal scaling** (adding more instances, not bigger servers)

**Vertical scaling** = upgrading a single server's CPU/RAM/disk — rarely the right move at scale.

---

## 3. Consistency Models

### Strong consistency
Every write waits for all replicas to acknowledge before returning success.
- Guarantee: reads always see the latest data
- Cost: slow — only as fast as your slowest replica
- Use case: financial transactions, critical inventory

### Eventual consistency
Write returns immediately after hitting primary. Replicas catch up asynchronously.
- Guarantee: data will be consistent... eventually (milliseconds to seconds)
- Cost: reads may see stale data temporarily
- Use case: social feeds, likes, comments, analytics

### Read-after-write consistency
Middle ground: after you write something, you can immediately read it back. But other users may see stale data briefly.
- Used by Twitter, most social platforms

### Real-time leaderboards — special case
Needs every user to see the same score immediately globally.

**Pattern:** Eventual consistency at DB layer + real-time messaging layer on top
1. Score update → write to Redis (fast in-memory store)
2. Broadcast to all connected clients via WebSocket
3. Asynchronously persist to database

Result: Users see consistent real-time state through messaging, not through DB strong consistency.

---

## 4. Transactions and ACID

### ACID properties
- **Atomicity**: All operations succeed, or all fail — no partial writes
- **Consistency**: Data remains valid before and after transaction
- **Isolation**: Concurrent transactions don't interfere with each other
- **Durability**: Committed writes survive crashes

### The distributed transaction problem
Money transfer: debit account A (shard 1) + credit account B (shard 2). Both must succeed or both must fail.

### Two-Phase Commit (2PC)
Coordinator manages a distributed lock across shards:

**Phase 1 (Prepare):**
- Coordinator asks shard 1: "Can you debit account A?" → Shard 1 locks account A, replies "yes"
- Coordinator asks shard 2: "Can you credit account B?" → Shard 2 locks account B, replies "yes"

**Phase 2 (Commit):**
- Coordinator sends commit to both → both execute and release locks

If either shard says "no" in Phase 1 → coordinator aborts entire transaction.

**Problem with 2PC:**
- Locks held during entire two-phase process
- Network slowness or shard downtime = all other transactions touching those rows are blocked
- Kills throughput and availability at scale

**What are locks?**
A lock prevents any other transaction from reading or writing to a specific row until your transaction completes. Scope is row-level — only those specific rows are blocked, not the entire database.

**Distributed locking** = locks that coordinate across multiple independent systems/shards. 2PC is a distributed locking mechanism.

### Compensating Transactions (alternative to 2PC)
No locks. Execute operations independently:
1. Debit account A (shard 1) immediately
2. Credit account B (shard 2) immediately
3. If step 2 fails → trigger a compensating transaction to reverse step 1

Trade-off: More complex, temporary inconsistency possible, but no blocking.

**When to use which:**
- Banking / payments: 2PC acceptable (lower transaction volumes, correctness is paramount)
- High-scale distributed systems: compensating transactions (throughput > locking overhead)

---

## 5. Denormalization and Materialized Views

### What is denormalization?
Storing redundant, pre-computed data in separate tables to make reads fast — intentionally breaking normalization rules.

**Why:** Some queries are too expensive to compute on the fly at scale (scatter-gather, large aggregates).

### Example — hashtag index
Instead of scanning all shards for hashtag "biryani":
- Maintain a separate `hashtag_index` table: `{ hashtag → [shard_ids] }`
- On tweet write: update both `tweets` table AND `hashtag_index`
- On hashtag read: look up index first → scatter only to relevant shards

Write complexity increases. Read speed skyrockets.

### Materialized views
Pre-computed aggregates stored as a separate table, refreshed periodically.

**Example — daily merchant transaction volume:**
- Without: scan millions of transactions on every report request
- With: batch job runs nightly, aggregates all transactions, stores result in `merchant_daily_volume` table
- Report reads hit pre-computed table → instant

### Refresh strategies
| Refresh frequency | Data freshness | Compute cost |
|---|---|---|
| On every write | Real-time | High (adds write latency) |
| Hourly | ~1 hour stale | Medium |
| Nightly | Up to 24h stale | Low |

**Hybrid pattern:** Critical denormalized data refreshes on write. Non-critical stuff refreshes in nightly batches.

---

## Key Interview Patterns

| Problem | Solution |
|---|---|
| Write throughput bottleneck | Sharding |
| Read throughput bottleneck | Read replicas |
| Hot shard | Secondary shard key OR read replicas OR caching |
| Scatter-gather on non-shard-key queries | Denormalized index table |
| Cross-shard atomic operations | 2PC (low scale) or compensating transactions (high scale) |
| Expensive aggregate queries | Materialized views |
| Geographic read latency | Geo-distributed replicas + edge caching |
| Primary database goes down | Leader election (Raft) → replica promoted to primary |
| Uneven data distribution on resharding | Consistent hashing |
| Write conflicts across primaries | Multi-primary replication with conflict resolution |

---

## Concept Relationships

```
Write scale      → Sharding (distribute writes across primaries)
Read scale       → Replication (replicas per shard primary)
Resharding       → Consistent hashing (minimize data movement)
Failover         → Leader election (Raft) — auto-promote replica
Correctness      → Transactions + ACID (2PC or compensating)
Query speed      → Denormalization + Materialized views
Consistency      → Strong (slow, safe) vs Eventual (fast, tolerant)
```

All concepts work together. In a real system you use all of them.