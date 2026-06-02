# HLD
## Phase 1 — Foundation (Week 1)
### 1. Rate Limiter
##### Redis
1. **Cache-aside** -> Store in cache and lookup if expired.
2. **TTL**
3. Redis is 
4. **single threaded** -> Isolation
5. Use Lua for atomic sequences -> Isolation leads to no race conditions.
6. **Hot keys** - cause shard imbalance and latency spikes.
7. Cache introduces **consistency vs latency tradeoffs.**
8. Always discuss **cache stampede** -> Request coalescing, Staggered TTL, background refresh(for hotkeys).
9. **Failure** - fail-open vs fail-closed 
##### BOTEC
- Start with DAU assumption. -> Reqs per day = DAU * (Reqs/user)/day
- (Reqs/user)/day = 
	- light app: 20–50
	- normal app: 100–500
	- chat/feed/video: 1000+
- Average RPS = Reqs per day / 84000 = Reqs * 1.2 * 10^<sup>-5</sup>
- Peak RPS = Avg RPS * Peak multipler(= 5x for normal system. 10-20x for consumer systems)
- Read Heavy -> Reads = 90-95%
- Bandwidth = Size * RPS
### 2. Notification System
1. Always identify the **hot path** and aggressively remove DB calls by using Cache
2. **DB write + queue publish**-> **Transactional Outbox Pattern**.
3. Consistency requirements are **product decisions**-> Explicitly state inconsistencies you're willing to tolerate.
4. Design **different reliability tiers** (e.g., OTP/security vs marketing) instead of one pipeline for everything.
5. Focus on **reads** more than writes.
6. When a user-facing state exists, always ask: **"What happens if the user has multiple devices?"**
7. Your biggest improvement area is **storage/read-model design**.
#### Multi Device Reads
- **Store source of truth in DB**
- User reads notification on Phone -> mark notifications as read.
- Trigger an event on websocket to update the UI for other devices.
### 3. URL Shortener
## Phase 2 — Realtime Systems (Week 2)
### 4. WhatsApp / Chat System
### 5. Uber / Swiggy Matching
![[Pasted image 20260520145128.png]]
#### 6. Live Location Tracking
## Phase 3 — Consistency Systems (Week 3)
### 7. Ticket Booking System
### 8. Payment / Wallet System
### 9. Inventory Management System
## Phase 4 — Scaling + Internals (Week 4)
### 10. Twitter Feed
### 11. Kafka-like MQ
### 12. Redis-like Cache

# LLD
## Phase 1 — Core OOP + Design Patterns
### 1. Parking Lot
### 2. Elevator System
### 3. Tic Tac Toe / Chess
## Phase 2 — Object-Oriented Systems
### 4. Library Management System
### 5. Splitwise
### 6. BookMyShow
## Phase 3 — Concurrency + Realtime
### 7. Pub/Sub System
### 8. Logger Framework
### 9. Rate Limiter
### 10. Kafkalike MQ
### 11. Redislike Cache
### 12. Cab Booking / Food Delivery