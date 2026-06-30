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
```
                                          ┌──────────────────────┐
                                          │   Client Services    │
                                          │ (Chat, Orders, CRM)  │
                                          └──────────┬───────────┘
                                                     │
                                                     ▼
                                    ┌─────────────────────────────┐
                                    │     Notification API        │
                                    │  (Validation, Auth, Rate    │
                                    │      Limiting, APIs)        │
                                    └──────────────┬──────────────┘
                                                   │
                                                   │ Transaction
                                                   ▼
                 ┌────────────────────────────────────────────────────┐
                 │                Notification DB                    │
                 │                                                    │
                 │ Notifications                                      │
                 │ NotificationPreferences                            │
                 │ OutboxEvents                                       │
                 └──────────────────┬─────────────────────────────────┘
                                    │
                                    │ CDC / Outbox Publisher
                                    ▼
                         ┌──────────────────────┐
                         │        Kafka         │
                         │ Notification Topic   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                    ┌────────────────────────────────┐
                    │ Notification Orchestrator      │
                    │                                │
                    │ • Preference Lookup            │
                    │ • Channel Selection            │
                    │ • Online/Offline Check         │
                    │ • Fanout                       │
                    └───────┬───────────┬────────────┘
                            │           │
                            │           │
                            ▼           ▼

               ┌────────────────┐   ┌─────────────────┐
               │ Preference     │   │ Presence Service │
               │ Cache (Redis)  │   │ (WebSocket Conn) │
               └────────────────┘   └────────┬────────┘
                                              │
                                              ▼
                                     ┌────────────────┐
                                     │ Presence Cache │
                                     │ (user→server)  │
                                     └────────────────┘

                            Fanout Channel Events
                                      │
      ┌──────────────┬───────────────┼───────────────┬──────────────┐
      ▼              ▼               ▼               ▼              ▼

┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐
│ Push Topic│  │Email Topic│  │ SMS Topic │  │ WA Topic  │  │ In-App Topic│
└─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └──────┬──────┘
      │              │              │              │               │
      ▼              ▼              ▼              ▼               ▼

┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐
│PushWorker │  │EmailWorker│  │ SMSWorker │  │ WAWorker  │  │ InAppWorker │
└─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └──────┬──────┘
      │              │              │              │               │
      ▼              ▼              ▼              ▼               ▼

  FCM/APNS      Email Vendor      SMS Vendor      WA Vendor     WebSocket
                                                             Delivery

```
### 3. URL Shortener
## Phase 2 — Realtime Systems (Week 2)
### 4. WhatsApp / Chat System
### 5. Uber / Swiggy Matching
![[Pasted image 20260520145128.png]]
#### 6. Live Location Tracking
## Phase 3 — Consistency Systems (Week 3)
### 7. Ticket Booking System
- Since we **cannot tolerate losing data** here, the **source of truth has to be DB.**
- **Waiting Room** - Implement a waiting room with Redis. Return a counter for the user who comes in.
	- Chron job which checks if active users < threshold -> Allow x users.
	- Threshold -> Assume every user generates seat map every 5 sec and locks seat in 30 sec.
	  Get RPS = X/5, Lock RPS = X/30.
	  if we want Lock RPS to be under 1000 then we can allow 30K users.
	  If we want Get RPS to be under 1000 then we can allow 5K users.
- **Service Crash Recovery** - After recovery try to poll the service whose reqs might be missed.
- **DAU is irrelevant** - For every system identify the hot problem and optimize for that.
- **Booking State Modelling** - Model bookings here not seat. This makes payment crashes easier.
- **Lock Expiry** - In case of seat lock(booking=PAYMENT_PENDING) expiry, run a chron job that resets the Lock.
  Index if you want faster DB searches.
### 8. Payment / Wallet System
#### Consistency Models
- Strong consistency	- **ACID Transaction**
- Cross-shard strong consistency - **2PC / Distributed Transaction**
- **External systems involved** - Saga => Eventual Consistency
- **Financial auditability** - *Ledger*
#### Ledger
- We use transaction history as the source of truth.
```
T1 A +1000
T2 B -100
T3 A +500
T3 B -500
```
- When you need to calculate the current balance, you go through the entries in the history.
```SQL
SELECT SUM(amount) FROM LedgerEntry WHERE account_id = A;
```
##### Flow
```
Transfer Request --> Create Transaction with PENDING state 
--> Create Ledger entries (Debit + Credit)
--> Commit
--> Publish event to mark Transaction as COMPLETED
```
### 9. Inventory Management System
## Phase 5 (Add These)

### 10. Distributed Job Scheduler
### 11. Search System
##### Inverted Index
- Store **Word → Documents**
- **Word** -> \[ {*docId, termFrequency, List\<positions>*} \] => posting
##### Searching
- For each word -> Retrieve all the List\<**documents**>.
- Smallest List **intersection** other lists + add the leftover items.
##### Phrase Search
- Add ids to each word.
- Docs with word in similar positions rank higher.
##### Ranking
- Use term frequency, words in title get higher weight. You can store some metric in the postings.
##### DB to use - **Lucene**
- 1B products, 100 terms each -> 100B documents -> **Relational DB is obsolete** -> We need fast reads.
- Words -> Posting is stored in immutable segments like LSM trees.
	- MemTables -> LSM Tables -> Compaction.
- You fetch results from each segment and return the results.
##### Pagination
- **Cursors** - These are metrics which store some metric which filters the results into different pages.
- You return the following so that the UI knows what to query.
```JSON
{
  "results": [...],
  "nextCursor": base64encode("score=92.1,id=123")
}
```

### 12. Dropbox / File Storage
```
API Gateway -> Metadata Svc -> Metadata DB
Client --<Predefined URLs> -> Blob Store
```
##### Upload Flow
- Step 1 - Create an entry for file in metadata DB and mark as UPLOADING
```http
POST /files
{
	"filename": "mymovie.mp4",
	"size":1GB
}
Response - 
{
	"fileId":"123",  
	"chunkSize":"64MB",  
	"uploadUrls":[...]
}
```
- Step 2 - Upload each chunk to provided urls through a POST request.
- Step 3 - Track progress. Every chunk upload updates the metadata DB with entry in *Chunk* table.
- Step 4 - After each update check if # chunk in Chunk table = # chunks proposed. If so mark file as UPLOADED.
##### Download Flow
```http
GET /files/{fileId}
{
  "chunkUrls":[...]
}
```
**Metadata DB** - Usually a Relational DB.
**Device Sync** - Trigger an event through Kafka that refreshes the state of other devices.
**Partial Uploads** - Run a job every few minutes that checks if uploads are still happening, else delete the chunks.
##### Deduplication
- For every chunk store hash - SHA256(chunk)
- If another user uploads the same chunk, check if chunk is present and dont store again.
- Increase the reference counts if the chunk is present in multiple places.
### 13. Recommendation System
#### Content Creation Flow -> Offline
```JS
Video Upload -> Content Service -> DB -> Kafka -> Consumers{Search Indexer, Recommendations, Update Feeds}
```
##### Recommendation System Flow
```JS
Kafka -> Recommendation System Consumer -> Flink/Spark -> Feature Score
{
  "videoId":123,
  "category":"Backend",
  "popularity":0.82,
  "engagement":0.91
}
```
- Flink/Spark convert the video into feature scores -> *how closely it relates to a topic.*
#### User Action Flow -> Offline
```JS
User Action(Click, Link, Sub) -> Kafka -> Flink/Spark -> Update Feature Score
{
  "userId": 1,
  "backendAffinity": 0.95,
  "frontendAffinity": 0.1,
  "sportsAffinity": 0.2
}
```
- Feature Score -> *User's likability to various topics/features* 
#### Model Training
- Based on user actions, generate P(user liking the video).
- {User affinity scores, Video Feature scores} -> P(like)
#### Candidate Generation
- Generate candidates from -> User follows, Users like you watched, Popular in neighbourhood etc.
- You get maybe 5000 videos -> Filter out based on 
	  simple metric = 0.4\*recency + 0.3\*popularity + 0.3\*engagement -> 500 videos
#### Rerank
- Apply diversity -> Choose topics from different topics.
#### Feed Caching
- Pre calculate the feeds and store them in the cache.
- Get the videos from popular sources when the user opens the app -> Merge the two lists.
#### Merging Lists
1. **Simple Concatenation** -> Get lists from various places - simple but no diversity.
2. **Round Robin** -> Get one from each list. Simple, diverse but does not consider quality.
3. **Weighted Round Robin** -> Based on user give weightage to different topics and add videos accordingly.
4. **Global Scoring** -> Every video gets a score based on user and then return the sorted list.
5. **Reranking** -> Dont show multiple videos from same topic consecutively.
## Phase 4 — Scaling + Internals (Week 4)
### 14. Twitter Feed
### 15. Kafka-like MQ
### 16. Redis-like Cache

# LLD
### Tier A (Must Master)
1. ~~Elevator~~
2. ~~Parking Lot~~
3. ~~Movie Ticket Booking~~
4. ~~Wallet
5. ~~Task Scheduler
6. ~~Rate Limiter
7. ~~Pub/Sub~~
8. LRU Cache
9. In-Memory KV Store
10. Splitwise
11. Inventory Management
### Tier B (Strongly Recommended)
12. ATM
13. Vending Machine
14. Ride Matching
15. Amazon Locker
16. Logging Framework
### Tier C (Nice to Have)
17. Library Management
18. Meeting Room Scheduler
19. Auction System
20. LFU Cache
21. Redis-like KV Store with Expiry