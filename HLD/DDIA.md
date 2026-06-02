## Reliability
**Reliability = correctness in the presence of faults**.
- **Fault** - One component of the system deviates from its desired behaviour.
- **Failure** - The whole system deviates from the desired behaviour.
- **Slow system** is essentially a failed system, even though it might give correct results. Quickness is desired.
##### Hardware faults
- Disk crashes, RAM corruption, power loss, machine restarts.
- Expected and unavoidable at scale.
- Handled via **redundancy** (replication, multiple machines, disks).
##### Software faults
- Bugs, memory leaks, resource contention, slow dependencies.
- Harder than hardware faults because they often manifest as **slowness**, not crashes.
## Scalability
**Scalability = how a system behaves as load increases**.
- Not about absolute speed.
- About whether performance degrades **predictably** as load grows.
**Elasticity = ability of a system to adapt automatically to load(Spinning up new machines)**
##### Load
- described with load parameters -> different for each application.
- Eg- Reads per sec, Writes per sec, no of users etc.
##### Example: Twitter
- Post Tweet(Write)
- Timeline(Read) - problematic -> Each user follows multiple users, so everytime someone tweets, the feed will be updated.
- Approaches - 
	1. **Fan-out on read**: Simply insert the tweet into the feed of each user when user accesses timeline.
	2. **Fan-out on write**: Maintain client side cache of each user's timeline and add the new tweet in the cache. Reads are cheap.
- When a person with less no of followers tweets, we follow approach 2.
  When someone with high no of followers -> Large no of updates due to Fan-out -> Better to fetch the tweet on read.
- Adding this introduces code complexity. You have your feed and then the tweet -> 2 sources of truth.
## Maintainability
A maintainable system minimizes long-term operational pain.
Key aspects:
- **Operability**: easy to run and debug in production.
- **Simplicity**: fewer concepts, fewer surprises.
- **Evolvability**: easy to change safely as requirements evolve.
## Data Models
- A **data model** defines how data is structured, stored, and accessed.
- Most applications are built as layers of Data Models. Application DM <- Data Structures and API DM <- DB DM <- Hardware DM.
### Relational Model
- Stored in tables(relations) and tuples(Rows).
- **Fixed Schema** - Shema on write. *Migration* required to change schema.
- **Joins, Transactions and Constraints** 
- **Consistent Writes but slower reads**
- **Object and Relational Mismatch** between the application objects structure and table data -> Use *ORM*
- **ACID**
#### Used when
- Data has clear structure.
- Relationships matter and are frequently queried.
- Strong consistency and transactions are required.
### Document Store
- Stored as objects(JSON/BSON) or XML.
- **Flexible or schema-on-read** design. -> Supports Heterogenous Data
- **Faster reads, Slower Inconsistent Writes** - *Store related objects together* -> Denormalized so leads to multiple updates. Usually scaled horizontally -> Replication lag, Consensus lag.
- **Indexing supported on all types** - Arrays can be index. This makes update of indices harder.
- **Limited support for joins**
- **BASE**
#### Use when
- **Scalability for read heavy loads** -> *Maintain small documents else slow reads(as small parts of documents are required)*
- Your data is a tree of one to many relationships.
>**Data Locality** - Can do wonders for reads.
### Graph Model
- Used when data is complex and has multiple many-to-many relationships.
- **Dynamic** - Vertices and edges both can store anything.
#### Property Graphs
- Vertex - ( id,  out_edges : List(edges), in_edges : List(edges),  properties : Map<String,String>)
- Edge - ( id,  tail_vertex : vertex, head_vertex : vertex,  properties : Map<String,String>)
- Can be thought of as 2 relational tables with JSON properties field.
#### ❌Triple Stores
- Store tree values which will indicate edge, node or property.
 - \_:vertex_name  
 - a -> relation
 - :property/relation
 ```Java
 _:lucy a :Person.
_:lucy :name "Lucy".
_:lucy :bornIn _:idaho.
_:idaho a :Location.
 ```
### Normalization
- Any data point will be stored just once.
- Leads to 1 to many and many to many relationships.
- Common to relational data bases.
- Advantages - **No duplication**, **Easy writes - singular place for updates**
- Costs - **joins**, **poor locality for Read-heavy workloads**
### De-Normalization
- Duplicate data to optimize reads. 
- Common in document stores.
- Advantages - **Faster reads**, **Better locality**
- Cons - **Slow writes**, **Inconsistency**
## Data Storage and Retrieval
### Basic DB Mechanics
- db_set() -> appends key value pair at the end of a document. -> **Great performance**
  db_get() -> Looksup the most recent value of the key and returns it. -> **Bad performance**
- Most databases use this approach. There is an internal file(log) onto which you append your data.
#### Why append only DB?
1. Appending and Compaction work well together.
2. Concurrency and Crash Recovery are simpler if file values are immutable.
### Index
- Additional metadata to ensure fast searches on the keys.
- One index for each way you want to query the DB.
- Each index incurs loads on writes. Each write will need to rearrange the index.
### Indexing
#### Hash Maps
- Map<Key, Offset> -> Offset where data for a key can be found.
- **Hashmap stored in memory** -> Works for small no of keys, write-heavy system.
- **Compaction** - 
	- Logs are broken into *segments*. Go through segments in reverse and create a new segment with latest values and update hashmap.
	- The old segments are discarded. This is called compaction.
- **Search** - look into the memory hash map.
- **Delete** - append an entry stating the value is deleted. When next compaction happens, the key will be discarded.
- **Crash Recovery** - Rebuild the hash map by scanning the current segment.(*Expensive*)
- **Partially Written Records** - Add checksums to ensure the value written is correct. Checksums will be created before write happens.
- [ ] **Range Queries** - not efficient.
#### LSM(Log-Structured Merge) Tree
- Has 3 components - WAL, MemTable and SSTables -> **Fast writes tradeoff with read amplification**
- *Sorted String Tables* - On disk DS where the index of the table is sorted.
- **Writes** - First to WAL then to an in memory table implemented using self balancing trees(AVL/Red Black Tree).
- **Memory Full** - Data flushed onto a SS Table.
- **Range Queries** - Find the SSTables within the range. Get data and merge with in-memory data.
- [ ] **DB Crashes** - In memory writes -> **Write-ahead Log(WAL)** -> Key,Value pair is first appended to log to prevent data loss in case of DB Crash.
- [ ] **Easy Merges/Compaction** - Simple merge sort algorithm to be used.
- ==Lucene==-Used by Elasitc Search uses similar architecture. Key-word, Value-Id of Doc. 
##### Making Reads Faster
- Reads first have to go to MemTable then to all the SSTables.
- **Sparse Indexing** 
	- If no of keys is large in a SSTable -> Searching the index is time consuming. 
	- Store offset of a few indices per block -> Search in the block your key lies in.
- **Bloom Filters**
	- To search you need to go to every SSTable until you find the key as SSTables are not sorted.
	- Bloom filters -> whether a key *does not exist* in a SSTable with *some false positives.*
##### Bloom Filters
- A **Bloom filter** is a space-efficient probabilistic data structure used to test **whether an element exists in a set**.
- It provides some false positives.
- Uses - *bit array of size **m***, ***k** independent hashing functions*.
- **Insert** - Run hashing functions and set the output bits to 1.(Hash1 gave 2 -> bit[2] = 1).
- **Lookup** - Run hashing functions -> If atleast one index is 0 -> Value not present.
- **Deletion** - Use counting bloom filters -> Increment the array by 1 for each insert and decrement for delete.
##### Compaction Techniques
- **Size tiered**- Newer and smaller SSTables are merged into older and larger ones.
- **Leveled** - SSTables are split based on range to make them smaller and managable.
#### B-Trees
- Stores key value pairs sorted by keys.
- Break the DB down into page size units and each node has a page assigned to it. 
- Each page is responsible to store values for a range of keys
- **Leaf node** - contains exact value for each key. **Other nodes** - Starting Key for child page, reference to page.
- **Branching Factor** - The number of references to child pages a node can have. Depends on size of keys and reference to pages. *High branching factor -> Low height of tree*
- **Updates** -> Find leaf node containing the key -> Update the page -> If there isnt enough space, you split the range in two, and recursively update the parent nodes. 
- **DB Crashes** 
  -> If the DB crashes and a page split happens, some nodes are not yet updated, you end up with a corrupted index.
  -> **Write Ahead Log(WAL)** - Any update must be first written onto a log file.
- **Concurrency** - Multi threads can work using special locks called latches.
##### B-Tree Optimizations
- A modified page is written on a different location and all its references are updated. Any resulting new pages updates will also be recreated. This helps in concurrency control.
- Store abbreviated keys(as the entire key is not required to act as boundary). This increases branching factor.
- Store pointers to prev and next sibling pages.
- Try to get leaf pages which are sequential so that you dont have to wait for HDD to rotate through the disk.
#### B-Trees vs LSM Trees Trade-offs
- LSM Trees - faster write but slower reads.
- B Trees - faster reads but slower writes(WAL, multipage update).
- **Write Amplification** -
	- Write in DB leads to multiple writes on the disk(WAL in BIndex and Compaction in LSM). 
	- Direct performance overhead in write heavy applications.
	- LSM Trees are faster in writes as they have lower Write Amplification. It happens only when compaction happens.
- **Compaction** - 
	- LSM Trees use memory better as compaction produces smaller files on disk compared to fragmented B-Tree pages.
	- Some requests will wait while the expensive compaction operation is completed -> *Tail Latency*
	- Disk write bandwidth needs to be shared between the incoming writes and compaction threads.
	- If write throughput is high -> compaction will not keep up -> lookup old uncompacted segments.
#### Secondary Indexes
- Another key value index but keys here are repeated.
- To make the index unique pair it with the key of the primary index.
#### Clustered Index
- The indexed row is directly stored in the index. So reads are faster. However Writes are slower.
- **Covering Index** - You store only some columns.
#### Multi Column Indexes
- If queries have to search on multiple dimensions together then our strategies will not work.
- Eg- Get all the data points with x,y in a rectangle.
- Translate the 2D data into 1D data.
#### Full Text Search
- Text Search needs fuzzy matching(similar keys need to be fetched).
- Include synonyms, Ignore grammatical errors. 
- To cope with grammatical errors, use edit distance(no of characters that differ from the actual word)
### In Memory DBs
- Some situations, DB is small enough to fit entirely in memory. So the In memory DB distributed across servers.
- **DB Crashes** - Take backups in a log or store a replica in another server.
- **Performance Advantage** - Not due to in memory caching(Normal DB pages are also cached by the OS and are in memory). It is because the in memory DS does not need to be encoded onto the disk.
- **Memory Restrictions** - Use page swapping. Least recently used page in the DB is swapped to disk and reloaded on invocation.
### DB for Analytics
- Analysts usually scan over large number of records to get some aggregation. ->(OLAP - Online Analytic Processing)
- Applications usually fetch a few rows and update them. ->(OLTP - Online Transaction Processing)
#### Data Warehouse
- Different from our usual DB. Implements the star/snowflake schema.
- **Fact Table** - central table. Logs all the events happening in the application.(Eg- sales, clicks etc.)
- **Dimension Tables** - Fact Table will have Foreign keys to Dimension Tables. These tables will contain the actual data.
- **Sub Dimension Tables** - More normalized tables(Dimension tables are split up).
- Fact Table + Dimension Table - Star Schema.
  Star Schema + Sub Dimension Table - Snowflake Schema.
#### Column Stores
- Fact tables are very large and usually analysts dont require more than 3-4 columns at once.
- Fact tables usually store data grouped by columns, instead of rows.
- This makes queries faster as the DB grouped by rows will first load all the rows and then filter which is slow.
- Column compression - distinct column values will be much lesser than the number of rows.
	- Store a bitmap for each column value - 0100101.. -> 1 indicates this row has this value.
	- Run-length encoding - (1 zero, 1 one, 2 zeros, 1 one ...) -> even smaller.
## Encoding and Evolvability
### Compatibility
- **Backward compatibility**: New readers can read old data.
- **Forward compatibility**: Old readers can read new data.
- **Encoding**: Convert object to bytes. **Decoding**: reconstructs object from bytes.
- Encoding defines how schema evolves and what compatibility you support.
### Problems with XML, JSON, CSV
- Cannot reliably distinguish numbers from strings.
- JSON:
	- No int vs float distinction
	- No precision guarantee
- No native binary support.
- Optional Schema
### Binary Encoding with Schemas
#### Thrift & Protocol Buffers
- **Schema defined beforehand**.
- Fields identified by **tag numbers**.
```JS
message Person {   
	required string user_name = 1;   
	optional int64 favorite_number = 2;   
	repeated string interests = 3; 
}
```
- Encoding uses tags, datatypes, lengths, and values.
- *repeated* replaces arrays, *required* fields are dangerous (deprecated in modern Protobuf).
#### Schema Evolution Rules
- New fields - **Optional/Have Defaults** + **Unused Tag Numbers**
- **Required fields - cannot be removed**
- **Tags - never reused**
- Datatype changes - Allowed **only if datatype-compatible** (e.g., int32 → int64)
**Limitation**: Evolution relies on **manual discipline** and static contracts.
### Avro
- **Schema stored separately from data.**
#### ❌Schema languages
- **Avro IDL** (developer-friendly)
- **JSON schema** (used at runtime)
```Js
record Person {
	string userName;
	union { null, long } favoriteNumber = null;
	array<string> interests;
}
{
	"type": "record",
	"name": "Person",
	"fields": [
		{"name": "userName", "type": "string"},
		{"name": "favoriteNumber", "type": ["null", "long"], "default": null},
		{"name": "interests", "type": {"type": "array", "items": "string"}}
	]
}
```
#### Encoding
- Encoded as: datatype → length → value
- Decoder needs - **Writer schema and Reader schema**
- Decoding - **Fields are matched by name. Missing fields get defaulted**
#### Schema Evolution
- **New fields** must have default values (for backward compatibility - *new reader, old writer*).
- **Removing fields** allowed if default exists (for forward compatibility - *old reader, new writer*).
- **Rename Field** - use aliases.
- **Datatype changes** - allowed if compatible.
#### Avro Use Cases
- **Within-system communication** (schema registry available).
- Large files with identical schema -> Use **Object Container Files** (schema stored once).
- Databases with individual records -> Store schema version with each record.
#### Dynamically Generated Schemas
- Old data remains valid after schema changes.
- No data migration needed if evolution rules are followed.
- Protobuf/Thrift require manual tag management.
- Avro avoids code generation → better for dynamic languages.
### Merits of Schemas
- Compact binary encoding.
- Enables backward and forward compatibility.
- Code generation helpful for statically typed languages.
### Dataflow through DB
- Data written by old writers needs to be read by new readers which are of newer versions.
- Thus DB needs to enforce strongest compatibility requirements.
- DB Schema evolution is harder than API schema as DB data needs to be readable forever.
- DBs are often the **tightest coupling point of a system**.
### REST vs SOAP vs RPC
#### REST (HTTP + JSON)
- REST is stateless. Resources are identified by their URLs.
- REST prioritizes **simplicity over strict contracts**
- **Pros** - Human readable, Easy to debug, Works well across org boundaries
- **Cons** 
	- Weak schema guarantees(No Schema is imposed), 
	- JSON encoding limitations
	- Ambiguous error handling
#### SOAP(Simple Object Access Protocol)
- XML-based, strongly typed.
- **Pros** - Strong contracts, Built-in security & tooling
- **Cons** - Verbose, Different vendors may have different implementations -> Difficult to evolve.
> SOAP is **rigid but predictable**
#### RPC (Remote Procedure Call)
- Call a remote function as if it were local
- Uses encoders like Thrift / Protobuf / gRPC
- **Enforces strong schemas**.
- **Pros** - Compact encoding, strong schema, high performance.
- **Cons** 
	- Network failures feel like local failures.
	- Tight coupling with the called function's implementation.
	- Versioning issues.
#### When to Use What
- Public APIs - REST.
- Internal Fast Performance APIs - REST
- Legacy Enterprise - SOAP
## Replication
- Keeping copy of the same data on multiple machines that are connected over the internet.
- Advantages - 
	- Reduce latency by having clones geographically close to the users.
	- **Failure tolerance** - system continues working even when a system fails.
	- **High Read Throughput** - Many replicas -> faster reads.
### Leader-Follower Replication
- One **leader** handles writes. Other **followers** handle reads.
- Eg - MySQL, PostgresSQL.
#### Synchronous Replication
- Leader waits for replication on followers before confirming write.
- *Strong durability* - No data is lost. **Slow Writes**
#### Asynchronous Replication
- Leader writes and confirms without waiting for replication. -> **Fast Writes**
- **Issues** 
	- *Durability*- leader fails -> un-replicated data is lost.
	- *Replication Lag*
#### Setting up new followers
- DB data changes regularly -> cannot be directly copied.
- **Approach**-
	- Take consistent snapshots of the leader -> Copy data from snapshot to the new follower.
	- Copy log changes from leader until the replica has caught up.
#### Handling Node Outages without downtime
##### Follower failure
- Follower keeps log of all data changes it receives -> Start from last processed entry until you catch up.
##### Leader failure- **Failover**
1. Determine the Leader has failed.
2. Elect Leader based on Consensus.
3. Redirect Writes to new leader.
###### **Issues**
- **Replication Lag** - If a lagged node is elected -> *lost writes*
- **Split Brain** - New leader elected but old one comes back online and still assumes it is the leader.
### Replication Logs Implementation
##### Statement based Replication
- Logs have SQL statements.
- **Issues**-
	- Non deterministic function(*NOW() or RAND()*) will generate different values.
	- If multiple txn execute concurrently, where clause will give different results on different node(Data change time varies).
##### Write Ahead Logs
- Before write on DB, the exact bits to be changed on disc are logged.
- **Issues**-
	- Tightly coupled to DB storage(What if follower in on a different version of DB which stored data in a different place).
##### Logical Logs
- Exact Row details logged.
- **Insert** - Contains all the values of the row.
  **Delete** - Contains information to uniquely identify the row to be deleted.
  **Update** - Contains information to identify the row and all the column values to be changed.
- Easier to consume and works across DB versions
- Easy for other applications to consume like Kafka, Cache etc.
##### Trigger based Replication
- Trigger lets you run a custom code that transforms the logs and then send it to the replicas.
### Replication Lag
- **Replication lag** = delay between leader write and follower update
#### **Issues** due to Replication Lag
##### Reading your own Writes
- Inconsistency if you read the data you just wrote.(**Read-after-write consistency**)
- **Solutions**-
	- **Session Stickiness** - Route all requests to the same replica(In multi leader).
	- Leader to serve reads for some time after a write.
##### Monotonic Reads
- Refresh routes the request to a follower with more lag than the previous follower. -> **Updates vanish**
- **Solution** - **Session Stickiness** Make sure that the user always reads from the same follower. *If this follower crashes, user blocked*
##### Consistent Prefix Reads
- Violations in causality -> Request B was sent after A but processed earlier.
- **Solution** - Store related data in the same shard. => Causality is maintained as the same node will service such requests.
#### Solutions to Replica Lag
- The problem is if the lag is too much.
- Use transactions - Ensure all your actions are executed. Although they are too expensive for distributed systems.
### Multi Leader Replication
- More than one leader.
- **Use cases** - 
	- Multi-data center operation - You need one leader per geographical location.
	- *Clients with offline operation* - Eg- Calendar where the app operates even when offline. Then each client has its own local DB which in itself is a leader whose changes are synced once online.
#### Write Conflicts
- Two leaders write the same records.
- **Conflict Detection** - 
  *Synchronous Detection*- Conflict is detected at a later as the system does not wait for writes to replicate.
  *Asynchronous Detection*- System waits for writes to be copied over to all the leader.
- **Convergence Mechanisms** - 
	- [[#Last Write Wins]]
	- **Conflict store DS** - Records all conflicts and resolves them when read using custom logic/human intervention.
### Leaderless Replication
- All nodes are allowed to accept read an write requests.
- **Stale Data**- If a node goes down and then comes back up, it might lag behind others -> you get stale response.
#### Handling Inconsistency
- **Read Repair**- Client makes read requests to all nodes. For the stale values it receives, it updates the nodes.
- **Anti-entropy process**- Checks the values stored across nodes per record and updates nodes where stale value is present.
- If anti entropy process is not there, some lagged data may never be updated.
#### Quorum based systems
- When a read/write request is made, request is routed to all nodes.
- Each key is hashed and a few nodes are responsible for its quorum.
- Let n = replicas, 
  w =#write acknowledgments after which write is successful, 
  r =#read acknowledgments after which read is successful
	 Then If w + r> n, we expect to get fresh values every time => At least one node will have updated value.
- If fewer than w or r nodes are available, writes and reads are blocked.
- **Write heavy systems** - Smaller w, larger r.
  **Read heavy systems** - Larger w, smaller r.
  **Eventually Consistent systems** - w=1, r=1 -> high availability
##### Sloppy Quorum
- Reads and writes may go to any reachable nodes(if designated nodes are down).
- **Hinted Handoff**- Once the node is up, the updates are handed off to it.
- Increases availability but even when w+r>n, you might get stale values.(as the node having the write is down)
##### Limitations of Quorum
- **Concurrent Writes** - Two writes succeed on different replicas, no consensus on which value is fresh.
- **Replication lag** - Only n-w replicas are updated. For the write to propagate there is replication lag. If r nodes are queried for read, they will still give stale data.
### Concurrent Writes
##### Last Write Wins
- Store timestamp for each write, take the latest and discard rest.
- **Data Loss** - All except one client will have data loss.
- T1 is run, write, dependent Q1 run, T2 write happens -> dependent Q2 fails => T2 write should have waited.
##### Version Number
- Used to detect conflicting writes.
- DB stores values with version numbers. Any reads are returned with version number.
- So if two different writes have the same version number but a different value -> Conflict detected -> Resolve/Last Write wins.
- Works well with single leader replication.
```Java
C1 writes A -> (A,1)
C2 writes B -> DB reads v=1, (B,2)
C1 writes C -> DB reads v=2, (C,3)
```
##### Version Vectors
- Used in multi-leader replication.
- Writes are stored in replicas with version vectors.
- When replication happens, conflicting writes are identified based on the version vector.
- Version number is stored per replica. The set of replicas queried will return a vector of version numbers.
- If for 2 vectors u,v. Then u>v if ui>=vi for all i and ui>vi for at least one i.
- If you cannot establish causality -> Resolve conflict.
```Java
Initial State - ("X", [A:0, B:0])
Write at A - ("A1", [A:1, B:0])
Write at B (after seeing A’s write) - ("B1", [A:1, B:1])
When you read vB > vA so value from B is newer.

If concurrent writes at A and B - 
V1 = [A:1, B:0]
V2 = [A:0, B:1]
Have application code deal with this.
``` 
## Partitioning
- **Scalability, Higher Throughput and Parallel Reads/Writes**.
- **Skewed Partitions**- Partitions where one partition handles more load than others. This partition is also called the *hot spot*
### Partition Strategies
#### By Key Range
- Each partition holds a range of keys. Internally, the range of keys sorted to **support range queries.**
- **Issues**- *Hotspots due to choice of keys* -> Some keys are hotter than other. Similar keys may be accessed in the same fashion.
- **Fix** - [[#Key splitting]]
#### By Hash Key
- Hash gives different values even if keys are similar -> fair division irrespective of choice of key.
- **Issue** - Range queries are difficult.
- **Compound primary keys** - You use one column for partitioning. Inside the partition the records are sorted -> like a secondary index.
### Key splitting
- Sometimes despite using hashes, hotspots may arise -> **Reads and writes to the same key** can overload a partition.
- Prefix/Suffix a 2 digit random number to the **hot key** -> key split into 100 keys -> split across partitions.
- Reads have to do additional work. You need to keep track of such keys.
### Secondary Indexes
#### Local Indexing
- Called **gather/scatter** -> Each partition has its own secondary indexes.
- **Expensive** - Each partition has to be queried.
#### Global Secondary Indexing
- Global index of the column which is partitioned independently across nodes.
- **Read** -> Go to partition storing index range with key. Then go to the partition which has the value of key. -> *2 Reads* -> *Faster*
- **Writes** -> Update the partition and the GSI. -> *2 writes* -> *Slower*
### Rebalancing Partitions
- When nodes are added/removed -> data must be repartitioned.
- Rules -
	1. Balanced distribution.
	2. Avoid downtime.
	3. Minimal data movement
#### mod N
- Redistribute the requests using the remainder of mod n(#partitions).
- **Con** - Most keys will have to move if n changes.
#### Large number of partitions
- Have many small partitions and every node stores multiple partitions.
- If a node fails or new node arrives -> Move few partitions form one node to another.
- **Less \#partitions** -> Rebalancing becomes a problem. 
  **High \#partitions** -> partition management overhead is too large.
#### Dynamic Partitioning
- Partitions are split and merged based on partition size.
- You start with a min number of partitions to ensure multiple nodes are available to server requests.
### Request Routing
- As partitions are rebalanced, how does the client/routing guide know which IP/port to route the request to?
- **Service Discovery** - Any service on the network may keep changing the machine its running on. Tracks where which service is located.
1. **Client-side Routing**
	- Client knows partition logic, sends request to correct server.
	- **Pros** - Low latency, scales well.
	- **Cons** - Client must track all partitions.
2. **Routing via a coordinator**
	- Client contacts a coordinator which then forwards. The coordinator becomes a bottleneck.
	- **Pros** - Simple client, internal changes are not revealed to the client
	- **Cons** - Extra network hop, bottleneck coordinator
3. **Internal Routing**
	- Client talks to a load balancer, Load balancer forwards to a node
	- Node routes internally if needed
	- **Pros** - Simple clients, centralized control.
	- **Cons** - Extra hop, Load balancer is a single point of failure.
## 🥊Transactions
- Group of reads and writes -> If even one of them fails, all the writes are rolled back.
### ACID
If any of these conditions are violated, DB will rollback.
- **Atomicity** - All operations succeed or none happen. Implemented via WAL.
- **Consistency** - DB constraints must hold. Application responsible for business level consistency.
- **Isolation** - Concurrent txns must execute as if they executed serially.
- **Durability** - Once committed -> data persists through crashes. Implemented using Logs.
*Leaderless replication works in a best effort basis where DB will try to revert on txn failure but there is no guarantee*
#### Single Object Transactions
- Txn modifies/reads only a single DB object -> Consistency and Durability handled by DB.
- Atomicity is implemented use Logs for crash recovery.
- Isolation is implemented by using a lock for each DB object.
### Isolation Levels
#### Read Committed
- Guarantees - 
	- **No dirty reads** - Txn cannot read uncommitted data.
	- **No dirty writes** - Txn cannot update uncommitted data.
- Implementation - 
	- **Write locks** - prevent dirty writes. 
	- DB keeps **committed value + uncommitted value**. When a new txn reads, it gets old value.*Read Locks slow down reads.*
- Issues -
	- **Read Skew** - Txn changes 2 objects but read on both happens between the two writes -> Data is inconsistent.
	  Eg- Money transfer -> Money deducted -> Inconsistent Read -> Money credited
	- **Non Repeatable read** - Same row read twice in txn but yield different results.
#### Snapshot Isolation
- **To resolve read skew, Non Repeatable read** - Ensure each txn reads from a consistent DB snapshot(all data was committed).
##### Implementation - Multi Version Concurrency Control(MVCC)
- *readers never block writers, and writers never block readers.*
- Each txn gets a ID and DB stores multiple versions of modified objects indexed by TxnID.
- *Visibility Rules*-
	- Ignore writes by uncomitted/aborted txn.
	- Ignore writes by txn started later.
- *Indices* - have all the versions of the key available as well.
#### Concurrency Anomalies
##### Lost Updates
- 2 Txn perform (read-modify-write cycle) => T1 reads X=10 ->T2 reads X=10 ->T1 writes X=25 ->T2 writes X=15 =>Update of T1 is lost.
- The problem here is if serially execured, T2 should have read X=25 and made changes accordingly. That is the **lost update**.
- **Solutions**
	- **Atomic read-modify-write** - Use lock on db object for the entire (read-modify-write) cycle.
	- **Compare and set** - New value is only updated if the value in DB matches the value in snapshot.
	- **Automatic lost update detection** - DB detects **write-write conflict** and aborts one transaction.
##### Write Skews/Phantoms
- Generalization of the lost update problem.
- T1 reads A,B -> T2 reads A,B -> T1 modifies A -> T2 modifies B. Together updates on A,B violate an invariant(A.x or B.x must be 1).
- Example - atleast one doctor must be on call but both log out simultaneously.
- *There is no write write conflict.*
- Eg- atleast 1 oncall doctor required. A and B are there. A,B trigger logout -> sees count = 2 -> both log out.
- **Resolution** -
	- **Predicate Locks** - 
		- Create a lock out of conditions of the read query. DB will prevent any query(read write or update) which satisfies condition.
		- Acquire the lock when reading the value and release on txn completion.
		- *Bad Performance* - check every query.
	- **Index-range Locks** -
		- If the condition has a range query, have a lock for the range on the index. 
		- This cancels out more queries than the predicate locks but has better performance.
		- *Better performance*
	- Can be resolved through **Serializable Isolation**
### Serializabile Isolation
- Highest level of isolation.
#### Actual Serial Execution
- Single threaded DB -> all txns execute serially.
- **Cons** - 
	- All working data must fit in memory.
	- DB txn should be short else other txns are held up. -> *No possible for interactive applications*
#### Two Phase Locking (2PL)
- *Readers and writers can both block each other*
- Locks have 2 different modes - *shared mode* and *exclusive mode*
	- **Read** -> get shared mode. **Write** -> get exclusive mode.
	- Any lock acquired must be *held till the txn is complete*.
- **Cons** - 
	- **Deadlocks** - run process to detect and resolve deadlocks.
	- **Bad Performance** - Overhead of acquiring all these locks.
	- **Low Concurrency** - All queries block each other -> most queries are waiting.
#### Serializable Snapshot Isolation (SSI)
- Txn execute with sanpshot isolation.
- **Read-Write(rw) dependency** - T1 reads A -> T2 writes A. ***T1 --rw-> T2***
- The problem occurs when a rw-dependency cycle forms.
- **Implementation**- 
	- Track **read-write dependencies** 
	- On commit, detect **rw-dependency cycles** and abort one txn.
- **Pros** - No blocking of txns.
- **Const** - Leads to retries of txns which has bad performance at high load.
## Partial Failures
- Some componenets fail while others keep working.
- Partial failures are *non deterministic*, i.e., system may sometimes work or fail.
#### [Network Failures]([[Networking Basics]])
- Async packet switched networks -> *Packet loss*, *Network delays*, *Congestion Control delays*
- Impossible to tell if another node is working in case of no response.
### Failure Detection
- Nodes detect failure using timeouts -> unreliable due to clock inaccuracy.
- Detection is *probabilistic* => Too short timeout -> false positives, Too long timeout -> slow recovery
### Unreliable Clocks
- Computer clocks are inaccurate and impossible to synchronize across devices.
- *Clock Drift*, *Network delay in sync packets*, *Clock adjustment inconsistency*
#### Time of Day Clocks
- Returns the current time and date. Synced with the NTP Protocol.
- **NTP Protocol** - sends network packets that syncs the time based on consensus of a group of servers.
- **Leap Seconds** - Clock drifts are synced -> Time on clock might skip to future/past times.
#### Monotonic Clocks
- Used to measure continuous intervals of time. Eg- timeouts or durations. 
- **Clock Skew**- hardware clock drifts due to temp, design -> NTP can mitigate this by syncing skew.
- **Comparision**- Cannot compare monotonic clocks of two nodes(clock drift)
#### Clock Syncronization and Accuracy
- Syncronization is difficult -> *Hardware clocks drift, NTP packets face delays.*
- It is better to use [Logical clocks]([[#Lamport Timestamp]]) / [Version Vectors]([[#Version Vectors]])
### Processing Pauses
- Reasons for Pauses 
	- Garbage Collection
	- Page Faults(load from disc)
	- Context Switching/Scheduling(responding process on hold)
	- Network delays
- **Problems**
	- **Unreliable Timeouts**- Process paused for 2 seconds but timeout is 0.5s -> Wrongfully declared dead.
	- **Unsafe distributed Locks**- Lock lease lasts for 5s, paused for 10s -> Declared dead -> New Lock Holder -> Unpaused -> 2 Holders
		- **Split Brain** - When this happens in leader elections. Leads to lost writes.
#### Fencing Tokens
- Whenever a lock is given, an incremental number called fencing token is issued with it.
- Whenever a reacquire request arrives, it includes this token. If it is the latest token lock is issued.
### Byzantine Faults
- Fencing token assumes that the nodes are unreliable but honest. When nodes lie, it becomes a problem.
- **Byzantine Fault** - There are n nodes, some of which lie. Lying nodes are unknown.
- **Byzantine fault-tolerant** - continues to operate correctly even if someof the nodes are malfunctioning and lying.
## Consistency and Consensus
- **Consensus** - Making all nodes agree on something
### Consistency Models
- Different data no two nodes because write requests arrive on different nodes at different times.
- Consistency models define what guarantees are offered to clients when reading data.
#### Eventual Consistency
- **no ordering guarantees**
- High availability and performance, Low consistency.
#### Causal Consistency
- **causal order**(Dependent req after dependee)
- Implemented using lamport timestamps, logical clocks.
- Eg - comments need to have causal consistency. 
#### Sequencial Consistency
- **Program order** -> requests ordered in the exact order that the client issued them.
- **No recency guarantee** -> no guarantee that writes are visible immediately after they are processed. 
- Eg - Single Leader, Quorum, Total Order Broadcast, Consensus.
#### Linearizability(Atomic Consistency)
- **Atomic Consistency** - DB creates a illusion that there is only one replica. Every action on DB appears atomic.
- **Recency Guarantee** - As soon as a write completes, all the reads have the access to the latest data.
- Implemented using Consensus Algorithms
##### Uses
- **Leader Election** - Leader is elected by leasing a lock. This operation must be linearizable -> all nodes must agree on the leader. 
- **Constraints and Uniqueness** - Username/email must be unique -> all nodes must agree that a username is available -> Linearizable.
- **Cross Channel Dependencies** - Write to DB, push to queue -> No ordering guarantees leads to race conditions.
*Single Leader - possible, Consensus - Linearizable, Multi-Leader - Impossible, Leaderless - difficult*
##### Cons
- Requires coordination between nodes -> costs performance
- High latency (network round trips to coordinate data).
- Under partition -> must sacrifice availability(To coordinate writes reads will have to be blocked).
### CAP Theorem
- Consistency, Availability and Partitions tolerance cannot be guaranteed together at all times.
- **Limitations** - Here consistency refers to linearizability and availability is only one of the system faults that can occur.
### Ordering Guarantees
#### Total Ordering
- Guarantees all nodes see the same order of operations. => Every node sees the same order.
- If a system is linearizable -> there is a total order of requests.
- Difficult in distributed systems.
#### Causal Ordering
- Only dependent operations must be ordered. Unrelated ones can be unordered.
- Most systems usually only require causal ordering guarantees.
- **Single Leader** - Use sequence numbers/ Timestamps/ Logical Clock to order the writes.
- **Multileader/leaderless** - Generating causally ordered sequence numbers is difficult -> writes can go to any node.
	- Use time of day clock for timestamp -> Clock Skew. 
	- Numbers generated on arrival are wrong as requests face network delays.
##### Lamport Timestamp(Logical Clock)
- Each node maintains a counter. Each msg sent/received is tagged with this counter.
- **Rules for counter update** - 
	- **Internal Event** - increment counter.
	- **Send message** - increment counter and add to msg.
	- **Receive message** - counter = max(msg.counter, node.counter) + 1.
- This ensures if A->B , L(A)<L(B). But if L(A)<L(B) doesn't mean A->B.
### Consensus
- Nodes agreeing on a single value.
- A consensus algorithm must satisfy the following properties - 
	- **Uniform agreement** - No two nodes decide differently
	- **Integrity** - No node decides twice
	- **Validity** - Decided value must be proposed
	- **Termination** - All non-failed nodes eventually decide. -> Algo will not wait for a crashed node.
	  *Assumed that if a node crashes it does not come back online. If a algo waits for nodes to come back, it violates termination.*
- **Assumption** - 
	- More than half of the nodes are alive at any point of time.
	- Byzantine faults can also be handled if less than 1/3rd of the nodes lie.
#### 2 Phase Commit
- **Atomic Commit** - Ensure atomicity for a distributed txn commit.
- Uses a *coordinator/txn manager* which is a separate service.
- Txn wants commit -> coordinator sends a *prepare request* to all nodes.
	- All nodes reply yes -> *commit request* sent to all nodes -> Node must make sure it commits(Yes reply after txn logged to WAL)
	- Any node replies no -> *abort request* to all nodes.
- If any prepare request fails/timeouts, the coordinator aborts the txn.
  If any commit/abort request fails/timeouts, the coordinator must *keep retrying* until a response is received.
- **Violates the Termination property** of consensus algorithms.
##### Issues
- Coordinator is a single point of failure.
- **Poor performance**- each txn has atleast 2 network calls.
- **Blocking Protocol**- Node failure leads to retries of txn -> holds locks it leased which blocks other txns.
#### Total Order Broadcast(Sequential Consistency)
- Causal Ordering is not sufficient when you want to perform uniqueness checks. You need to know the total order.
- Protocol for sending messages between nodes which has 2 guarantees -
	- *Reliable Delivery* - No messages will be lost
	- *Total Ordered Delivery* - Messages will arrive in the order they were sent.
- Essentially a consensus algorithm(Where to route which msg is consensus-based). Can be thought of as a log with messages as entries.
- Implemented by zookeeper/etcd.
- **Asynchronous** in nature.
##### Implementation
- Linearizable register and compare and set operation. Every message can be assigned a id using this combination. This ensures order.
##### Uses
- **Replication** - To send writes to replicas
- **Fencing Tokens** - Each request to acquire a lock is appended as a message in the service's queue.
- **Linearizable Storage** - Total order of things -> Uniqueness checks using compare and set -> (Username will have 0 if unoccupied. Set to 1 if occupied. All subsequent requests will fail).
#### ❌ Epoch Numbers
- All Consensus algorithms internally use a leader in some form.
- They define a epoch number and node having the same epoch number will have a unique leader always.
- For any election/any decision by leader, the node trying to be the leader will send a request to all nodes and wait for response.
- If the response reveal a node with a higher epoch number, then the node is not declared the leader/proposal is rejected.
#### Limitations of Consensus
- Voting for leader election/proposal, all are synchronous -> costs performance due to network delays and halting of DB.
- Atleast more than half nodes should be functioning for consensus to work.
- Consensus uses timeouts to detect failed nodes -> Some nodes may be declared dead falsely.
#### ⭐️ Membership and Coordination Services
- Zookeeper, Kafka, etcd provide coordination services.
- They implement Total Order Braodcast.
- Replicated key-value stores for metadata.
- Features - 
  - **Linearizable atomic operations** - Implements locks with a linearizable register. Locks acquired through compare and set and only one node can succeed to get any lock.
  - **Total ordering of operations** - Linearizable register with *cas* operation -> incrementing ids can be generated -> total order.
  - **Failure detection** - All nodes maintain session on zookeeper. There is constant polling to check for failure detection. 
  - **Change notifications** - If any service depends on another, Zookeeper can send notifs to indicate if a node failed.
- **Service Discovery** - All nodes/services must register itself on Zookeeper. As services fail and can come up on on new servers, any node trying to use a service should request zookeeper to give where(IP:Post) to direct this query.
## Batch Processing
### Unix Philosophy
- Programs should do one thing well and you compose multiple programs together.
- Data should be **immutable**.
### MapReduce Model(Hadoop)
- Used in large scale batch processing systems like Hadoop.
- Just write code for mapper and reducer. MapReduce **automatically parallelizes** it.
	- **Mapper** called per record -> Extract key-value from each record. 
	- **Reducer** called per key-value pair -> Produce output based on key/value
- **MapReduce Workflows** - chain several jobs together -> Ouput directory of one job becomes input of another.
#### Distributed Execution of MapReduce
- There is a **Master node** which schedules tasks and handles worker failures.
- **Worker nodes** - Execute map and reduce tasks and store data locally.
1. **Input Split** - Input is split into partitions and stored on nodes.
2. **Map Phase** - Map task run on each node, input record into key-value pairs, sorted by the key.
3. **Shuffle and Sort** - Mapper output is exposed over the network. Reducer nodes fetch their partition of keys from each node.
4. **Reduce Phase** - Key value pairs are reduced into the required information based on the user code.
- **Skew/Hotspot** 
	- One key has a lot of records in the second record set(*Hot Key*) -> Overloading of one Reducer. 
	- Use Key-Salting -> Spread the same user over multiple keys and distribute orders accordingly.
#### Fault Tolerance
- If Mapper fails, map task is rerun on a different worker(as map is deterministic). 
- If Reducer fails, reducer is rerun -> Fetch data from nodes -> generate output.
### Joins
#### Reduce-Side Join
- Both datasets are shuffled across reducers.(Each node has partitions of each dataset, Reducers get values from both partitions).
- **Heavy network shuffle** but *Works on all DBs*.
#### Sort-Merge Join(Reduce-Side Join)
- One of the datasets is small enough to store in memory(Stored in reducer job's memory). Large data set is processed using MapReduce.
- **One table must fit in memory** but *No extra shuffle*
#### Broadcast Join(Map-side Join)
- Small dataset is kept in memory. Large dataset is set for batch process and all references are mapped in-place.
- Leads to no reduce phase as all data is processed in the map phase itself.
- Faster, No shuffle cost.
- **Cons** - Table must fit in memory.
#### Partitioned Join(Map-side Join)
- Both datasets **partitioned using same key + hash function** -> Related records are present in the same partition.
- **Efficient** - No shuffle is required.
- **Cons** - Preprocessing required. Harder to operate.
### MapReduce Problems
1. **Intermediate HDFS writes**- Each job writes output to disk.
2. **High latency**- Next job waits for previous to finish.
3. **Unnecessary sorting**
### Output of Batch Processing 
- **Atomic Output Replacement**- Write new version of data in a new file, Rename it atomically -> Quick switch to new data, versioning.
- **Exactly-once semantics**- MapReduce has deterministic jobs with output replacement(no in-place updates)-> No record is processed>once
### Distributed DB vs Hadoop
- DB stores data in structures. MapReduce has no such requirements input is a sequence of bytes.
- DB is optimized for low latency queries. Hadoop is optimized for high throughput processing of entire data.
- DB usually services OLTP loads. Hadoop services ETL jobs, Log Analysis etc.
- Hadoop stores files in HDFS, with variable schema and immutable datasets.
### Dataflow Engines
- *Operators* defined instead of Map/Reduce which can perform any configured task.
- Intermediate results are stored in memory, reduces time to write data out to disk.
- Next operator can start executing as soon as the input is produced to memory.
- Sorting is optional.
#### Fault Tolerance
- No intermediate state -> durability is reduced.
- To reconstruct the state, the **operator must be deterministic**.
#### Graph and Iterative Processing
- Graph algorithms run in multiple rounds. Update the node state repeatedly until convergence -> *Iterative*
- MapReduce is expensive -> In one iteration only a small number of nodes will update but you read/write all data.
- **Spark is better** -> Stores intermediate state in memory so no writes to disk.
## Stream Processing
- **Event** - Every record is also known as an event. Contains a timestamp, UID(Idempotency) and partition-key(for partitioning).
- **Schema Evolution** - Event schema evolves over time. Events need to be backward and forward compatible.
### Processing Guarantees
- **At-most-once** - No retries. Dataloss is possible.
- **At-least once** - Retries are allowed -> Duplicate processing possible.
  If msg processed but just before ack, consumer crashes OR ack gets lost in network 
- **Exactly once** - No loss, No duplicates.
	- Requires
		- *Transactional writes* - Multiple writes across partitions must either all succeed or all fail.(Distributed txn).
		- *Idempotent producers* - Retries dont produce duplicates messages.
		- *Atomic offset and output commits* - Consumer produces output and commits offset to broker. 
			  If it crashes before committing offset, message will be retried. -> Output and offset commit should be atomic.
##### Idempotent Operations
- Operation that produces the **same result even if repeated**.
- Make any event idempotent -> Use (procuderId, sequenceId) -> If broker processed it before, reject the event.
### Message Brokers 
- Distributed append-only log optimized for streamsing.
- **Offset** - Each event is given a sequence number which is increasing. Order is guaranteed within the partition.
- **Durability** - Depends on the broker -> some keep messages in memory while others write them to logs.
#### Partitions
- Logs can be partitioned, each machine has 1 partition. A set of partitions can be dedicated to a **Topic**.
- Every partition is assigned to one consumer in the consumer group. Although one consumer can consume multiple partitions.
- Ordering guaranteed **within partition** through offsets. Not guaranteed across partitions.
### Consumer Groups
- Multiple consumers processing same topic.
- **Consumer** - 
	- Maintains the offset of messages it has consumed.
	- *Backpressure* - If consumer is slow, broker must slow down the event production.
	- Kafka has a pull system -> Consumer controls read rate not the broker.
- **Types of parallelization** - 
	- **Load balancing** - Message is sent to the one of the consumers in the consumer group to parallelize processing.
	- **Fan-out** - Message delivered to each consumer group so that they can carry out their own processing of messages.
#### Rebalancing
- Happens when a consumer drops/joins or the partition count changes.
- Causes temporary pause as redistribution of partitions across consumers.
- May lead to duplicate processing of messages.
#### Hot partition problem
- One partition is overloaded due to bad distribution of keys.
- **Key-salting** -> If a key is too hot, split it into multiple keys(append 00 till 99 to the key -> 100 keys) -> better distribution across partitions.
- **Dynamic Sharding** -> Hot keys initially have 1 key but as the messages increase, the number of keys assigned also increase.
#### Circular Bounded Buffer
- Logs are only deleted after the disk is full. If consumer falls behind-> the offset is pointing to a deleted log -> lost messages.
- Disk is very large -> Long time before messages get dropped -> Manually reset the consumer.
### Change Data Capture
- Used to keep data in different systems in sync.
- Read WAL -> push messages to broker.
- There needs to be an initial point to build a system - use **Snapshots or Compacted Logs**.
#### External Side Effect Problem
- If consumer has a side effect of message processing(Eg- write to DB, notifications), retries will lead to duplicate issues.
- Use Idempotent Producers.
- **Transactional Outbox Pattern** - Event processing and side effect are handled together as a txn.
### Event Sourcing
- Architectural Pattern where instead of storing state you store the events. All actions in system are events.
- State is in memory, constructed through logs.
- **Properties** - 
	- **Immutable Events** - Events cannot be modified.
	- **Append-only Log** - Events are written sequencially to the log.
	- **Derived State** - State can be derived by going through the sequence of events.
#### Advantages
- **Full Audit History**
- **Easy Debugging**
- **New Views and Easy Migration** - New View -> stream the events. Migration -> stream events and add changes.
#### Disadvantages
- **Complex**
- **Migration needs to be backwards compatible**
- **Replay is costly** - You need to store snapshots which is costly.
### Event Time vs Processsing Time
- **Event Time** - Time at which the event occurred. **Processing Time** - Time at which the event is processed.
- Processing time is different due to network delays, wait in queue etc.
### Stream Joins
#### Stream-stream join
- Joining of two streams within a time window.
- Eg- Stream of user searches and Stream of purchases. Say you need to track number of purchases within 5min of a search.
- Processor maintains a state in memory. In our eg, we track the last search and count the number of purchases.
#### Stream-table joins
- Joining of a stream with a changing table(through CDC).
- Eg- Stream of orders, you need to stamp the address based on the userId passed.
### Fault Tolerance
How do we deal with consumer crashes
#### Microbatching
- Break streams into batches and retry the batch if anything fails.
- **Latency vs Overhead**- Small batch size -> scheduling overhead, Large batch size -> Delay in generating the results.
#### Checkpointing
- Checkpoints are generated by writing the state to the disk. If crash -> restart from the last checkpoint.
#### Dead Letter Queue
- If message fails repeatedly, you put it into the DLQ.
- This prevents the pipeline blocking and infinite retry loops.
#### Message-Passing vs RPC

| Aspect           | Message Passing | RPC     |
| ---------------- | --------------- | ------- |
| Coupling         | Loose           | Tight   |
| Timing           | Async           | Sync    |
| Failure handling | Explicit        | Hidden  |
| Scalability      | High            | Limited |
### Distributed Actor Frameworks
- **Actor** -
	- Has private state
	- Communicates only via messages
	- Processes messages sequentially
###### Why Actors Scale Well
- No shared memory
- No locks
- Failure isolation
This ensures no locks and no race conditions.
###### Actor Systems vs Message Brokers

| Actor Framework           | Message Broker        |
| ------------------------- | --------------------- |
| Tight runtime integration | Infra-level component |
| In-memory messaging       | Persistent messages   |
| Fast, low latency         | Durable, slower       |
| Handles logic             | Handles delivery      |

## Back of the Envelope Calculations
Before design, ALWAYS calculate:
1. **API QPS**
2. **Reads/sec (after amplification)**
3. **Writes/sec (after fanout)**
4. **Data per day/year**
5. **Top bottleneck**
#### Rules
- Concurrent users = 1% of DAU.
- Peak RPS = 10x average RPS
- Read heavy systems - reads = (10 - 100) x writes
- Rate = X req per day = X/84000 per sec = X * 1.2 * 10 ^ -5
###### Example - 
- 500M DAU
- Each user watches **5 videos/day**
- Average video size = **50 MB**
- 10% users upload videos
- Average upload size = **50 MB**

- RPS = 2.5B per day ~ 30K per sec.
  Peak RPS = 300K per sec.
- WPS = 50M per day ~ 600rps
  Peak WPS = 6K rps
- Bandwidth - 
  Each video = 50Mb upload, download
  Total downloaded data = 2.5B videos * 50Mb per day = 125B Mb perday = 125M GB per day = 125 PB per day.
  Download Bandwidth = 1.5 TB per sec
## Consistency and Distributed Txn Patterns
#### CQRS (Command Query Responsibility Segregation)
- Separate DBs for Read and Write requests.
- *Write DB* - Normalized, consistent. 
  *Read DB* - Denormalized, eventually consistent.
```
Client → Write API → Write DB → Event → Update Read DB

Client → Read API → Read DB
```
- **Tradeoffs** - Eventual Consistency, Complexity
#### Event Sourcing
- Store all events happening with the system.
- Use write optimized DB based on append only logs.
- Replicas maintain a view by aggregating the state.
- *Snapshots* - help rebuild state in case of failure.
```
Command -> Validate -> Generate Event -> Store Event -> Update State
```
- **Tradeoffs** - Complexity, Bad Read Performance, Schema evolution.
#### Saga Pattern
- Used when -> Perform action involving multiple services each having their own DB.
- If we use a global txn(2PC), we have to block other txns.
```
Order → Payment → Inventory
```
- Performs a **local transaction**, Each txn has a **compensating action** (undo) -> Instead of rollback.
- **Choreography based** -
	- Each service after completing generates an event, which is consumed by other service.
	- *Pros* - Simple, Loosely Coupled.
	  *Cons* - Hard to debug, Logic is scattered and difficult to track flow.
- **Orchestration based** - 
	- Central service is responsible for controlling other serivces.
	- *Pros* - Simple Logic, Easy to Observe.
	  *Cons* - Tight Coupling, SPOF.
- Eventual Consistency, Requires Idempotency, Used for Handling Partial Failures.
#### 2 Phase Commit
- **Atomic Commit** - Ensure atomicity for a distributed txn commit.
- Uses a *coordinator/txn manager* which is a separate service.
- Txn wants commit -> coordinator sends a *prepare request* to all nodes.
	- All nodes reply yes -> *commit request* sent to all nodes -> Node must make sure it commits(Yes reply after txn logged to WAL)
	- Any node replies no -> *abort request* to all nodes.
- If any prepare request fails/timeouts, the coordinator aborts the txn.
  If any commit/abort request fails/timeouts, the coordinator must *keep retrying* until a response is received.
- **Violates the Termination property** of consensus algorithms.
##### Issues
- Coordinator is a single point of failure.
- **Poor performance**- each txn has atleast 2 network calls.
- **Blocking Protocol**- Node failure leads to retries of txn -> holds locks it leased which blocks other txns.
#### Transactional Outbox Pattern
- Save data in DB and publish event to kafka at the same time.
- Store data in a **Outbox table**- Store eventId, event details, origin_id etc.
- A service periodically polls the outbox table to fetch the events to be fired.
- **Pros** - No event if DB write fails, Protection against Message Queue failure.
#### Inbox Pattern
- Service **stores every incoming message/event in a DB table before processing it**
- Some messages/events may be delivered twice(atleast once guarantee by queues) -> Process only once.
- **Pros** - - Prevents message loss, Helps with retries, Enables deduplication
#### Idempotency Pattern
- processing the same request multiple times gives the same result
- duplicates can happen - Network retries, Client retries, Message re-delivery
- Use **idempotency key** and store processed keys. -> Duplicates not processed.
#### Change Data Capture
- Capture DB changes directly
- DB → binlog → CDC tool → Kafka Eg- Debezium
- No need for outbox table -> Works with legacy systems
## Messaging and Event Driven Patterns
#### Pub-Sub
- Producer produces messages to a topic. All subscribers of the topic must receive messages.
- *Steps* - 
	- **Publisher publishes Message** - Message is sent to Broker.
	- **Broker identifies topic and adds metadata**
	- **Broker pushes the message** - into a queue(RabbitMQ)/log(Kafka).
	- **Subscriber registers** - Broker manages registrations and subscriber is added as a consumer of the queue.
	- **Consumer(Subscriber) processes messages** - 
		- *Push Model* - Broker sends the messages to consumer.(RabbitMQ)
		- *Pull Model* - Consumer is responsible to poll the broker to get the message.(Backpressure in Kafka)
	- **Subscriber acks message**
	- **Broker commits offset** - Store the offset of processed message -> Crash recovery.
#### Event Notification vs Event carried State
- Event notification -> Store data in DB, pass id -> *Extra DB fetch on read*.
- Event carried State -> Send entire data through message -> *Large payload but faster consumers*.
- **Tradeoff** - Payload size vs Latency
#### Competing Consumers
- RabbitMQ -> Multiple consumers can consume the same queue by attaching
- Kafka -> Each queue has partitions - Each partition can be consumed by one consumer in a consumer group.
  Multiple consumer groups can attach to the same queue.
#### Dead Letter Queues
- If a message keeps failing, you move it to the DLQ -> Prevent infinite retires.
- Debug failures later
#### At-Least-Once / At-Most-Once / Exactly-Once
- **At-Least-Once** - Message delivered ≥1 times -> Duplicates -> Requires Idempotecy
- **At-Most-Once** - Delivered once or never. Risk of data loss
- **Exactly-Once** - Ideal but hard to achieve - Outbox + Inbox + Idempotency
#### Ordering
- Order matters in some systems. Eg- Chat System.
- Solutions - **Keep one consumer per queue** or **Partition by DB key**
#### Backpressure
- Producer is faster than consumer
- **At Queue level** - Consumer polls the queue for messages.
- **Techniques at API level**- Queue buffering, Rate limiting and Reject requests (HTTP 429)
#### Request Coalescing
- Multiple reads with same query -> Do one DB call and return result.
- Map<key, Promise> - stores hash(query params) as key, Promise to the ongoing request.
- First request creates the Promise. Subsequent requests look for the promise in map, if found await on it.
#### Hedged Requests
- This is done in response to tail latency.
- If a request is taking a lot of time, you send a backup request hoping to bypass the tail latency.
## Read Write Optimization Patterns
#### Materialized View Pattern
- Precompute and Cache the results of expensive queries.
- Example - News feed per user. Dashboard metrics/Analytics systems.
- *Implementation* - Updates to DB -> events to Kafka -> Consumer updates the cache.
- **Tradeoffs** - Fast Reads vs Eventual Consistency + Extra storage.
#### Denormalization
- Duplicate data to optimize reads -> Fewer joins.
- **Tradeoff** - Data Duplication vs Read Latency
#### Fan out on Read vs Write
##### Fan-out on Write
- When data is written → push it to all Materialized Views
- **Tradeoffs** - Super fast reads vs Expensive Writes. Bad for users with large number of followers.
##### Fan-out on Read
- Compute feed when user opens app
- Fetch posts from all followed users → merge → sort
- **Tradeoffs**- Cheap writes vs Slow Reads, Complex queries.
#### Read Repair
- In a quorum system, replicas become inconsistent.
- When the coordinator receives conflicting reads, it updates the inconsistent replica.
#### Write Repair
- Write fails on some replicas -> Fix later asynchronously
- Ensures Eventual consistency
#### Quorum-Based Replication
#### Cache Aside vs Write Through vs Write Back
- **Write Through** - Cache and DB are written to in the same txn.
	- + High Consistency
	- - High Latency.
- **Write Back** - Data written to cache and then DB is updated asynchronously.
	- + Low Latency
	- - Eventual Consistency, Cache failure leads to data loss.
- **Write Around** - Data is written to DB and cache is updated on a cache miss.
	- + Applications that dont read immediately written data -> Avoid unused item in cache
	- - Cache miss is expensive
#### Hot Key Handling
- Some keys get massive traffic
- **Solution**- Replicate hot key, Cache aggressively and Shard key artificially
#### Pagination & Windowing Pattern
- Large datasets and you need fetch a set of results.
- **Offset-based pagination**
	- Use offset based queries
	- **Pros** - Simple to implement, easy to jump to any page.
	- **Cons** - Slow for large offsets as full scan to the offset, Inconsistent results if data changes.
```SQL
	SELECT * FROM posts
	ORDER BY created_at DESC
	LIMIT 10 OFFSET 20;
```
- **Cursor-based pagination**
	- Use a **cursor (last seen value)** from a index.
	- **Pros** - Fast, No skipping and consistent results.
	- **Cons** - Cannot jump to arbitrary page easily, More complex API
``` SQL
	SELECT * FROM posts
	WHERE created_at < '2026-01-01'
	ORDER BY created_at DESC
	LIMIT 10;
```
#### Ranking/Merging Patterns
- When you get a request like top 20 posts. You have multiple(say 5) sources, and merge them into a final list.
- Eg - You have feed list and you fetch posts from the celebs. -> Multiple sources -> One list required.
##### K-way Merge
- Treat each source as a **sorted list** and merge based on relevance score.
	- **max heap on ranking score** -> Push top of precomputed feed, top post of each celebrity.
	- **Pop the best item**, add the next item from that store.
- `O(K log N)` → very efficient, No full reranking.
##### Score-Based Merge (Light Re-ranking)
- Precompute a **score** for each post based on recency, engagement.
- At read time- Fetch candidates (~100 max) and apply **lightweight adjustments**:
    - Boost recent celebrity posts
    - Personalization tweaks
- Keeps personalization flexible and avoids heavy ML inference at read time
##### Two-Level Feed
- Dont merge, interleave the lists.
- Show: 15 items from precomputed and 5 items from celebrity.
- Predictable latency and Avoids heavy ranking
##### Pre-Merged Buckets
- Maintain celebrity feed cache as well.
## Reliability, Failure Handling & Multi-Region Patterns
#### Retry Patterns
- In case of transient failures, you do a retry.
- *Retry Storm* - User calls -> System down -> 10K requests fail -> 10K Retry requests -> System overload and fails. This is a loop.
- **Immediate Retry** - Causes load to increase on the system as multiple users try to retry again.
- **Retry with a delay** - Retry periodically after x time. -> Leads to retry storms, does not consider load.
- **Exponential Backoff** - Retry delay increases exponentially
	- *Reduces load on a failing system*, *Gives time for recovery*.
- **Retry with Jitter** - Exponential delay + Some randomness.
	- Exp delay will still cause bulk of requests arriving at the same time -> Retry storm.
	- With jitter, requests are spread apart.
- *Ideal Retry* - Set timeout, Exponential Backoff and Idempotency.
#### Circuit Breaker
- A Circuit Breaker **stops your system from repeatedly calling a failing service**,
- It has **3 states**:
	- Closed - Requests go through normally, Failures are monitored
	- Open - Too many failures → breaker opens, Calls are **blocked immediately**
	- Half-Open - After some time → allow a few requests
```Java
Closed → (too many failures) → Open  
Open → (after timeout) → Half-Open  
Half-Open → success → Closed  
Half-Open → failure → Open
```
- *Installed on the service side.*
- **Pros** 
	- *Prevents cascading failures* - One service failure does not impact another.
	- *Fast Failure* - Client does not wait for timeouts.
	- *Helps Recovery* - Moderates incoming traffic for a service that is recovering.
- **Tradeoffs**
	- *False Positives* - Service may recover and breaker is still waiting to send a request.
	- *Added Complexity*
#### Graceful Degradation
- If services fail, return partial functionality instead of failing
#### Heartbeat Pattern
- Detect failed nodes by sending periodic signals.
#### Active-Passive (Failover)
- Keep one node as a standby to the leader node.
- When the leader fails, quickly do a failover and make the standby as primary.
- **Tradeoffs** - Simple and Safe vs Failover Delay
#### Active-Active (Multi-Region)
- Multiple regions serve traffic
- **Tradeoffs** - Low latency, High availability vs Data consistency,Conflict resolution
#### Geo-Partitioning
- Store data close to users. Indian users → India DB, US users → US DB
- Lower latency
## Patterns I noticed
### DB + Queue Txn
- **Outbox Pattern**
- **Change Data Capture**
### Idempotency
- Pass idempotency-key with request.
- Store idempotency key in DB/Cache. Query the Cache/DB when the message comes in if it has been processed.
- Have a uniqueness constraint on the idempotency-key.
- Have a TTL for the records in this table.
### Real Time Chat Application
#### Ordering
- Ordering is done per conversation. -> Partition based on conversation_id
##### Hot Key
- For a large group(1M+) - we need to use multiple consumers -> Order breaks. -> Relax ordering guarantee.
- **Shard conversation into partitions**. *Number of shards* = (# write requests per sec)/(server capacity).
- Partition the queue based on conversation_id and user_id. -> maintain user_id based ordering. 
- Guarantees:
    - per-sender ordering
    - per-shard ordering
    - best-effort global ordering
#### Read Path
- For online consumers, consumers send the message back by the connection management service.
- Client updates which messages are read and updates the DB.
- For offline users, consumers do nothing.
- When user comes back, fetch message from DB.
#### Connection Management Service
- Maintains WebSocket connections records -> **KV Store Connection Registry** 
  -> *user_id → [{server_id, connection_id}]* - Supports multiple devices per user
- Tracks online users through **heartbeats**.
	- **Connection Lifecycle** - 
		- Client → LB → connection server -> Register in KV store
	- **Heartbeat** - Periodic ping/pong to detect liveness
	- **Disconnect** - Remove mapping (or expire via TTL)
	- **Message Routing** - Consumer queries registry -> RPC to server -> pushes via WebSocket
- **Optimizations**
	- Local cache for active connections
	- Batch routing per server
	- Avoid per-user RPC calls
- **Failure Handling**
	- Server crash → stale entries → cleaned via TTL
	- Stale routing → retry after refresh
### L4 vs L7 Load Balancers
- **L4** - Transport Layer(TCP/UDP), Decision based on IP+port, 
	- No visibility into request.
	- does NOT terminate application protocol -> Forwards TCP packets to the downstream server.
- **L7** - Application Layer(HTTP/HTTPs), Decision based on URL, Header, cookies, 
	- *Can look into requests*. -> As it read the request, it becomes the endpoint for the client.
		-> L7 terminates TCP from Client -> LB creates another TCP to the Server.
### Rate Limiter

### Uber
#### DB vs Cache — Quick Framework
**1. Durability**
* Must not lose → **DB else Cache**
**2. Latency**
* < 50 ms → **Cache**, else **DB OK**
**3. Read QPS**
* < 1K → **DB**
* 1K–10K → **DB + cache**
* > 10K → **Cache**
**4. Write QPS**
* < 1K → **DB**
* 1K–10K → **DB (scaled)**
* > 10K → **Cache / streaming**
**5. Access Pattern**
* Key lookup → **DB/Cache**
* Geo / Top-N / Ranking → **Cache**
* High-frequency updates → **Cache**
* Historical / analytics → **DB**
**6. Consistency**
* Strong → **DB**
* Eventual → **Cache**
**7. Rule of thumb**
* **Cache = speed**
* **DB = source of truth**
### Swiggy
- Single Source of truth -> must be DB.
- **Multi System Transaction** - Eg- DB + Cache write txn -> Write to DB, propagate through Queue.(Outbox)
	- This ensures atomicity of updates.
- Implement Idempotency everywhere -> Queue, Payments, Updates etc.
- **CDC vs Async Queue Updates** -
	- CDC generates binlog file and reads it later.
	- Slower than Queue.
