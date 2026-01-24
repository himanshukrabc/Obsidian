# Phase 0: Pre-requisites (VERY light, 2–3 days)
### Learn:
- Client–Server model    
- REST vs RPC    
- Latency vs throughput    
- Stateful vs stateless services    
### Resource:
- **ByteByteGo – “System Design Basics” playlist**      (Only the _introductory_ videos)
⏱️ Time: 2–3 days  
✅ Done when: You can explain these without jargon.
# 3️⃣ Phase 1: Distributed Systems Core (REAL FOUNDATION)
This is where HLD actually starts.
## 3.1 Scalability & Load
### Learn:
- Vertical vs horizontal scaling    
- Load balancing (L4 vs L7)    
- Sticky sessions (why dangerous)    
- Stateless services    
### Resource:
- **DDIA – Chapter 1**    
- ByteByteGo videos on:    
    - Load Balancers
    - Horizontal scaling

⏱️ 4–5 days  
✅ Done when:  
You can answer:
> “Why does stateless scale better?”
## 3.2 Data Storage Basics
### Learn:
- SQL vs NoSQL (not religious — practical)    
- When joins break    
- Read-heavy vs write-heavy workloads    
### Resource:
- DDIA – Chapter 3 (selected sections)    
- ByteByteGo: SQL vs NoSQL    
⏱️ 4 days  
✅ Done when:  
You can justify **why** you chose a DB for a system.
## 3.3 Replication
### Learn:
- Leader–follower replication    
- Sync vs async replication    
- Failover basics    
### Resource:
- DDIA – Chapter 5 (core sections)    
- ByteByteGo: Replication    
⏱️ 4 days  
✅ Done when:  
You can answer:
> “What happens if leader crashes?”
## 3.4 Partitioning / Sharding
### Learn:
- Why sharding is inevitable    
- Hash vs range partitioning    
- Consistent hashing    
- Rebalancing problems    
### Resource:
- DDIA – Chapter 6    
- ByteByteGo: Sharding    

⏱️ 5 days  
✅ Done when:  
You can answer:
> “How do you add a shard without downtime?”
# 4️⃣ Phase 2: Consistency & Reliability (MOST INTERVIEW FAILURES)
## 4.1 Consistency Models
### Learn:
- Strong vs eventual consistency    
- Read-your-writes    
- Monotonic reads    
- Quorums (N, R, W)    
### Resource:
- DDIA – Chapter 5 (consistency sections)    
- Gaurav Sen – CAP theorem (old video)    
⏱️ 5 days  
✅ Done when:  
You can **state what consistency your system provides**.
## 4.2 Failure Handling
### Learn:
- Partial failures    
- Timeouts    
- Retries & idempotency    
- Circuit breakers    
### Resource:
- DDIA – reliability sections    
- ByteByteGo: Fault tolerance    
⏱️ 4 days  
✅ Done when:  
You naturally ask:
> “What happens when this node is slow, not dead?”
# 5️⃣ Phase 3: Performance Patterns (HLD bread & butter)
## 5.1 Caching
### Learn:
- Cache-aside    
- Write-through / write-back    
- TTL trade-offs    
- Cache stampede    
### Resource:
- ByteByteGo: Caching    
- Redis docs (architecture overview)    
⏱️ 3 days  
✅ Done when:  
You explain cache **invalidation strategy** confidently.
## 5.2 Async & Messaging
### Learn:
- Why queues exist
- Kafka vs RabbitMQ (conceptually)
- Ordering & delivery guarantees
### Resource:
- DDIA – messaging sections    
- ByteByteGo: Message queues    
⏱️ 3 days  
✅ Done when:  
You can explain **why async helps scalability**.