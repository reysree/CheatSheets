# CAP Theorem - Complete Reference

## What is CAP Theorem?

CAP theorem states that in any distributed system, you can guarantee only **two of three properties**:
- **Consistency**: All nodes see the same data at the same time
- **Availability**: System responds to requests even if some nodes fail
- **Partition tolerance**: System continues operating when network partitions occur

**Key insight**: In real distributed systems, network partitions WILL happen, so you're really choosing between Consistency and Availability.

## Network Partitions Explained

**Definition**: A network partition occurs when nodes in a distributed system cannot communicate with each other.

**Example**: Banking system with Virginia and California data centers
- Normal state: Both centers sync account balances
- Partition occurs: Network cable cut, Virginia can't talk to California
- Problem: Virginia processes a transaction but California doesn't know about it

**The Trade-off**:
- **Lose Availability**: Virginia waits for California → request hangs forever
- **Lose Consistency**: Virginia processes anyway → California has stale data

## System Types

### CA Systems (Consistency + Availability)
- Single server, no distribution, no replicas
- No partitions possible because there's nothing to partition
- Don't exist in real distributed systems at scale

### CP Systems (Consistency + Partition tolerance)
- Block writes when can't reach all nodes
- Maintains data consistency at cost of availability
- Example: Traditional SQL databases

### AP Systems (Availability + Partition tolerance)
- Let nodes operate independently during partitions
- Accept temporary inconsistency for availability
- Examples: Cassandra, DynamoDB

## Consistency Models

### Strong Consistency
- Every read returns the most recent write
- Nodes must coordinate before responding
- If partition occurs, writes block until coordination possible
- **Trade-off**: Loses availability during partitions

### Eventual Consistency
- Writes succeed immediately without waiting for all nodes
- Other nodes receive updates asynchronously (milliseconds to seconds)
- Temporary window where nodes have different data
- **Trade-off**: Accepts temporary inconsistency for availability

**Why Eventual Consistency Dominates**: Users prefer system response over perfect consistency. Better to get "request processed" than "system temporarily unavailable."

## Quorum-Based Systems

**Core Concept**: Require majority agreement instead of unanimous agreement.

**Formula**: Quorum = (N/2) + 1, where N = total nodes
- 3 nodes → quorum = 2
- 5 nodes → quorum = 3

**How it Works**:
- **Write**: Require quorum acknowledgment before responding
- **Read**: Require quorum agreement on data value
- **Partition handling**: Side with quorum can operate, other side blocks

**Example**: Keypads with 3 regions (Virginia, California, Texas)
- Partition: Virginia+California vs Texas
- Virginia+California have quorum (2/3) → can process bookings
- Texas alone lacks quorum → goes silent

## Conflict Resolution Strategies

When partitions heal, conflicting writes must be resolved:

### Last-Write-Wins
- **Logic**: Latest timestamp wins, other data discarded
- **Pros**: Simple, fast
- **Cons**: Can lose data silently
- **Use case**: Non-critical data where speed matters

### Vector Clocks
- **Logic**: Track causal ordering with version numbers per node
- **Process**: 
  1. Each node increments its clock for writes
  2. Writes tagged with all node version numbers
  3. On sync: compare version vectors
  4. If one is ahead → sequential, later wins
  5. If neither ahead → concurrent conflict → escalate to application
- **Pros**: Detects true conflicts vs sequential updates
- **Cons**: More complex
- **Use case**: Critical data where conflicts matter

### Application-Level Resolution
- **Logic**: Custom business logic handles conflicts
- **Examples**: 
  - Marketplace: "Reject later booking, notify user"
  - Banking: "Never allow conflicting transfers"
  - Social: "Merge friend lists from both sides"

## Real-World Implementations

### Cassandra (AP System)
- Prioritizes availability and partition tolerance
- Eventual consistency with tunable quorums
- Default conflict resolution: last-write-wins
- Can layer vector clocks for sophisticated conflict detection

### DynamoDB (AP System)
- AWS-managed AP system
- Quorum writes with eventual consistency default
- Option for strongly consistent reads (with latency cost)
- AWS handles complexity of conflict resolution

### Kafka (CP for Partitions)
- Partition has single leader handling all writes
- Replicas follow leader
- Leader failure triggers failover
- Prioritizes consistency of message log over availability during transitions

## Design Guidelines for Keypads

### Current Approach
- **POC Phase**: Use Supabase, focus on functionality
- **Philosophy**: Build first, optimize later based on real issues

### Future Considerations
When scaling Keypads, consider:

1. **Listing Updates**: Eventual consistency acceptable?
   - Users can tolerate slightly stale apartment availability
   - Availability more important than instant global consistency

2. **Booking Conflicts**: Strong consistency needed?
   - Double bookings are unacceptable
   - May need quorum-based booking confirmation

3. **Geographic Distribution**: 
   - Multiple regions serving users locally
   - Network partitions between regions inevitable
   - Choose AP architecture with conflict resolution for bookings

### Key Questions for Database Selection
1. Can you tolerate temporary inconsistency in listings?
2. How do you handle booking conflicts across regions?
3. Is user experience more important than perfect data consistency?
4. What's your acceptable window for "eventual" consistency?

## Interview Keywords & Triggers
- **Partition tolerance** → Always required in distributed systems
- **Quorum consensus** → Practical implementation of majority voting
- **Eventual consistency** → AP systems trade-off
- **Vector clocks** → Sophisticated conflict detection
- **Strong vs eventual consistency** → Core trade-off in system design

## Next Steps
1. Complete database fundamentals
2. Build Keypads POC with Supabase
3. Identify real consistency/availability issues
4. Apply CAP theorem to choose production database
5. Implement appropriate conflict resolution strategy