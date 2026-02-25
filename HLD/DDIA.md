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
- **Schema on write** - Rigid Schema. *Migration* required to change schema.
- Supports joins, transactions and constraints. 
- **Object and Relational Mismatch** between the application objects structure and table data. 
- **ORM(Object Relational Mapping)** used for the translation as it reduces boilerplate code.
#### Used when
- Data has clear structure.
- Relationships matter and are frequently queried.
- Strong consistency and transactions are required.
### Document Store
- Stored as objects(JSON/BSON) or XML.
- **Flexible or schema-on-read** design. -> Supports Heterogenous Data
- **Better locality of data** - Similar data stored in the same object so **fewer joins** are required.
- **Limited support for joins** - Application is responsible for maintaining consistency.
- **Writes are harder** - In-place updates can only happen if encoded document's size does not change. Also, consistency issues.
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
#### Triple Stores
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
## Query Languages
### Declarative Languages
- Instead of telling the machine how to do something, tell it what to do. Machine will internally handle the optimizations and logic required.
- Example - SQL, CSS
- **Abstraction** - hides away internal details of how a query functions. Helps in performance improvements without any change in query.
- **Parallel Execution** - Imperative languages are hard to parallelize due to the order of instructions. Declarative languages have a better chance parallelize.
### Map Reduce Queries
- It is neither declarative, nor imperative. Somewhere in the middle.
- Based on two functions, **map** - filters relevant data, **reduce** - makes sense out of this data.
- Example - MongoDB
```js
db.observations.mapReduce(
	function map() {
		var year = this.observationTimestamp.getFullYear();
		var month = this.observationTimestamp.getMonth() + 1;
		emit(year + "-" + month, this.numAnimals);
	},
	function reduce(key, values) {
		return Array.sum(values);
	},
	{
		 query: { family: "Sharks" },
		 out: "monthlySharkReport"
	}
);
```
- **Limits** - map and reduce cannot perform internal queries. They can only operate on data passed into them.
		- map() and reduce() must be carefully coordinated. SQL is usually easier to write that this.
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
- <Key, Offset> -> Offset where data for a key can be found.
- **Hash Map must be stored in memory to ensure fast searches.**
- Log is broken into segments of fixed size -> Store recent values of keys and throw duplicate values(**Compaction** -> Essentially a map).
- Usually there are few keys so we can merge multiple segments together. 
- Read requests are initially directed to the unmerged segment and then to the compacted segment file.
- **Search** - look into the memory hash map.
- **Delete** - append an entry stating the value is deleted. When next compaction happens, the key will be discarded.
- **Crash Recovery** - Rebuild the hash map by scanning the current segment.(*Expensive*)
- **Partially Written Records** - Add checksums to ensure the value written is correct or not. Checksums will be created before any write happens.
- [ ] **Hash Table size** - needs to fit in memory. Issue if more number of keys
- [ ] **Range Queries** - not efficient.
#### LSM(Log-Structured Merge) Tree
- Has 3 components - WAL, MemTable and SSTables
- *Sorted String Tables* - On disk DS where the index is sorted. -> **Fast writes tradeoff with read amplification**
- **Writes** - Go to an in memory table implemented using self balancing trees(AVL/Red Black Tree).
- **Memory Full** - Data flushed onto a SS Table.
- [ ] **DB Crashes** - In memory writes -> **Write-ahead Log(WAL)** -> Key,Value pair is first appended to log to prevent data loss in case of DB Crash.
- [ ] **Easy Merges/Compaction** - Simple merge sort algorithm to be used.
- ==Lucene==-Used by Elasitc Search uses similar architecture. Key-word, Value-Id of Doc. 
##### Sparse Indexing
- If number of keys is large, store the offset of a few keys per unit of data(Say 1Mb)
- To find the value go the nearest key and search from the offset there till the offset of the next key.
##### Reads and Bloom Filters
- A Bloom filter can tell you whether a key does _not_ exist in an SSTable. It gives *some false positives*.
- To read data you search in the memTable. If not found you check bloom filters of each table.**- Read Amplification** 
- If the bloom filters say maybe then search the table.
##### Compaction Techniques
- **Size tiered**- Newer and smaller SSTables are merged into older and larger ones.
- **Leveled** - SSTables are split based on range to make them smaller and managable.
#### B-Trees
- Stores key value pairs sorted by keys.
- Break the DB down into page size units and each node has a page assigned to it. 
- Each page is responsible to store values for a range of keys
- **Leaf node** - contains exact value for each key. **Other nodes** - Starting Key for child page, reference to page. \
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
- LSM Trees are for faster writes. LSM Trees are slow as they read multiple DS.
- B Trees are for faster reads. Writes have to be written twice, once to the WAL and then to the tree page.
- **Write Amplification** - One write in DB leads to multiple writes on the disk(WAL in BIndex and Compaction in LSM). Direct performance overhead in write heavy applications.
- LSM Trees are faster in writes as they have lower Write Amplification. It happens only when compaction happens.
- LSM Trees also use memory better as compaction often produces smaller files on disk as compared to B-Trees(due to fragmentation in BTree pages).
- Some requests might have to wait while the expensive compaction operation is completed. Specially evident om tail latencies(higher percentiles).
- Disk finite write bandwidth needs to be shared between the incoming writes and compaction threads.
- If write throughput is high, compaction will not keep up and you will have to lookup old uncompacted segments to get work done.
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

### Why evolvability matters
- During upgrades, **old and new versions** of clients and servers coexist.
- Systems must support reading data across versions.
### Compatibility
- **Backward compatibility**: New readers can read data written by old writers.
- **Forward compatibility**: Old readers can read data written by new writers.
### Encoding / Decoding
- Encoding converts in-memory data into bytes for storage or transmission.
- Decoding reconstructs data from bytes.
- The encoding format defines how evolvability is handled.
### Language-specific encodings
- Require the **same library** on both sides.
- Often instantiate arbitrary classes → **security risk**.
### Problems with XML, JSON, CSV
- XML/CSV cannot reliably distinguish numbers from strings.
- JSON:
	- No distinction between integers and floats
	- No precision guarantee
- JSON/XML lack native binary support.
- Schemas are optional and **not enforced at read/write time**.
### Binary Encoding with Schemas
#### Thrift & Protocol Buffers
- Schema defined upfront.
- Fields identified by **tag numbers**.
```JS
message Person {   
	required string user_name = 1;   
	optional int64 favorite_number = 2;   
	repeated string interests = 3; 
}
```
- Encoding uses tags, wire types, lengths, and values.
- `repeated` replaces arrays.
- Required fields are dangerous (deprecated in modern Protobuf).
#### Schema Evolution Rules
- New fields - **Optional/Have Defaults** + **Unused Tag Numbers**
- Required fields - cannot be removed
- Tags - never reused
- Datatype changes - Allowed only if wire-compatible (e.g., int32 → int64)
**Limitation**: Evolution relies on **manual discipline** and static contracts.
### Avro
#### Schema languages
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
- No field tags → position-based on writer's schema
- Decoder needs - **Writer schema and Reader schema**
- Schema resolution happens at decode time. Decoding takes place based on writer schema and matched with reader schema by adding default values.
#### Schema Evolution
- Field order does not matter. 
- **New fields** must have default values (for backward compatibility - *new reader, old writer*).
- **Removing fields** allowed if default exists (for forward compatibility - *old reader, new writer*).
- Datatype changes allowed if compatible.
- Field renaming supported using **aliases** (backward compatible only).
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
- Clear documentation and contracts.
- Enables backward and forward compatibility.
- Code generation helpful for statically typed languages.
### Dataflow through DB
- Data written by old writers need to be read by new readers which are of newer versions.
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
- XML-based, strongly typed. Schema defined by WSDL
- **Pros** - Strong contracts, Built-in security & tooling
- **Cons** - Verbose, Different vendors may have different implementations -> Difficult to evolve.
> SOAP is **rigid but predictable**
#### RPC (Remote Procedure Call)
- Call a remote function as if it were local
- Uses encoders like Thrift / Protobuf / gRPC
- Enforces strong schemas
- **Pros** - Compact encoding, Clear contracts, Type safety
- **Cons** - Network failures feel like local failures so explicit handling is needed, Tight coupling, Harder versioning
	- Latency feels as if function call is taking a lot of time. Partial Failures, Retries and Timeouts are all treated as failures without distinction. This causes production bugs.
#### When to Use What
- Public APIs - REST.
- Internal Fast Performance APIs - REST
- Legacy Enterprise - SOAP
### Message-Passing Dataflow 
- Messages are delivered asynchronously and stored in a queue until they can be processed.
- Combination of a DB(msg stored in queue) and async RPC(procedure called to process the msg).
- *Message Brokers* facilitate this. Eg- Kafka, RabbitMQ, AWS SQS etc.
#### Why Message Brokers Exist
- **Durability** – messages not lost on system crash
- **Backpressure** – consumers control pace
- **Decoupling** – sender doesn’t know receiver
- **Retry & buffering**
#### Dataflow Pattern
Producer → Broker → Consumer(s)
Producers and consumers - 
- Do not need to know each other
- Do not need to be online simultaneously
#### Encoding in Message Brokers
- Messages are immutable
- No Schema is imposed so works well with JSON and Avro which do not have a concrete schema requirement.
- Schema evolution is critical because old consumers may read new messages
#### Message-Passing vs RPC

| Aspect           | Message Passing | RPC     |
| ---------------- | --------------- | ------- |
| Coupling         | Loose           | Tight   |
| Timing           | Async           | Sync    |
| Failure handling | Explicit        | Hidden  |
| Scalability      | High            | Limited |

### Distributed Actor Frameworks
- **Actor** - a single threaded entity that owns its state and communicates only via messages.
- Each actor:
	- Has private state
	- Communicates only via messages
	- Processes messages sequentially
- Actors can be local(within a process) or distributed(on a different machine).
- Actors are arranged in a hierarchy. 
  Parent is responsible for restart, stop, resume and error escalation of the child.
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

## Replication
- Keeping copy of the same data on multiple machines that are connected over the internet.
- Advantages - 
	- Reduce latency by having clones geographically close to the users.
	- **Failure tolerance** - system continues working even when a system fails.
	- **High Read Throughput** - Many replicas -> faster reads.
### Leader-Follower Replication
- One replica is called the *leader* where all **write requests** are routed. All others are *followers* which **server reads**.
#### Synchronous Replication
- Leader waits for the replication to happen on some replicas, then informs the user.
- Usually there is only one sync node. So that if the leader fails, this sync node can be made leader.
- **Issue**- If the sync follower responds slowly, all the writes will be blocked.
#### Asynchronous Replication
- Leader writes and informs the user. Does not wait for replication.
- **Cons**- Leader can continue processing writes even when all replicas have fallen behind.
- **Issue**- *Durability*- If the leader fails, any writes not replicated would be lost.
#### Setting up new followers
- **Data in DB is in flux** - just copying data sequentially would lead to corruption in data.
- **Locking the main DB**- Leads to downtime which compromises Availability.
- **Approach**-
	- Take consistent snapshots of the leader.(Usually done for data warehousing)
	- Copy data from snapshot to the new follower.
	- Copy log changes from leader until the replica has caught up.
#### Handling Node Outages without downtime
##### Follower failure
- Follower keeps a log of data changes it received from the leader.
- If follower crashes, it can recover by following logs from the last transaction.
- Then it can connect to the leader to catch up to the leader.
##### Leader failure- **Failover**
1. Determine the Leader has failed.
2. Choosing the new leader. => Leader Election(Choose the least lagging node/Decision on a predefined control node)
3. Reconfiguring the system to use the new leader.
###### **Issues**
- If async replication used, the new leader will **have to discard writes** that it lagged behind on.
- If the new leader lagged behind another replica, it would lead to **inconsistency**.
- If two leaders happen, it will lead to **split brain**-> data confilicts.
- What should be the **time to declared a leader failed**?
### Replication Logs Implementation
##### Statement based Replication
- Leader logs all SQL statements, then all the followers are to execute the statements.
- **Issues**-
	- Any non deterministic function(*NOW() or RAND()*) will generate different values.
	- If SQL has a WHERE clause, they must be executed in the same order. This is a problem if multiple txn execute.
	- Procedures/Triggers/User-defined Functions are also not deterministic.
##### Write Ahead Logs
- All writes before writing to DB are written on the WAL logs.
- This contains information about the exact bits that have to be changed in the DB.
- **Issues**-
	- WAL logs are dependent on the version of DB you run. New DB version may right in a different place.
	- Thus when you update the version, you require downtime.
	- If this was not the case, you could easily update the followers and do a failover to update the leader.
##### Logical Logs
- Sequence of records describing writes to DB tables.
- **Insert** - Contains all the values of the row.
  **Delete** - Contains information to uniquely identify the row to be deleted.
  **Update** - Contains information to identify the row and all the column values to be changed.
- Logical Logs are decoupled from the internal logs of the DB -> Do not depend on the internal storage.
- Easy for other applications to consume like Kafka, Cache etc.
##### Trigger based Replication
- Sometimes it is required for the application to have some control over the replication.
- You may want to replicate only portion of data OR replicate to a different type of DB.
- Trigger lets you run a custom code that transforms the logs and then send it to the replicas.
### Replication Lag
- In async replication, there is a lag between the write time and the time at which the replication is completed on a followers. This is the **Replication Lag**.
- Eventually the followers will catch up with the leader. Thus async replicated systems have **Eventual Consistency**.
#### **Issues** due to Replication Lag
##### Reading your own Writes
- Inconsistency if you read the data you just wrote.(**Read-after-write consistency**)
- **Solutions**-
	- Route read requests to leader for pieces of code that only the user can edit(If the amount edited is small).
	- Route all read requests to leader for some time after a write.(*Store a timestamp for every write*)
	  -> Complicated when the same user has multiple devices. Plus they might connect to different followers leading to inconsistency.
##### Monotonic Reads
- Refresh routes the request to a follower with more lag than the previous request follower.
- This leads to updates in data disappear -> Things moving back in time.
- **Solution** - Make sure that the user always reads from the same follower. *If this follower crashes, user blocked*
##### Consistent Prefix Reads
- Violations in causality.
- In sharded DBs, different partitions operate differently so some query which is supposed to execute later on a replica may be executed early leading to break in causality.
#### Solutions to Replica Lag
- The problem is if the lag is too much.
- Use transactions - Ensure all your actions are executed. Although they are too expensive for distributed systems.
### Multi Leader Replication
- More than one leader.
- **Use cases** - 
	- Multi-data center operation - You need one leader per geographical location.
	- *Clients with offline operation* - Eg- Calendar where the app operates even when offline. Then each client has its own local DB which in itself is a leader whose changes are synced once online.
#### Topologies
- **Circular**- Writes are forwarded to the next leader until it reaches back to the original.
- **Star**- Writes are forwarded to a central leader which forwards it to other leaders.
  *To prevent infinite replication, keep track of the last write made.
  If one node fails, it may block updates to other nodes*
- **All to All**- Writes are forwarded directly. *Writes from one leader may reach faster than others and break causality*
#### Write Conflicts
- Happens when two users simultaneously update the same record.
- Eg - concurrent writes, writes routed to different leaders.
- **Conflict Detection** - 
  *Synchronous Detection*- Conflict is detected at a later as the system does not wait for writes to replicate.
  *Asynchronous Detection*- System waits for writes to be copied over to all the leader.
- **Conflict Avoidance** - Best approach as resolution can be tricky. Try routing all requests from a user to one leader.
#### Convergence Mechanisms
- **Last Write Wins** - Have a timestamp assigned to each write and keep only the latest write.
- **Conflict store DS** - Records all conflicts and resolves them when read.
#### Conflict Resolution
- *On Write* - As soon as the system detects conflicts(i.e. on write), it calls the conflict handler.
- *On Read* - When different versions are received by user, it triggers the conflict handler/application logic.
### Leaderless Replication
- All nodes are allowed to accept read an write requests. Any request made will be **routed to all the nodes**.
- No failover required as there is no leader.
- **Stale Data**- If a node goes down and then comes back up, it might lag behind others -> you get stale response.
- *Replication Scheme* - 
	- **Read Repair**- Client makes read requests to all nodes. For the stale values it receives, it updates the nodes.
	- **Anti-entropy process**- A process that constantly checks the values stored across nodes per record and updates nodes where stale value is present.
	- If anti entropy process is not there, some lagged data may never be updated.
#### Quorum based systems
- No need to wait for all nodes to respond for success of read and write requests.
- Configure DB in a way that, writes are acknowledged after *w* successes and reads after *r* successes and *n*=#replicas per key.
- If w + r> n, we expect to get fresh values every time => At least one node will have updated value
- Typically, n=odd and w=r=(n+1)/2. This tolerates (n-1)/2 failures.
- If fewer than w or r nodes are available, writes and reads are blocked.
- **Write heavy systems** - Smaller w, larger r.
  **Read heavy systems** - Larger w, smaller r.
  **Eventually Consistent systems** - w=1, r=1 -> high availability
##### Sloppy Quorum
- Reads and writes may go to any reachable nodes(if designated nodes are down).
- **Hinted Handoff**- Once the node is up, the updates are handed off to it.
- Increases availability, Even when w+r>n, you might get stale values.
##### Limitations of Quorum
- **Concurrent Writes** - Two writes succeed on different replicas, no consensus on which value is fresh.
- **Replication lag** - Only n-w replicas are updated. For the write to propagate there is replication lag. If r nodes are queried for read, they will still give stale data.
#### Dealing with Concurrent Writes
##### Last Write Wins
- Store timestamp for each write, take the latest and discard rest.
- All the reads will be successful, but only one write will be there. 
- All except one clients will have data loss
*To deal with this you need to establish one happened before other relation*
##### Version Number
- This is used to identify the causal relation between writes.
- Each write will be stored along with version number = read VersionNum + 1.
- The reads will come back with a version number as well.
- Works well with single leader replication.
```Java
C1 writes A -> (A,1)
C2 writes B -> DB reads v=1, (B,2)
C1 writes C -> DB reads v=2, (C,3)
```
##### Version Vectors
- Used in multi-leader replication.
- Version number is stored per replica. The set of replicas queried will return a vector of version numbers.
- If for 2 vectors u,v. Then u>v if ui>=vi for all i and ui>vi for at least one i.
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
- Improve scalability by splitting the data up into partitions known as sharding.
- **Skewed Partitions**- Partitions where one partition handles more load than others. This partition is also called the *hot spot*
### Partition Strategies
#### By Key Range
- Each partition holds a range of keys decided by the amount of data.
- Each partition internally has the range of keys sorted to support range queries.
- **Issues**- *Hotspots due to choice of keys*: 
	- Suppose the key is the timestamp, and you are recording sensor readings, all the readings will go to the same partition.
	- Prefix sensor id to the timestamp so the records are routed better.
#### By Hash Key
- Hash gives different values even if keys are similar -> fair division irrespective of choice of key.
- **Issue** - Range queries are difficult.
- A workaround is using **Compound primary keys** - You use one column for partitioning. Inside the partition the records are sorted based on the the other fields of PK so you can search based on other keys.
### Skewed Workloads and Relieving Hotspots
- Sometimes despite using hashes, hotspots may arise -> **Reads and writes to the same key** can overload a partition.
- Prefix/Suffix a 2 digit random number to the **hot key** -> key split into 100 keys -> split across partitions.
- Reads have to do additional work. You need to keep track of such keys.
### Secondary Indexes
#### Local Indexing
- Each partition has its own secondary indexes.
- Reads need to read up all partitions as the value being fetched on secondary index would be spread across partitions.
- Called **gather/scatter** -> very expensive.
#### Global Secondary Indexing
- An index of cols other than PK that spans all partitions.
- Each partition will store a range(or hashed range) of this index.
- Query GSI key = k -> Partition containing GSI key k -> get the value(id of row) -> Partition containing ID.
- Makes *read more efficient* but *writes are slower* and affect multiple partitions.
### Rebalancing Partitions
- If query throughput increases, data changes, the workload changes -> Rebalancing of partitions.
- Usually done manually as automatic can be unpredictable.
- Rules -
	1. Workload distribution should be fair.
	2. Database system should not go down while rebalancing.
	3. Only the necessary amount of data should be moved between systems.
#### mod N
- Redistribute the requests using the remainder of mod n(#partitions).
- **Con** - Most keys will have to move if n changes.
#### Large number of partitions
- Make \#partitions very large. Allow multiple partitions to stay in the same node.
- This will make the the workload split approximately equal.
- If a node fails or new node arrives, it just has to steal a few nodes from an existing node.
- Choosing the right number of partitions is difficult. 
  **Less \#partitions** -> Rebalancing becomes a problem. 
  **High \#partitions** -> partition management overhead is too large.
#### Dynamic Partitioning
- Partitions are split and merged based on partition size.
- You start with a min number of partitions to ensure multiple nodes are available to server requests.
### Request Routing
- As partitions are rebalanced, how does the client/routing guide know which IP/port to route the request to?
- **Service Discovery** - Any service on the network may keep changing the machine its running on. You need a tracker which tracks where which service is located.
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
## Transactions
- Several reads and writes logically grouped. If even one of them fails, all the writes are rolled back.
### ACID
If any of these conditions are violated, DB will rollback.
- **Atomicity** - Writes are grouped in a txn and if any one errors out, all the writes need to be rolled back.
- **Consistency** - There are some constraints defined on data which should not be violated.
  DB can provide some guarantees(uniqueness, type validation) but end to end consistency is handled by the application.
- **Isolation** - Txn should be isolated. Even if many execute concurrently, result should be same as if they executed serially.
- **Durability** - Once the txn has committed, any data written will not be forgotten. 
*Leaderless replication works in a best effort basis where DB will try to revert on txn failure but there is no guarantee*
#### Single Object Transactions.
- All DBs provide Atomicity and Isolation for single objects. These are txn where only 1 object is modified.
- Atomicity is implemented use Logs for crash recovery.
- Isolation is implemented by using a lock for each DB object.
### Weak Isolation Levels
#### Read Committed
- Guarantees - 
	- **No dirty reads** - Txn cannot read any value that has not been committed yet.
	- **No dirty writes** - Txn can only overwrite data that has been committed.
##### Implementation
- **Dirty writes can be prevented by having locks** on every DB object.
- Read locks do not work very well as one txn may have a lot of reads. If this runs very long it slow down DB response times.
- To prevent dirty reads, **DB stores the old committed value and the new value** as well. 
  When **txn tries to read, it reads the old value.**
#### Snapshot Isolation
- *Problem* 
	- If a txn changes 2 objects simultaneously and a read happens when one object is changed and other is not.
	- Data appears inconsistent temporarily -> **Read skew/Non Repeatable read**.
- Ensures that each txn reads from a consistent snapshot(all data was committed) of the DB.
##### Implementation
- *readers never block writers, and writers never block readers.*
- **Multi Version Concurrency Control(MVCC)**
	- several versions of DB object, for each in progress txn. Used for read committed as well.
	- Each txn gets a ID and each object changed is tagged with this TID.
- *Visibility Rules for consistent snapshots*-
	- Any writes by ongoing/aborted txn are ignored.
	- Any writes by any txn that was started later is ignored.
- *Indices*
	- **Approach 1**- Index has all the versions of the key available as well.
	- **Approach 2**- *Append Only / Copy on write B Tree* - When you write, you update each page from root to leaf. Instead of inplace update, create new pages and have parent point to the new page. The new root page becomes your new index root.
##### Stale Reads
**Case 1: Stale MVCC object version**
- T1 reads an object. T2 modifies the object and commits. T1 modifies another object based on what it read.
- DB needs to track all writes that violate visibility rules for any other txn. 
- When any txn commits, the DB checks against all these writes.
**Case 2: Detecting writes that affect prior reads**
- When a txn writes, it should notify all the txns that have acquired the read lock on the index.
### Preventing Lost Updates
- *Lost Update* - 2 txn perform (read-modify-write cycle) concurrently. One of the writes will be lost due to loss of causality.
- **Atomic Writes** - Use lock on db object for the entire (read-modify-write) cycle. Also called *Cursor Stability*
- **Compare and set** - New value is only updated if the value in DB matches the value in snapshot.
- **Automatically detecting lost updates** - If DB detects a lost update, the entire txn is rolled back. Works well with snapshot isolation.(DB can check if an updated row was modified by other txn called a *write write conflict*).
*The above approaches assume there is a up to date version of data available.*
- **Using conflict resolution DS** - All conflicts are stored in a DS and application is responsible for resolving the conflicts. 
### Write Skews/Phantoms
- Generalization of the lost update problem.
- 2 Txn executing concurrently 
  read the same set of rows 
  write to different objects
  together violate an invariant.
  There is no write-write conflict detected in such cases.
- Eg- atleast 1 oncall doctor required. A and B are there. A,B trigger logout -> sees count = 2 -> both log out.
- **Resolution** -
	- **Predicate Locks** - 
		- Create a lock out of conditions of the read query. DB checks every write against this condition internally.
		- Acquire the lock when reading the value and release on txn completion.
		- *Bad Performance*
	- **Index-range Locks** -
		- Simplify the lock to a lock for subset of the conditions out of the set of conditions.
		- You lock range of keys in the index instead of the condition(for range condition).
		- *Better performance*
	- Can be resolved through **Serializable Isolation**
### Serializability
- Highest level of isolation.
#### Actual Serial Execution
- A single thread loop -> serially executing txns.
- Works if all data that you would need can fit in your RAM.
- DB txn should be short -> other txns are not held up. -> *No possible for interactive applications*
- One way to work around this is having predefined *stored procedures* -> Hard to debug, old languages, slow execution.
- **Cross Partition txns** - slower.
#### Two Phase Locking (2PL)
- *Readers and writers can both block each other*
- Locks have 2 different modes - shared mode and exclusive mode
	- To read, a txn must acquire the lock in shared mode.
	- To write, a txn must acquire the lock in exclusive mode.
	- If txn who read want to write, it can upgrade the lock to exclusive mode.
	- Any lock acquired must be held till the txn is complete.
- Process running which detects and resolves deadlocks.
- **Bad Performance** - Overhead of acquiring all these locks. There is also loss of concurrency.(writes block reads and vice versa)
#### Serializable Snapshot Isolation (SSI)
- Full serializabilty but small impact on performance
- Instead of aborting txn violating conditions, it allows txns to proceed and checks the values on commit.
- Adds algo on top of SI to detect serialization conflicts.
  read-write dependencies -> T1 reads something which T2 writes to. T1 -rw-> T2. Checks if these dependencies are dangerous.
- **Has bad performace at capacity** - because many txn are aborted and need to retried.
  Better than 2 Phase Locking as no transaction is blocked while other executes.
## Partial Failures
- Parts of system are broken and unpredictable while others work fine. This is called partial failure.
- Partial failures are non deterministic, i.e., system can give correct and incorrect results based on situation.
#### [Network Failures]([[Networking Basics]])
- Async packet switched networks -> Packet loss, Network delays, Congestion Control delays
- Impossible to tell if another node is working in case of no response.
### Failure Detection
- Nodes detect failure using timeouts. Timeouts are unreliable due to clock inaccuracy,
- Detection is probabilistic. Nodes do not know anything about each other so there is always a chance of false positives.
- Too short timeout → false positives, Too long timeout → slow recovery
### Unreliable Clocks
- Computer clocks are inaccurate and impossible to synchronize across devices.
#### Time of Day Clocks
- Returns the current time and date. Synced with the NTP Protocol.
- **NTP Protocol** - sends network packets that syncs the time based on consensus of a group of servers.
	- If the clock jumps too far ahead it would have to be rewinded back. Hence some can see the time going back.
- **Leap Seconds** - Time on a clock might skip to future/past times.
#### Monotonic Clocks
- Used to measure continuous intervals of time.
- Based on hardware timers per CPU.
- **Clock Skew**- The NTP Protocol may adjust the frequency of the clock if it detects the clock moves too fast/slow.
- **Comparision**- Cannot compare monotonic clocks of two nodes(drift)
### Clock Syncronization and Accuracy
- Syncronization is difficult because of the following reasons
	- The hardware clock is not accurate. It drifts based on temperature, machine.
	- NTP packets also face network delay. Between 2 packets, the computer time may be inaccurate.
- This is why it is **difficult to rely on clocks for versioning and causality**.
- A writes before B but A's timestamp is after B's. -> Wrong.
- It is better to use **[Logical clocks/Version Vectors]([[#Version Vectors]])**
### Processing Pauses
- Reasons for Pauses - 
  Garbage Collection, Page Faults(load from disc), Context Switching/Scheduling(responding process on hold), Network delays
- Pauses last from milliseconds to seconds to minutes.
#### Unreliable Timeouts
- Process paused for 2 seconds but timeout is 0.5s -> Wrongfully declared dead.
#### Unsafe Locks
- Lock Lease lasts for 5s. Paused for 10s. -> Declared dead -> New Lock Holdder -> Unpaused -> 2 Holders
- When this happens for Leader Lock, it leads to **Split Brain**. This leads to data loss(non leader loses writes).
##### Fencing Tokens
- Whenever a lock is given, an incremental number called fencing token is issued with it.
- Whenever a reacquire request arrives, it includes this token. If it is the latest token lock is issued.
### Byzantine Faults
- Fencing token assumes that the nodes are unreliable but honest. When nodes lie, it becomes a problem.
- **Byzantine Fault** - There are n nodes, some of which lie. Lying nodes are unknown.
- **Byzantine fault-tolerant** - continues to operate correctly even if someof the nodes are malfunctioning and lying.
## Consistency and Consensus
- **Consensus** - Making all nodes agree on something
### Consistency Guarantees
- Different data no two nodes because write requests arrive on different nodes at different times.
- **Eventual Consistency** - If you stop writing, all the nodes will eventually return the same value.
- Stronger consistency models come with performance treade-offs
#### Eventual Consistency
- New writes will be replicated but there is no time guarantees.
- High availability and performance, Low consistency.
#### [Sequencial Consistency][[#Total Order Broadcast(Sequential Consistency)]]
#### Atomic Consistency(Linearizability)
- **Atomic Consistency** - DB creates a illusion that there is only one replica. Every action on DB appears atomic.
- **Recency Guarantee** - As soon as a write completes, all the reads have the access to the latest data.
- Implemented using Consensus Algorithms
##### Uses
- **Leader Election** - Leader is elected by leasing a lock. This lease operation must be linearizable -> all nodes must agree on who the leader is. 
- **Constraints and Uniqueness** - Username/email must be unique -> all nodes must agree that a username is available -> Linearizable.
- **Cross Channel Dependencies** - If there are multiple channels of communication between 2 services in parallel. Eg - You write on DB and then send a msg on queue -> If not linearizable, race condition between msg queue and replication of DB.
*Single Leader - possible, Consensus - Linearizable, Multi-Leader - Impossible, Leaderless - difficult*
##### Cons
- Requires coordination between nodes -> costs performance
- High latency (network round trips to coordinate data).
- Under partition → must sacrifice availability(To coordinate writes reads will have to be blocked).
### CAP Theorem
- Consistency, Availability and Partitions tolerance cannot be guaranteed together at all times.
- **Limitations** - Here consistency refers to linearizability and availability is only one of the system faults that can occur.
### Ordering Guarantees
#### Total Ordering
- Guarantees that all the writes will be ordered in the order that they were created.
- Difficult to implement with distributed systems. On single node systems, can be implemented using locks and logs.
- If a system implements linearizablity, it means there is a total order of requests.
#### Causal Ordering
- The order only matters if a request influences the consequent writes.
- Causal order ensures that dependent requests are ordered but unrelated events might not be ordered properly.
- Most systems usually only require causal ordering guarantees.
- **Single Leader** - Use sequence numbers/ Timestamps/ Logical Clock to order the writes.
- **Multileader/leaderless** - Generating causally ordered sequence numbers is difficult -> writes can go to any node.
  Use time of day clock for timestamp -> Clock Skew. 
  Numbers generated are based on number of requests arriving. Requests face network delays -> no guarantee of causality.
##### Lamport Timestamp
- (nodeId, counter) -> Counter decides which request is causally greater. If two have same counter, greater nodeId wins.
- Each node keeps track of its own counter. If any request has a greater counter, it updates its own counter to that value.
- Ensures causal order as follows - Request1 - (2,1), Request2(dependent on 1) -> (1,2). Request3(independent) -> (2,2).
  R3 occurs before R2. But L(R3) > L(R2). -> 
  If L(A)>L(B) that does not mean B occured before A. But if B occurred causally before A then L(B)<L(A).
- **Logical Clocks** on the other hand ensures order of operations to detect concurrency.
### Consensus
- Refers to making nodes agree on something.
- A consensus algorithm must satisfy the following properties - 
	- **Uniform agreement** - No two nodes decide differently.
	- **Integrity** - No node decides twice.
	- **Validity** - If a node decides value v, then v was proposed by some node.
	- **Termination** - Every node that does not crash eventually decides some value. It is assumed that if a node crashes it does not come back online. If a algo waits for nodes to come back, it violates termination(node may never comeback).
- Termination is subject to the assumption that less than half of the nodes will crash at a time.
- Byzantine faults can also be handled if less than 1/3rd of the nodes lie.
#### 2 Phase Commit
- **Atomic Commit** - Ensure atomicity for a distributed txn go to commit.
- Uses a coordinator/txn manager which is a separate service.
- Txn wants commit -> coordinator sends a *prepare request* to all nodes.
- All nodes reply yes -> *commit request* sent to all nodes -> Node must make sure it commits(Yes reply only after txn logged to WAL)
- Any node replies no -> *abort request* to all nodes.
- If any prepare request fails/timeouts, the coordinator aborts the txn.
  If any commit/abort request fails/timeouts, the coordinator must keep retrying until a response is received.
- **Violates the Termination property** of consensus algorithms.
##### Issues
- Coordinator is a single point of failure. If it crashes, the entire DB crashes.
- Poor performance as each txn has to wait through atleast 2 network calls.
- If any node goes down the entire DB is stalled 
- Txn stuck due to retries -> holds locks it leased which blocks other txns.
#### Total Order Broadcast(Sequential Consistency)
- Causal Ordering is not sufficient when you want to perform uniqueness checks. You need to know the total order.
- TOB is a protocol for sending messages between nodes which has 2 guarantees -
	- *Reliable Delivery* - No messages will be lost
	- *Total Ordered Delivery* - Messages will arrive in the order they were sent.
- Consensus algorithm. Can be thought of as a log with messages as entries.
- Implemented by zookeeper/etcd.
- Asynchronous in nature.
##### Uses
- **Replication** - To send writes across replicas
- **Fencing Tokens** - Each request to acquire a lock is appended as a message in the service's queue.
- **Linearizable Storage** - 
  As total order is ensured, To check for uniqueness you use compare and set commands.
  (Username will have 0 if unoccupied. Set to 1 if occupied. All subsequent requests will fail).
  This does not ensure linearized reads -> Async requests so reads that occur before writes are broadcasted will be stale.
##### Implementation
- Linearizable register and compare and set operation. Every message can be assigned a id using this combination. This ensures order.
- If a node recievs 6 before 5, it waits for 5.
##### Consensus and Total Order Broadcast
- Total Order Broadcast can be thought of as multiple rounds of consensus. 
- In each round we make nodes agree on which message will flow to which node.
#### ❌ Epoch Numbers
- All Consensus algorithms internally use a leader in some form.
- They define a epoch number and node having the same epoch number will have a unique leader always.
- For any election/any decision by leader, the node trying to be the leader will send a request to all nodes and wait for response.
- If the response reveal a node with a higher epoch number, then the node is not declared the leader/proposal is rejected.
#### Limitations of Consensus
- Voting for leader election/proposal, all are synchronous -> costs performance due to network delays and halting of DB.
- Atleast more than half nodes should be functioning for consensus to work.
- Consensus uses timeouts to detect failed nodes -> Some nodes may be declared dead falsely.
#### Membership and Coordination Services
- Zookeeper, Kafka, etcd provide coordination services.
- They are key value stores with small enough data to fit in memory.
- They implement Total Order Braodcast.
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
- MapReduce works in the following way -
	- Read input files and break them up into records.
	- **Mapper** called per record -> Extract key-value from each record. 
	- Sort all the key-value pairs
	- **Reducer** called per key-value pair -> Produce output based on key/value
- **MapReduce Workflows** - chain several jobs together -> Ouput directory of one job becomes input of another.
#### Distributed Execution of MapReduce
- There is a **Master node** which schedules tasks and handles worker failures.
- **Worker nodes** - Execute map and reduce tasks and store data locally.
1. **Input Split** - Input is split into partitions. Each node may have multiple input splits stored.
   One Map task is run for each input split.
2. **Map Phase** -
   Each map task converts the records in the split into key-value pairs. The logic is written by user.
   Output is written to a file on disk.
3. **Shuffle and Sort** - 
   Mapper Output is ordered by keys and exposed over the network.
   Reducer nodes fetch their partition of keys from each node.
4. **Reduce Phase** - 
   Key value pairs are reduced into the required information based on the user code.
#### Fault Tolerance
- If Mapper fails, map task is rerun on a different worker(as map is deterministic). 
- If Reducer fails, reducer is rerun -> Fetch data from nodes -> generate output.
### Joins
#### Reduce-Side Join
- Records may have foreign key references and final output would require data from the referenced records.
- Query the DB once per record.
- Use machine copy of DB instead of network calls to remote DB. The copy can be generated using Warehouse ETL process.
#### Sort-Merge Join(Reduce-Side Join)
- Map arranges the record such that the dependent records come later in the order(Causal Ordering).
- Eg - Orders require User data. User record followed by all the orders of the user.
- Reducer keeps the dependee records in memory.
- **Skew/Hotspot** 
	- One key has a lot of records in the second record set(*Hot Key*) -> Overloading of one Reducer. 
	- Use Key-Salting -> Spread the same user over multiple keys and distribute orders accordingly.
#### Broadcast Join(Map-side Join)
- Small dataset is kept in memory. Large dataset is set for batch process and all references are mapped in-place.
- Leads to no reduce phase as all data is processed in the map phase itself.
- Faster, No shuffle cost.
- **Cons** - Table must fit in memory.
#### Partitioned Join(Map-side Join)
- Partition both datasets using the same kay(on which join happens) -> Related records are present in the same partition.
- Use the same Hash function and sort based on the same key -> Automatic ordering of the records.
- No shuffle/reducer is required.
- Very efficient
- **Cons** - Preprocessing required. Harder to operate.
### Output of Batch Processing
#### Atomic Output Replacement
- In case of workflows, another job reads the current job's output.
- The downstream job should only be able to see either the old data or the new data. Partially written data should be hidden.
- So you write to a new file and on success, you atomically rename the file.
- This also helps in versioning of the job output.
#### Side Effects
- If Reducer directly calls a side effect, failures will lead to duplication issues.
- Output should be deterministic -> Reducer not allowed to have side effects -> downstream job calls the side effect.
#### Exactly-once semantics
- Batch processes achieve this using Deterministic processing, Full Replacement Output and Avoiding in-place updates.
- If a job fails you just rerun it.
### Distributed DB vs Hadoop
- DB stores data in structures. MapReduce has no such requirements input is a sequence of bytes.
- DB is optimized for low latency queries. Hadoop is optimized for high throughput processing of entire data.
- DB usually services OLTP loads. Hadoop services ETL jobs, Log Analysis etc.
- Hadoop stores files in HDFS, with variable schema and immutable datasets.
### Problems with MapReduce
- **Intermediate States** - When you have 2 jobs queued, the output of 1st one has to be written on the HDFS before job 2 can start. This is a intermediate state which is expensive and redundant as it is temporary.
- **Redundant Mapper** - Most times, mapping logic can be part of the previous producer.
- MapReduce jobs can only start once the previous dependent jobs are all finished.
- Sorting might not be required but mappers have inbuilt sort which is expensive both memory wise and complexity wise.
### Dataflow Engines
- Instead of using map and reduce functions, they define *operators*, which run on the files to generate output.
- Parallelize the work by partitioning same as mapReduce.
- No unnecessary map tasks and sorting is optional.
- Can store the intermediate state in memory or on disc not in the HDFS.
- Operators can start executing as soon as the input is ready. No need to wait to write it out to HDFS.
#### Fault Tolerance
- Since there is no intermediate state, durability is reduced. Task failures now have to reconstruct the state.
- To reconstruct the state, the **operator must be deterministic** and the system needs to remember what operations it performed in which order.
- If a fault occurs and the operator is not deterministic, all subsequent jobs need to be restarted -> **Cascading Failure**
### Graph and Iterative Processing
## Stream Processing
- **Event** - Every record is also known as an event. Contains a timestamp, uniqueId(Idempotency) and key(for partitioning).
- **Schema Evolution** - Event schema evolves over time. Events need to be backward and forward compatible. So add/remove optional fields only.
### Producer 
- publishes an event. Attaches a sequenceId(UID) to the event
#### Processing Guarantees
- **At-most-once** - No retries. Dataloss is possible.
- **At-least once** - Retries are allowed. Possible duplicate processing of messages.(If a later message arrives earlier, it will be retried -> 2 processing cycles)
- **Exactly once** 
	- No loss and no duplicates 
	- Requires
		- *Transactional writes* - Multiple writes across partitions must either all succeed or all fail.(Distributed txn).
		- *Idempotent producers* - Retries dont produce duplicates messages.
		- *Atomic offset and output commits* - Consumer produces output and commits offset. If it crashes before commiting output, message will be retried. -> Ouput and offset commit should be atomic.
##### Idempotence
- Idempotent Operation -> Even if you perform it multiple times, result is the same as performing it once.
- Make any event idempotent -> Use (procuderId, sequenceId) -> If broker processed it before, reject the event.
### Message Brokers 
- It is a distributed append-only log optimized for streamsing.
- Defines offset for each event which is tracked by the consumer.
- **Offset** - Each message is given a sequence number which is increasing. Order is guaranteed within the partition.
- **Durability** - Depends on the broker -> some keep messages in memory while others write them to logs.
#### Partitioning
- Logs can be partitioned, each machine has 1 partition. A set of partitions can be dedicated to a Topic.
- Every partition is assigned to one consumer in the consumer group.
- Although one consumer can consume from multiple partitions.
- Offsets guarantee the ordering of messages within a partition.
### Consumer
- processes the event. 
- Maintains the offset of messages it has consumed.
- *Backpressure* - If consumer is slow, broker must slow down the event production.
- Kafka has a pull system -> Consumer controls read rate not the broker.
#### Consumer Groups 
- Groups of consumers assigned to process through the events of the same topic.
- Types of parallelization
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
### Circular Bounded Buffer
- Logs are only deleted after the disk is full.
- So if a consumer falls behind such that the offset is pointing to a deleted log, it can lead to loss in messages.
- However, the disk is very large -> Long time before messages get dropped -> Manually reset the consumer.
### Change Data Capture
- Used to keep data in different systems in sync.
- Data changes to the DB are extracted and sent over a message broker to various data systems.
- A connector reads DB's WAL and produces to a broker, all systems are consumers of this broker.
- There needs to be an initial point from where the entire system will be built - 
	- **Snapshot with offset** - Initial state of DB, build from this snapshot then consume the CDC. *Expensive, Point in time consistency*
	- **Log Compaction** - A full view of the state, build from this and then consume CDC. *Continuous and latest data*
#### External Side Effect Problem
- If consumer has a side effect of message processing(Eg- write to DB, notifications), retries will lead to duplicate issues.
- Use Idempotent Producers.
- **Transactional Outbox Pattern** - Event processing and side effect are handled together as a txn.
### Event Sourcing
- All interactions with the application are logged as a stream of *immutable* events.
- Instead of storing the data, you store the operations happening on the data. The state can be arrived at by going through the logs.
- **Immutable Events** - Events cannot be modified.
- **Append-only Log** - Events are written sequencially to the log.
- **Derived State** - State can be derived by going through the sequence of events.
- Command(issued by user) → Validate → Generate Event → Append to Log → Update Read Model
#### Advantages
- **Auditablity** - As events are immutable, the entire history of changes is available which helps in audit.
- **Debugging and Replay** - Immutability makes debugging easy. You can just *replay the stream to rebuild* the state in case of *crash.*
- **New Views and Easy Migration** - Making new views is simple, just run the stream and apply new filters to the data. Migration is simple because you can add your changes with the new filters.
#### Disadvantages
- **Complex**
- **Migration needs to be backwards compatible**
- **Replay is costly** - You need to store snapshots which is costly.

### Reasoning about time
- Each even needs to tagged with a timestamp. 
- **Event Time** - Time at which the event occurred. **Processing Time** - Time at which the event is processed.
- Processing time is different due to network delays, wait in queue etc.
- Confusion between event time and processing time can lead to bad data. Eg - Measuring rate of requests -> If processing time, a server coming back up will process request faster(due to backlog) giving the illusion of a network spike.
- **Straggler Events** - Event of an earlier timestamp arrives when the processor is computing the metrics of a different one.
### Stream Joins
#### Stream-stream join
- Joining of two streams. Usually involves a time window.
- Eg- Stream of user searches and Stream of purchases. Say you need to track number of purchases within 5min of a search.
- Processor maintains a state in memory. In our eg, we track the last search and count the number of purchases.
#### Stream-table joins
- Joining of a stream with a changing table.
- Eg- Stream of orders, you need to stamp the address based on the userId passed.
- Usually done using a CDC for the table changes. So it is very similar to a stream-stream join.
#### Table-table join
- Involves 2 CDCs.
- Used for maintaining derived views(indices etc), combining multiple CDCs.
### Fault Tolerance
How do we deal with consumer crashes
#### Microbatching
- Break streams into batches and retry the batch if anything fails.
- Small batch size -> greater scheduling overhead, Large batch size -> greater delay in when the results are available.
#### Checkpointing
- Checkpoints are generated by writing the state to the disk. If crash -> restart from the last checkpoint.
#### Stateful Processor Failure
- A stateful processor must ensure that it is able to restore its state post failure.
- Replicate the state by storing it in a different machine so that a new machine can take over.
- Use logs to write out the state changes so that it can be rebuilt.
#### Dead Letter Queue
- If message fails repeatedly, you put it into the DLQ.
- This prevents the pipeline blocking and infinite retry loops.