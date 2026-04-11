# Async Processing & Message Queues
> System Design Curriculum — Topic 4

---

## 1. The Core Problem: Why Async?

Synchronous processing ties up server threads while slow operations run. Example: a user uploads a video. Transcoding takes 5 minutes. Synchronous = user waits 5 minutes. Async = you return immediately, push work to a queue, and notify the user when done.

**The three gains from async:**
- Better UX (instant acknowledgment)
- Server threads freed immediately
- Independent scaling of producers and consumers

---

## 2. What Is a Message Queue?

A buffer between **producers** (services that create work) and **consumers** (services that process work).

```
[Producer] → [Queue] → [Consumer]
  API           MQ        Worker
```

**Popular options:**

| System     | Best For                          | Key Trait                        |
|------------|-----------------------------------|----------------------------------|
| Kafka      | Event streaming, audit logs, analytics | Durable log, replay support  |
| RabbitMQ   | Task queues, fire-and-forget jobs | Flexible routing, simple setup   |
| AWS SQS    | Cloud-native task queues          | Managed, scalable, no ops        |
| BullMQ     | Node.js background jobs           | Redis-backed, concurrency control|

---

## 3. Decoupling

Producers don't know who consumes. Consumers don't know who produced. They communicate only through the queue.

**Result:** Scale each independently. Add workers when jobs back up. Add API servers when requests spike. No tight coupling between them.

---

## 4. Delivery Semantics

### 4.1 At-Least-Once Delivery
- Message is guaranteed to be delivered
- May be delivered **more than once** (if consumer crashes mid-processing)
- Consumer must be **idempotent**
- Used by: most real-world distributed systems

### 4.2 Exactly-Once Delivery
- Message delivered exactly once, no duplicates
- Requires distributed transactions or two-phase commit
- Expensive — introduces latency and complexity
- **Production reality:** most teams use at-least-once + idempotency instead

### 4.3 Idempotency
Processing the same message twice produces the same result as processing it once.

**Bad (not idempotent):** "Add $10 to user balance" — runs twice = +$20  
**Good (idempotent):** "Set user status to verified" — runs twice = same result

**How to implement idempotency:** Attach a unique idempotency key (e.g. webhook event ID, order ID) to every message. Before processing, check if you've already handled this key. If yes, skip. If no, process and record.

---

## 5. Ordering vs Throughput

| Priority    | Use Kafka (partitions)                          | Use RabbitMQ/SQS                        |
|-------------|------------------------------------------------|-----------------------------------------|
| Ordering    | Strong ordering within a partition             | No ordering guarantees                  |
| Throughput  | Parallel across partitions                     | Fully parallel across all workers       |
| Use case    | Financial transactions, state machines         | Email sending, image resizing, encoding |

### Kafka Partitions Explained

```
Topic: "orders"
  Partition 0: [msg1] → [msg4] → [msg7]   ← Consumer A
  Partition 1: [msg2] → [msg5] → [msg8]   ← Consumer B
  Partition 2: [msg3] → [msg6] → [msg9]   ← Consumer C
```

- Within a partition: **sequential, ordered**
- Across partitions: **parallel**
- If 5 messages are in the same partition → processed sequentially
- If 5 messages are spread across partitions → processed in parallel

---

## 6. Kafka vs RabbitMQ/SQS — When to Use Which

### Kafka
- You need **event history and replay** — messages persist for days/weeks
- New consumers can read from the beginning of the log
- Multiple independent services consuming the same event stream
- Analytics pipelines, audit logs, event sourcing, fintech

### RabbitMQ / SQS / BullMQ
- Fire-and-forget background jobs
- You don't need history or replay
- Simpler, lighter weight, cheaper
- Email sending, image processing, booking notifications

**Rule of thumb:** If you need "what happened" → Kafka. If you need "get this done" → RabbitMQ/SQS/BullMQ.

---

## 7. Core Patterns

### 7.1 The Outbox Pattern

**Problem:** You want to write to your database AND publish a message to a queue atomically. But they're separate systems — if the queue publish fails after the DB write, you're out of sync.

**Solution:**

```
[API Handler]
     ↓
[Single DB Transaction]
  ├── Write actual data (e.g. booking record)
  └── Write message row to outbox table
     ↓
[Background Worker]
  ├── Reads outbox table
  ├── Publishes message to queue/external service
  └── Deletes outbox row on success
```

- DB is always the source of truth
- Worker retries on failure — nothing is lost
- If worker crashes, it picks up unpublished rows on restart

**Keypads example:** When a user books a slot:
1. Insert booking record (status: "pending")
2. Insert two outbox rows: one to notify owner, one to notify user
3. All three in a single DB transaction
4. Worker publishes notifications asynchronously

### 7.2 Dead Letter Queues (DLQ)

Some messages will always fail — malformed data, business logic errors, downstream service down.

```
[Main Queue] → [Consumer] → fails repeatedly
                    ↓
             [Dead Letter Queue]
                    ↓
             [Admin review / manual replay]
```

- Prevents poison messages from clogging the main queue
- Logs failed messages for inspection
- Ops team reviews, fixes the issue, replays the DLQ

### 7.3 Retry with Exponential Backoff

Don't retry blindly. Space retries out to give downstream services time to recover.

```
Attempt 1: immediate
Attempt 2: wait 1 second
Attempt 3: wait 4 seconds
Attempt 4: wait 16 seconds
→ Send to DLQ
```

**Distinguish error types:**
| Error Type       | Example                        | Strategy           |
|------------------|--------------------------------|--------------------|
| Transient        | Network timeout, service down  | Retry with backoff |
| Permanent        | Malformed data, auth failure   | Send to DLQ immediately |

### 7.4 Circuit Breaker

Prevents cascading failures when a downstream service is degraded.

```
CLOSED → failures accumulate → OPEN (stop sending requests for 30s)
                                  ↓
                            HALF-OPEN (try one request)
                           /              \
                     success            failure
                        ↓                  ↓
                     CLOSED             OPEN again
```

---

## 8. Consumer Groups & Scaling

### How It Works

**RabbitMQ/SQS:** Pull model. Worker asks for message, processes it, acknowledges. Queue removes message only after acknowledgment. If worker crashes before ack → message requeued after timeout.

**Kafka:** Consumer groups. Kafka assigns each partition to exactly one consumer in the group. Parallel processing across partitions, sequential within each partition.

```
Kafka Topic (10 partitions) + 3 Consumers:

Consumer A → Partitions 0, 1, 2, 3
Consumer B → Partitions 4, 5, 6
Consumer C → Partitions 7, 8, 9

Consumer A crashes →
Consumer B → Partitions 0, 1, 2, 3, 4, 5
Consumer C → Partitions 6, 7, 8, 9
```

**Rebalancing:** Brief pause (few seconds) while partitions are reassigned. Plan for this in real-time systems.

**Rebalancing strategies:**
- Range: Assigns contiguous partitions to each consumer
- Round-robin: Distributes partitions evenly
- Sticky: Minimizes movement — keeps partitions with same consumer where possible

### BullMQ Concurrency

```javascript
const worker = new Worker('bookings', processJob, {
  concurrency: 5  // max 5 jobs in parallel per worker instance
});
```

Adding a second worker instance: jobs are pulled independently, no disruption to existing jobs.

---

## 9. Kafka as an Event Log — Fintech Example

### The Architecture

```
[User places buy order]
         ↓
    [API Server]
         ↓
[Kafka Topic: "orders"]
         ↓
  ┌──────┼──────┐──────────┐
  ↓      ↓      ↓          ↓
[Matching] [Accounting] [Compliance] [Analytics]
 Engine    System        System      Pipeline
  ↓           ↓              ↓           ↓
[orders DB] [balances DB] [risk DB] [dashboards]
```

- One event written to Kafka
- Multiple independent consumers read the same event
- Each consumer owns its own database and state
- Adding a new consumer (e.g. fraud detection) = zero changes to the trading system

**Why Kafka here and not RabbitMQ?**
- Thousands of orders per second → need throughput
- Compliance and audit → need event history and replay
- Multiple independent services → need fan-out to multiple consumers
- New requirements → just add a new consumer reading from the log

---

## 10. Keypads — Booking Flow with Async

### State Machine

```
User books slot
      ↓
[DB Transaction]
  ├── booking.status = "pending_owner_approval"
  └── outbox: notify_owner, notify_user
      ↓
[Outbox Worker publishes to BullMQ]
      ↓
[Workers send notifications]
      ↓
Owner approves/rejects in UI
      ↓
[Queue event: booking_decision]
      ↓
[Worker sends final notification to user]
      ↓
booking.status = "confirmed" or "rejected"
```

**DB is source of truth. Queue is delivery mechanism.**

### What Happens in Parallel vs Sequential

| Sequential (must be ordered)          | Parallel (independent)                   |
|---------------------------------------|------------------------------------------|
| Check slot availability               | Notify owner                             |
| Reserve slot                          | Notify user (pending state)              |
| Create booking record                 | Send confirmation email after approval   |
| Wait for owner approval               |                                          |

### Stripe Webhook Failure Handling

```
Webhook arrives → check idempotency key in DB
  ├── Already processed → skip (idempotent)
  └── Not processed →
        ├── Process payment update in DB
        ├── Mark webhook as processed
        └── If fails → retry with backoff → DLQ after 3 attempts
```

---

## 11. Rate Limiting

Controlling how many requests a user or service can make in a time window.

**Purpose:** Protect your system from external overload. Reject excess requests with `429 Too Many Requests`.

### Algorithms

#### Fixed Window
Divide time into buckets. Allow N requests per bucket.

```
Window: 0s–1s → 10 requests allowed
Window: 1s–2s → 10 requests allowed
```

**Problem:** User can burst 20 requests in 200ms at the boundary (0.9s + 1.1s).

#### Sliding Window
Track timestamps of all requests in the last N seconds. Count only those in the rolling window.

```
At t=1.5s, window = [0.5s → 1.5s]
Count requests in that range → if ≥ limit, reject
```

**Fairer than fixed window. More memory expensive.**  
**Implementation:** Redis sorted set with timestamps as scores.

```
ZADD user:requests <timestamp> <request_id>
ZREMRANGEBYSCORE user:requests 0 (now - 1 second)
ZCARD user:requests  → if >= limit, reject
```

#### Token Bucket ✅ Most Common in Production
Each user has a bucket. Tokens accumulate at a fixed rate. Each request costs one token. Bucket has a max capacity — excess tokens discarded.

```
Rate: 10 tokens/second
Capacity: 50 tokens

User idle for 5s → bucket has 50 tokens (capped)
User makes 30 rapid requests → uses 30 tokens
User makes 25 more → 20 succeed, 5 rejected
```

**Allows controlled bursts. Enforces long-term limits.**

#### Leaky Bucket
Requests enter a queue and leak out at a fixed rate. If queue fills, new requests are rejected. Smoother than token bucket, less flexible for bursts.

### Rate Limiting for AI Systems

For AI orchestration, **token-based quotas** matter more than request counts.

```
User quota: $1 worth of tokens per 24 hours
Track: cumulative tokens consumed in rolling 24h window
On request: estimate tokens → if would exceed quota → reject
```

This is a **quota system**, not a rate limiter. Different concerns:
- Rate limiting: requests per second
- Quota: total consumption over a period

---

## 12. Backpressure

What happens inside your system when producers are faster than consumers.

**Rate limiting = external protection (from outside world)**  
**Backpressure = internal protection (components protecting each other)**

### How Backpressure Works (Automatic)

```
Producer → Queue grows → Consumer falls behind
                ↓
   Queue signals producer to slow down
   OR consumer pulls fewer messages at a time
   OR auto-scaler spins up more consumer instances
```

**BullMQ:** Set concurrency limit. Worker pulls N jobs, processes them, then pulls N more. Automatically limits throughput.

**Kafka:** Producer can be configured to wait for consumer acknowledgment before sending next batch.

### Monitoring for Backpressure

| Metric             | What It Tells You                                      |
|--------------------|-------------------------------------------------------|
| Queue depth        | How many messages are waiting                         |
| Consumer lag       | How far behind consumers are vs producers             |
| Processing latency | How long a typical message takes to process           |
| Failed message count | How many are retrying or going to DLQ              |
| DLQ size           | How many messages need manual intervention            |

**If queue depth grows + lag is constant:** consumers keeping up, but volume increasing → scale horizontally  
**If lag is growing:** consumers falling behind → add workers or tune concurrency

---

## 13. Bulkheading

Isolating failures so one broken consumer doesn't drag down others.

```
Topic: "orders"
  ├── Matching Engine Consumer   → own thread pool
  ├── Accounting Consumer        → own thread pool
  └── Compliance Consumer        → own thread pool (crashes here)
                                          ↓
                               Only compliance is affected.
                         Matching and accounting keep running.
```

Give each consumer its own process or thread pool. Failure in one doesn't cascade.

---

## 14. Production Gotchas

| Pitfall                         | Reality                                                                 |
|---------------------------------|-------------------------------------------------------------------------|
| Debugging async bugs            | Bug could be in producer, queue, or a consumer that ran hours ago. Use distributed tracing (Jaeger). |
| Eventual consistency surprises  | User books slot, owner doesn't see it immediately. Design UI for "pending" states. |
| Exactly-once complexity         | At-least-once + idempotency solves 95% of real problems at far lower cost. |
| Monitoring only queue depth     | Also monitor consumer lag, latency, failure rates, DLQ size.           |
| Bad partitioning strategy       | Partitioning by wrong key serializes unrelated work and kills throughput. |
| Poison messages                 | One malformed message can crash all workers. Always have a DLQ.        |
| Rebalancing pauses              | Adding/removing Kafka consumers causes brief processing pauses. Plan for it. |

---

## 15. Interview-Ready Mental Model

**Trigger words → concepts:**
- "Scale background work" → Message queue, async processing
- "Don't lose messages" → At-least-once + idempotency + DLQ
- "Event history / replay" → Kafka
- "Fire and forget" → RabbitMQ / SQS / BullMQ
- "Fan out to multiple services" → Kafka consumer groups
- "Atomic DB + queue write" → Outbox pattern
- "Cascading failure" → Circuit breaker + bulkheading
- "Too many requests" → Rate limiting (token bucket)
- "Producer faster than consumer" → Backpressure + auto-scaling

**The hierarchy:**
```
At-least-once delivery
    + Idempotency keys
    + Dead letter queues
    + Exponential backoff
    + Circuit breakers
= Production-grade async system
```