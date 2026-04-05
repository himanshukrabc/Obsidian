## RPC
- Caller sends the params and function to call to the RPC stub(code for send/receiving data).
- RPC client constructs a msg, sends it to the server and waits for the response.
- Server's stub unpacks msg, calls the procedure and sends the result back to the caller.
- Caller unpacks the msg and gets back the result.
## Consistency Models
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
## Failure Models
1. **Fail Stop** - Node fails but can communicate with other nodes.
2. **Crash** - Node fails and cannot communicate with other nodes.
3. **Temporal Failures** - Slow responding nodes.
4. **Byzantine Faults** - The node behaves unpredictably sending correct/incorrect results randomly. Caused by malicious attack/bug.
## Non Functional Characteristics
### Availability
- Percentage of time the service can be accessed by the clients.
- Availability - 90%(1 nine)->10% downtime. 99%(2 nines)->1% downtime. 99.9%(3 nines)->0.1% downtime.
	- **Redundancy** - If service fails, we replace the service. So the system is always available.
	- **Fault Tolerance** - The sys should function even if some components fail.
	- **Rate Limiting** - Restricts the number of calls a user can make to control the load on a server.
	- **CDNs** - Cache servers distributed across regions -> Static content served faster + cache some API resp.
	- **Stress Testing and Monitoring**
### Reliability
- Probability that the service will perform properly within a time interval under varying operating conditions.
- **Mean Time To Repair** = (Maintenance Time)/(#Repairs)
  **Mean Time Between Repairs** = (Total Time - Total Downtime) / (#Failures)
- Reliability is how consistently a service operates without failure. Accessibility is how often it is available
### Scalability
- System's ability to handle increasing number of users without compromising performance.
- Ensures spikes in traffic can be handled effectively.
- **Types of workload** - **Request Workload** and **Data/Storage Workload**
- **Dimensions** - *Size Scalability*, *Administrative* and *Geographical*
- *Vertical Scalability* -> Upgrade the hardware resources of system. -> **Simple but Costly**
- *Horizontal Scalability* -> Setup new machines to distribute the load. -> **Cheap but Complex,Latency**
- *Autoscaling* -> Monitor CPU usage, Network traffic etc to spin up new machines to handle the spikes.
#### Scalability Techniques
- **Load Balancing** - Distribute load across machines to stop single node overload.
- **Caching and CDN** - Reduce number of calls to DB using cache. CDN serves static data which reduces server calls.
- **Data replication** - Duplicate data across machines to split the load. Improves scalability and fault tolerance.
- **Data sharding** - Spilt data across machines to split the load. Improves scalability and performance.
- **Microservices** - Split the services across machines so that each one can scale independently.
### Maintainability
- How easy it is to maintain the system after building.
	- **Operability** - Ease of operation under normal conditions and to return to normal under failure.
	- **Lucidity** - Simplicity of code.
	- **Modifiability** - Capability to modify and adapt to new changes without faiures.
### Fault Tolerance
- Ability of system to continue performing even when some of its components fail.
- Ensures Availability and Reliability of the system.
- **Error Recovery** - *Backward*(Restore to previous version) or *Forward*(Identify error and resolve)
- **Error Masking** - Introduce code that masks the other error.
- *Replication* is widely used for fault tolerance
	- Swap out failed nodes/DB with healthy ones => System becomes unavailable.
	- Async Replication => Stale data in replicas => Eventual Consistency.
- *Checkpointing* - save state in storage for failure backup. Performed in regular intervals,
	- **Consistent state** - System must have consistent view of sequence of events.
		- All updates upto this are saved. In-progress rolled back.
		- The checkpoints across processes must be coherent.(No extra event must me processed by any node).
### Some Examples to achieve NFCs
#### Performance
- Ability of system to respond to user requests and process data effictively.
- **Caching** - 
	- X uses caching for the timeline service. 
		- For inactive users, timeline is generated instantly by reading through the DB.
		- For active users, timeline is cached.
		- Celebrity posts are appended to the feed(**fan-out on read**)
		- Normal user's post are fetched by reading from the DB(**fan-out on write**)
- **DS/Algo Selection** -
	- Uber - DS to choose for *Geolocation*(storing driver positions) -> **Quadtree**
		- Each node represents a sqare area. Each node has 4 childern(NE,NW,SE and SW quadrants).
		- Each node stores - (Area it covers, # points, pointer to children).
		- For the leaf node, if # points >threshold, split into 4 quads.
		- **Efficient Range Queries** and **Proximity Detection**.
- **Load Balancing** - 
	- Distribute incoming traffic among different servers. Prevents bottlenecks.
## Back-of-the-Envelope Calculations
### Latency Numbers
CPU cache   1 ns
RAM         100 ns
SSD         100 µs
Disk        5 ms
Network     1–100 ms
### Throughput Numbers
RAM bandwidth         10–50 GB/s         
SSD sequential read   ~1 GB/s            
HDD sequential read   ~100–200 MB/s      
1 Gbps network        ~125 MB/s          
10 Gbps network       ~1.25 GB/s         
### Time Conversion
1 day = 86400s
### System Capacity Numbers
Single server QPS   10k–100k           
Redis QPS           100k–1M            
Kafka throughput    1M msgs/sec  
MySQL writes        10k/sec (rough)    
### Traffic Estimation
Peak QPS ≈ DAU × 0.1 / seconds in day
### Cache Rates
Cache hit rate	80–95%
CDN hit rate	90%+
Memcached latency	<1 ms
## Domain Name Servers
- Resolves human readable domain names to IP addresses. Every request requires DNS resolution.
- It is a distributed DB where servers communicate through UDP. 
	- **Low latency** -> response from nearby DNS servers.
	- No Single Point of Failure.
	- High availability
- Hierarchy of DNS servers -> 
  **Root Server** - Contains addresses of TLD Servers. Total 13 
  **Top-Level-Domain(TLD) Sever** - Stores address of Auth servers storing domain with given TLD. Eg- .org, .com etc
  **Authoritative Name Server** - Contains actual translations of host vs IP.
- Queries to DNS -
	- **Iterative**- User contacts root,TLD and auth DNS servers.
	- **Recursive**- User contacts local DNS server, internally requests are routed to Auth server. 
- **Caches** the frequently accessed domain values to reduce load on actual DNS servers.
- **Easy to scale** - Due to hierarchy, you can keep adding auth servers.
- **Reliability** - Caching, Replication and UDP allow DNS to be highly reliable.
- **Eventual Consistency** - New domains are propagated asynchronously.
- **DNS For GSLB** - 
	- DNS reorders the list of IPs for each domain in a round-robin fashion. 
	- Each user will read the list serially, so this prevents overloading of single server(Load Balancing).
	- *Cons* 
		- Due to different ISPs, many users will still attempt on the same Server(**Uneven Load**)
		- DNS does not track server failure. Caching delays updates to IP.
		- DNS packet size is small so all IPs are not included.
## Load Balancers
- Divides the requests fairly amongst the available servers.
- **Scalability** - New nodes can be added easily
  **Availability** - Even if node goes down, route to other nodes
  **Performance** - Forward requests fairly to avoid overloading servers
- Client -> LB -> Webservers -> LB -> Application Servers -> LB -> DB Servers
#### Services offered by Load Balancers -
- **Health Checking** - Checks for dead nodes using heartbeat protocol.
- **TLS Termination** - Handles TLS Termination reducing burden on servers.
- **Service Discovery** - Stores the IP of services(as the keep going down and coming back up).
- **Request Analytics**
#### Type of Load Blancing
- **Global Server Load Balancing** - 
	- Distributes traffic across multiple geographic locations.
	- Internally it routes request to different load balancers based on geography.
	- Usually DNS is used for Global Server Load Balancing.
- **Local Load Balancing** -
	- Local load balancing distributes traffic **within a single data center**.
	- Acts as a **reverse proxy** -> Client connects to LB's IP
#### Load Balancing Algos
*Even if request distributed this way, Some servers may run hot. Eg- long running queries by the same client.*
- **Least Connections** - Assign request to a server containing the fewest existing connections. -> Load off the server serving more heavy duty requests.
- **Least Response Time** - Server with least resp. time serves the request.
- **Round-robin scheduling** 
	- Requests routed to server in a repeating sequencial pattern.
	- **Weighted Round Robin** - Powerful servers are assigned weights and requests proportional to weight.
	- Hash(IP)%N = nodeId will decide which node the request is routed to.
	- **Con** - If you have to add a new server, almost all keys will route to a new server now.
- **Consistent Hashing**
	- Servers and request hashes are placed in a ring.
	- For a request, move clockwise to fin the server that will handle the request.
	- Now if a node fails/new node added, just move over the request to the next server.
	- Each server is assigned multiple hashes(**Virtual Nodes**) so on failure only a few of the keys are moved.
- *Static Algos* - dont consider state of server.
  *Dynamic Algos* - consider state of server as well.
#### Stateful vs Stateless Load Balancing
- **Stateful Load Balancing**-
	- Involves maintaining a state of the sessions. This state is shared across LBs.
	- Increases complexity and limits scalability
- **Stateless Load Balancing**-
	- No state is maintained. Uses consistent hashing.
	- Fast, Light weight and Scalable.
#### Types of Load Balancers
- **Layer 4 load balancers**-
	- Works at TCP/UDP layer. Uses IP/Port and connection details to route the request.
	- Forwards the entire connection to the server -> All requests go to the same server.
	- Once a connection is assigned → **all packets of that connection go to the same server**.
	- *High Performance, Low latency and simple connection distribution*.
- **Layer 7 load balancers**-
	- Works at application layer. Uses URL, HTTP Headers, cookies etc to route the request.
	- Reads the HTTP request before routing.
	- Helps in rate limiting, API routing, authentication etc.
#### Load Balancer Deployment
- **Tier 0**- *DNS* -> Used for georouting.
- **Tier 1**- *ECMP(Equal Cost Multipath) routers* -> based on IPs.
	- Distributes requests across multiple LB machines. Provides **Horizontal Scalability**.
- **Tier 2**- *L4 Load Balancers* -> Uses TCP/UDP level details.
	- Ensures all packets go to the same backend LB => *uses Consistent Hashing*
- **Tier 3**- *L7 Load Balancers* -> Uses HTTP details.
#### Implementation of Load Balancers
- **Hardware Load Balancers**
	- Dedicates physical devices for load balancing.
	- **Pros** - High performance, can handle large number of connections.
	- **Cons** - Expensive, Requires Hardware Failover, Hard to configure.
- **Software Load Balancers**
	- Load balancers implemented **as software running on commodity servers**.
	- **Pros** - Cheaper, Easy to program, Easy Failover, Horizontally Scalable.
- **Cloud Load Balancers**
	- Cloud providers offer **Load Balancing as a Service**
	- **Pros** - Fully Managed, Auto Scaling, built in monitoring.
## Databases
- File system ->
	- No concurrency support.
	- User permissions cannot be managed, search of files is difficult.
	- Not scalable.
- DB is a organized collection of data facilitates storage and retrieval.
- Adv - Security, Scalability, Availability.
### Relational DB
- Stores data in rows and columns.
- Rigid Schema.
- Provides ACID guarantees.
	- **Atomicity** - All operations succeed or none happen. Implemented via WAL.
	- **Consistency** - DB constraints must hold. Application responsible for business level consistency.
	- **Isolation** - Concurrent txns must execute as if they executed serially.
	- **Durability** - Once committed -> data persists through crashes. Implemented using Logs.
#### Advantages
- **Reduced Redundancy** - Normalized data, reference by foreign key -> No repeated data.
- **Concurrency** - Multiple txns can be executed concurrently.
- **Backup and Disaster Recovery** - Guarantees consistent state due to txns -> Backups are easy.
#### Disadvantages
- **Object Relational Mismatch** - Data is stored in tables, used as objects => Impedance mismatch.
### No-SQL DBs
- Stores data is documents/key-value pairs.
- Flexible Schema -> Good for unstructured data.
- Provides BASE(Basically Available, Soft State and Eventual consistency)
#### Advantages
- **No Mismatch**
- **Proximal Data Storage** - Related data stored together + less joins -> Fast reads.
- **Availability** - Horizontally scalable leading to high availability(failed nodes can be swapped).
- **Cheaper than SQL databases**
#### Types of NoSQL DBs
- **Key Value Stores** - Data stored in key-value pairs.
	- Key is a unique Id and value can be any complex object.
	- Efficient for session oriented data -> Store data per sessionId.
	- Eg- DynamoDB, Redis and Memcached DB
- **Document DBs** - Stores data in documents, whose structure can vary.
	- Suitable for unstructured data like JSON.
	- Eg- MongoDB, Google Cloud Firestore
- **Graph DBs** - Data stored as nodes and edges represent relations between them. Each can store props.
	- Used in social media like data.
- **Columnar DBs** - Stores data oriented by columns instead of rows.
	- Works well for analytics and summarizing large DBs.
	- Eg- Amazon Redshift, Google BigQuery
#### Disadvantages
- **No strong integrity**
### When to use what?
|                                                                    |                                                     |
| ------------------------------------------------------------------ | --------------------------------------------------- |
| **Relational Database**                                            | **Non-relational Database**                         |
| If the data to be stored is structured                             | If the data to be stored is unstructured            |
| If ACID properties are required                                    | If there’s a need to serialize and deserialize data |
| If the size of the data is relatively small and can fit on a node) | If the size of the data to be stored is large       |
## Replication
- Keeping multiple copies of data.
- **Advantages**-
	- Keeps data geographically close to reduce latency
	- Ensures fault tolerance.
	- Scalability
#### Sync Replication
- Leader waits for replication to happen on all nodes then acknowledges the write.
- Consistency is ensured but leads to high latency(if a node does not respond, leader waits)
#### Async Replication
- Leader writes and acknowledges. Writes are then forwarded to other servers.
- Makes system available at the cost of consistency of writes. 
- Leads to data loss if node crashes with unreplicated writes.
### Leader-Follower Replication
- Good for read-heavy application with small number of writes.
- **Read-resilient** - Reads can still be handled even if leader goes down.
- **Inconsistent** - Replication Lag leads to inconsistency.
#### Replication Methods
- **Statement Based** 
	- SQL statements are run first on Leader and then sent out to the followers.
	- **Cons** - Non-deterministic functions(NOW() or RAND()) -> inconsistencies.
- **Write Ahead Logs(WAL)**
	- Writes are written to WAL in disk then run on leader then on followers.
	- Writes to WAL are physical disk changes -> pages vary across DB versions -> incompatible of versions.
- **Logical Logs**
	- Exact rows are written out to the logs
	- **Insert**->Exact row data written. **Update**->Id + row data. **Delete**->Id
#### Cons
- If leader goes down, new leader will not contain all writes of old master.
- Inconsistency due to replication lag. => Route to leader for some time after write.
### Multi leader Replication
- Multiple leaders accept writes. Writes are forwarded to all other nodes.
- Used in application where writes need to be tracked across offline devices -> Each device is a leader.
- Once online, replicate changes across DB.
#### Write Conflicts
- Due to multiple leaders, concurrent writes can be made to the same object -> **conflicts**
- **Conflict Avoidance**- can happen when the requests are on the same server.
- **Last Write Wins** - 
	- All requests are assigned timestamps to generate order of requests.
	- Clock skew/sync problems lead to issues.
	- Use vector clocks to detect conflicts.
- **Custom Logic** - Track rw dependencies on the DB txns. When a loop is found, abort the last txn to commit.
### Multi Leader Replication
- All nodes can accept writes -> Increases write scalability,
#### Quorum of Reads and Writes
- Each request key is hashed and routed to a designated group of nodes.
- Let n = replicas be responsible for the request, w =#write acknowledgments, r =#read acknowledgments
  If **w + r> n**, we expect to get fresh values every time => At least one node will have updated value.
- *Sloppy Quorum* - If fewer than w or r nodes are available, writes and reads are blocked
	- If a designated node goes down, you route the requests to a different node.
	- When the node comes up, you pass on the writes -> **Hinted Handoff**
	- Can lead to stale reads(Node having fresh value is down).
- **Write heavy systems** - Smaller w, larger r.
  **Read heavy systems** - Larger w, smaller r.
  **Eventually Consistent systems** - w=1, r=1 -> high availability
## Partitioning
- Each node manages some part of the whole data -> Sharding/Paritioning.
### Vertical Sharding
- Each node stores subset of columns of the table.
- Used to speed up retrieval of columns containing large text/blobs.
- Usually requires manual partitioning.
### Horizontal Sharding
- Split the table row-wise
#### Key-range partitioning
- Each partition handles a range of *partition key*.
- Other table entries which reference the partition key are all stored on the same node.
- **Advantages**- Range based queries are easy to implement.
- **Disadvantages** - 
	- Range queries other than partition key is difficult to perform.
	- If keys are improperly selected, some nodes may be overloaded due to more no of requests.
#### Hash-based partitioning
- Hash the partition key and mod it by N.
- **Advantages** - Keys are uniformly distributes across nodes.
- **Disadvantages** - Cannot perform range queries
#### Consistent Hashing
- Place requests, nodes on a circle and route the request to the next node.
- Each node has multiple hashes
### Rebalancing Partitions
- Rebalancing - redistribution of data when a node goes down/comes back up.
- **Hash mod N** -> Rebalancing causes almost all keys to move.
- **Fixed number of partitions** 
	- Create a large no of partitions, store multiple partitions on the node.
	- Redistribution -> move some partitions to new nodes.
- **Dynamic Partitioning** -> If the data increases, split the partition equally into two nodes.
### Partitioning and Secondary Indices
#### Local Indexing
- Called **gather/scatter** -> Each partition has its own secondary indexes.
- **Expensive** - Each partition has to be queried.
#### Global Secondary Indexing
- Global index of the column which is partitioned independently across nodes.
- **Read** -> Go to partition storing index range with key. Then go to the partition which has the value of key. -> *2 Reads* -> *Faster*
- **Writes** -> Update the partition and the GSI. -> *2 writes* -> *Slower*
### Request Routing
- As partitions are rebalanced, how does the client/routing guide know which IP/port to route the request to?
- **Service Discovery** - Any service on the network may keep changing the machine its running on. Tracks where which service is located. Eg - Zookeeper
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
### DB Tradeoffs
#### Centralized DB
- **Advantages** - 
	- Maintanance is easy.
	- Strong consistency guarantees.
	- Efficient for companies with small amount of data.
- **Disadvantages** - 
	- Single point of failure.
#### Decentralized DB
- **Advantages** -
	- Low latency as data retrieved from nearest DB.
	- Parallelization of requests
- **Disadvantages** - 
	- More network hops -> Increased network calls -> More network latency
	- Cross node joins are very difficult.
	- Consistency is difficult to maintain.
## Key Value Stores
- Supports get and put functions.
### Functional Requirements
- **Configurable Service** - Provide options for users to use different consistency models.
- **Always available to write**
- Seamlessly integrate new servers and take into account their capabilities.
### Non Functional Requirements
- **Scalability**
- **Fault Tolerance**
### API Design
- get(key) -> Returns value stored for the key.
- put(key, value) -> Stores the value. May store additional info like the version. Value can be of any datatype.
### Adding Scalability
#### Adding Nodes
- We have the system with one server, we have get and put requests.
- To add scalability, you add multiple nodes. Hash the requestId and divide the requests amongst nodes.
- Use consistent hashing to divide amongst the nodes.
	- Use virtual nodes -> easier and fair redistribution of the requests if a node dies or new node arrives.
#### Adding Replication
- **Leader-follower approach**
	- Speeds up reads. 
	- Leader is a single point of failure. Due to replication lag, there will be stale reads.
- **Leaderless approach**
	- Inefficient and costly to replicate data to n nodes.
	- We decide the replication factor = n(usually 3 or 5). 
	- We use consistent hashing -> get the next n nodes after hash(request) for quorum.
	- Once you write, the w/n nodes will ack. Then when you read r/n nodes will ack.
	- We are going with this approach.
#### Handling Temporary Failures
**Sloppy Quorum and Hinted Handoff**
- If the designated node goes down, you write the query onto another node, hinting the actual node.
- This write is then handed to the original node when it comes online.
#### Versioning
- Say a request arrives for A, B and C. Now A was down so D handled the query.
  Now D goes down and another request arrives -> E handles the query.
  Conflicting versions of data from D and E are handed to A. => Reason: **Loss of causality**
- **Vector Clocks** 
	- Used to define causality and detect conflicting writes.
	- Each object is versioned with a vector => [A:1,B:1,C:1].
	  Now when D handles the query -> [A:1,B:2,C:2,D:1]
	  Now when E handles the query -> [A:1,B:3,C:3,E:1]
	  Now the above two vectors are not comparable -> Write conflict.
	- Client is sent the conflicting writes and it reconciles the data -> [A:1,B:3,C:3,D:1,E:1]
	- **Cons** - Version vectors can grow very large in size -> *Store counters with a TTL* -> Purged regularly.
##### Restructured APIs
- **get(key)** - Return value, *version vector* 
	- List of conflicting values with their versions will be returned.
- **put(key, value)** - Pass (key, value, context). 
	- Conflicting values exist for this object, update will be processed if the context has a reconciled version vector.
#### Configurability
- You can change the values of r and w to attain configurations
- r=2,w=2 -> Balanced reads and write speeds.
- r=1,w=3 -> Fast reads
- r=3,w=1 -> Fast writes
### Handling Permanent Failures
- Hints are stored in extra nodes for limited time. After this, the hints are purged.
- If a node is down for a long time, you get missed writes on some nodes -> Inconsistency.
- **Read Repair** - If on read, data is still old, you update the data
- **Anti Entropy** - Background process which checks for old data. This helps for data that is not read frequently.
#### Merkle Trees
- Binary tree
- Each leaf node stores the hash of a value. Each parent node, stores the hash of its childeren.
- If Hash at root of two trees are same, the entire tree is the same.
- If different, you go through the left and right then find the exact node that is different and update.
- Anti Entropy is run in this fashion. Instead of O(n), O(log(n)) operations are done per value.
### Failure Detection
- **Gossip Protocol** - unordered, eventually consistent and async way to share info.
- Each node when setup chooses 2 other nodes as its buddies. Eg - A:[C,E] D:[E,B]
- Now whenever a node processes any change, it informs its buddies of it. Then the buddies can sync up.
- **Information Shared** - 
	- Node Status(alive/dead)
	- Cluster Membership
	- Load information
	- Metadata(tokens, partition etc)
- **Scalable** - Takes O(log(n)) requests to spread info.
- Heartbeat Mechanism requires a leader to monitor -> SPOF. Here there is no leader requirement.
## CDN
- Global applications face the following issues - 
	- **High Latency** - Due to the distance to the datacenter.
	- **Data-intensive Application** - Large amounts of data to be transferred bogs down the network.
	- **Scarcity of Datacenter resources**
- **CDN** - Group of servers spread geographically to improve the speed and reliability of online content delivery.
- These are proxy servers place on the network edge(closest to the user).
- **Geographically Distributed Cache** - Stores frequently accessed data. Data has a TTL.
- Mostly static data is stored but some dynamic data(API response) may also be stored.
- **Push CDN** - 
	- Content gets sent by origin servers automatically.
	- Better for static content delivery -> origin server knows when to serve new content.
	- Static content is more cachable so Push CDNs have more servers.
- **Pull CDN** -
	- Content is requested by the proxy server. Cached for some time and then purged.
	- Better for dynamic content -> keeps changing per user -> Request based on user.
	- Dynamic content is changing so less no of servers are required + low storage demand.
### Functional Requirements
- **Retrieve** - CDN should be able to make requests to the origin servers for content.
- **Request** - Users can request data from the CDN
- **Deliver** - For a push model CDN, server should be able to push data to the users.
- **Search** - CDN should be able to search for data related to user from the cached data.
- **Update** - On run of script, CDN should be able to update the content through a peer server.
- **Delete** - Delete of cached entries should also be possible.
### Non Functional Requirements
- **Performance** - Should have minimum latency
- **Scalability** - Should scale globally.
- **Availability** - Should be highly available to server users all the time. -> Fault tolerant
- **Reliability** - CDN should not be the SPOF and should protect the data stored.
### Design
![[Screenshot 2026-03-19 at 12.22.36 PM.png|500]]
#### CDN Components
- **Client** - End users of CDN
- **Routing System** - Directs client to the nearest CDN server. 
	- Receives info - Where content is places, #(requests) for a content, URI for content etc.
- **Scrubber Servers** - Filters out the good traffic from malicious traffic.
- **Proxy Servers** - Serves content to the user. Provide accounting data, receive data from distribution system.
- **Distribution System** - Distributes content to proxy servers from origin servers.
- **Origin Servers** - Store the actual data.
- **Management System** - Used to track metrics related to the proxy servers.
#### Workflow
- Origin servers provide the URIs(www.goo.com/api/) for CDN cached object to the Routing System.
- Origin server publishes the content to the Distribution System.
- Distribution System also provides feedback to the routing system to help it identify which server is closest to the user.
- Client request the routing system for content. Routing system returns the IP of the nearest proxy with content
- Request goes through the scrubber server which then forwards to the proxy.
- Proxy server returns content. Also sends info to the Management System.
#### API Design
- **retrieveContent(proxyId, contentType, contentVersion, description)**
	- Exposed by origin server -> Called by proxy server.
- **requestContent(contentType, contentVersion)**
	- Exposed by proxy server -> Called by client.
- **deliverContent(originId, contentType, contentVersion)**
	- Exposed by distribution system -> called by origin server.
- **search(proxyId, contentType, contentVersion)**
	- Exposed by proxy server -> called by other proxy servers to search for data.
- **update(proxyId, contentType, contentVersion)**
	- Exposed by proxy server -> called by other proxy servers to update the content.
#### Dynamic Content Caching
- **Edge Computation** - Some content which depends on data that a edge server might have(time, location, req headers), can be recomputed on edge servers.
- **Partial Caching** - Cache static parts and fetch only parts of the webpage which have dynamic content.
#### Multi tier CDN Architecture
- CDN is has tree like structure. Origin servers contact the top level DNS servers.
- Leaf nodes are contacted by user. Intermediate(*Cache Layer*) nodes are contacted by leaf.
- Requests have a long-tail distribution -> Some content is very popular then, a range of less popular content.
- Leaf servers store the Popular content and query the cache layer to get the less popular content.
### Finding the nearest proxy server
- Best proxy server -> *Network distance from client*, *Amount of traffic the proxy is handling*
#### DNS Redirection
- DNS returns different IPs based on user's location.
- Client -> Root DNS -> TLD DNS -> Authoritative DNS(Controlled by CDN)
- Now based on TLD DNS's IP, Auth DNS gets an approx. location of user.
- Based on location, latency and server load, the request would be routed to a proxy server.
- DNS replies with a short TTL -> CDN can return different IPs based on server load -> DNS as a load balancer.
#### Analyst
- All the proxy servers have the same IP. BGP protocol broadcasts to nodes -> server can serve an IP.
- Routers decide which is the shortest time to reach the IP -> closest CDN server.
#### Client Multiplexing
- Sends the client a list of servers. Client chooses which server to call. -> A way to load balance by DNS
- Inefficient as client does not have enough info -> More calls to a loaded server.
#### HTTP Redirection
- Server responds with a 300 request to route to the correct server.
### Consistency in CDN
- Consistency in CDN means that the CDN servers should serve the latest content.
- **Periodic Polling** - 
	- Used in pull method. Poll origin servers at regular intervals(TTRefresh) to get latest data.
- **TTL(Time to Live)** -
	- Each data object has a TTL on the CDN, then purged. Next request is fetched from origin server -> new data
- **Leasing** - 
	- Proxy obtains a lease on the content it serves from the origin server.
	- Once leased, the origin server controls the content the CDN serves -> Fresh content based on the origin.
#### Placement of CDN Proxy servers
- **On premises** - Add small data centres near major IXP(network points where multiple networks interchange data)
- **Off premises** - Placing proxies directly in the ISP servers.
## Sequencer
- **Unique IDs**
- **Scalability**- Generate at least one billion unique IDs per day.
- **Availability**- assign ids to concurrent events at nano second level.
- **64-bit numeric ID** - Helps in migration. Any larger would lead to slow migration.
- *Causal Order*
### UUID
- Each server generates random UUID.
- Highly scalable, available.
- **Duplicates**- chance of collisions, **128-bit UUID**- slow migration
### DB auto-increment
- If there are m servers -> Each server starts from (1,2,...m) and then increments by m.
- Highly scalable, available, fits in 64bit
- **Duplicates**- If a server fails -> m reduces -> requires coordination to inform and leads to duplicates
  A- 1,4,7. B- 2,5,8. C- 3,6,9. Now B fails -> m=2 -> A generates 9.
### Range Leasing
- Central distributed service responsible to lease ranges of ID to servers.
- If a server exhausts a range, it leases a new available range from the service.
- Scalable, available, 64-bit and unique IDs.
- **No ordering**
### Timestamps
- Uses milisecond timestamps for ordering events.
- Simple, scalable, and supports multiple servers
- Concurrent events within the same millisecond may receive the same timestamp
#### Twitter Snowflake
- ID is constructed as follows - 
	- **Sign Bit** - Always 0(+ve sign)
	- **Timestamp** - 41bits
	- **Worker Number** - Server Id(10bits) -> 1024 servers supported.
	- **Sequence Number** - 12bits(4096) unique ids to separate the concurrent requests in a millisecond
- IDs are time-sortable and the generator is highly available.
- **41bit timestamp** -> Ids over in 69yrs. **Time sync** - clock skew etc.
### Lamport Clocks
- Each node maintains a counter. Each msg sent/received is tagged with this counter.
- **Rules for counter update** - 
	- **Internal Event** - increment counter.
	- **Send message** - increment counter and add to msg.
	- **Receive message** - counter = max(msg.counter, node.counter) + 1.
- This ensures if A->B , L(A)<L(B). But if L(A)<L(B) doesn't mean A->B.
- Guarantees causal ordering but **concurrency cannot be detected**.
### Vector Clocks
- Made up of the following-
	- **Sign bit:** A single bit is assigned as a sign bit, and its value will always be zero.   
	- **Vector clock:** This is 53 bits, and the counters of each node.
	- **Worker number:** This is 10 bits. It gives us 210=1,024210=1,024 worker IDs.
- Vector of size n -> counter of each node updated when a request hits server.
- Captures full causality with concurrency detection.
- **Not very Scalable**- the storage and bandwidth requirements become prohibitive.
### True Time API
- Returns a time interval(time, deviation). Guarantees the time is within the deviation.
- If A_earliest​ < A_latest​< B_earliest​ < B_latest​, then B definitely happened after A.
- Structure - 
	- **Time stamp:** 41 bits, uses the earliest time (TE​TE​​) of the interval.
	- **Uncertainty:** 4 bits, stores the interval width (epsilon).
	- **Worker number:** 10 bits, supports 210210 = 1,024 worker IDs.
	- **Sequence number:** 8 bits, supports 2828 = 256 combinations, resetting to zero when the limit is reached.
- *Guarantees globally unique IDs, causality, 64-bit identifiers*
- **Intervals overlap** - cannot determine order. **Complex**
## Distributed Monitoring

## Distributed Caching
- Cache store data temporarily for faster retrieval.
- Request on server -> Check cache -> if(cache hit) return data; else{ get DB data -> populate cache -> return }
- Advantages of caching - 
	- *Reduced Latency*, *Database Offloading* - load reduced on DB
	- *Session Storage* - User sessions can be effectively managed.
	- *Availability and Scalability* - Designed to be scalable and available.
- **Distributed Cache** - If data set is too large, multiple servers are required to cache the data.
	- Advantages - No SPOF, Reduced Latency(Geo-distributed servers -> faster)
- Eg- Memcached, Redis, Hazelcast
### Writing Policies
- **Write Through** - Cache and DB are written to in the same txn.
	- + High Consistency
	- - High Latency.
- **Write Back** - Data written to cache and then DB is updated asynchronously.
	- + Low Latency
	- - Eventual Consistency, Cache failure leads to data loss.
- **Write Around** - Data is written to DB and cache is updated on a cache miss.
	- + Applications that dont read immediately written data -> Avoid unused item in cache
	- - Cache miss is expensive
### Eviction Policies
- LRU, LFU, MRU, MFU
### Cache Invalidation
- Every item in the cache is stored with a TTL.
- On TTL expiration - 
	- **Active Invalidation** - A service scans the cache for invalidated items and removes them.
	- **Passive Invalidation** - Triggered only when the key is requested. If expired, cache miss. Long unused data would have been evicted by the cache.
==Bloom Filters can be used to check if key exists in the distributed cache==
### Storage Mechanism
- Distributed cache uses clusters(1 leader + 2 followers) to store sharded data.
- Data is updated in the the leader and replicated synchronously.
- Keys are distributed amongst cluster using **consistent hashing**.
- A server in a cluster could be part of application server or a dedicated server. Dedicated Servers allow independent scaling of cache and no contention with application processes.
- **Storage Hardware** - 
	- Specialized hardware can be used which offers better performance but costs more.
	- Bunch of common servers can also be used which are cheaper.
- **Cachhe Server Data Structures** - 
	- **HashMap** - Stores the key and offset where the value is stored.
	- **Doubly Linked List** - Used to implement eviction strategy. We require the following actions -
		- Evict the last element in the list - O(1)
		- Move an element to the front of the list - O(1) if we store the offset of the node with the value.
### Cache Client
- It is a library that communicates with the cache servers.
- It manages hash functions, list of available servers and their IPs.
- **Maintaining the list of servers** - 
	- **Local Configuration** 
		- Each host server maintains a config file with list of healthy cache servers.
		- Server added/removed -> Push to update all the host servers.
		- Updates are manual and operationally slow.
	- **Centralized Service**
		- Config file is stored on a central server and updated when a server added/removed.
		- Push service is not required but still manual updation.
	- **Configuration Service**
		- A central service tracks cache server health and notifies clients if a server is down.
### Design
- **Functional Requirements** - Insert and Retrieve data
- **Non-functional Requirements** - High performance, Scalability, Avaliability, Consistency and Affordability.
#### Implementation
- Client Request -> Load Balancer -> Host Server.
- Cache Client  -(Consistent Hashing)-> Cache Shard -> if(write){Leader} else {any server}
- Config Service ensures all clients see an upto date view of the cache servers.
- A monitoring service can also be used to record metrics.
- *High performance* - Consitent Hashing locates server in log(N) time. Eviction is O(1). Value Location is O(1).
- *Scalability* - Sharding distributes load, Hot shards can be partitioned.
- *Availiability* - Multi-leader replication. 
	- Cache can be distributed across data centres to guard against data center failure -> Introduces latency in synchronous replication. -> leads to eventual consistency.
- *Consistency* - If async replication, eventual consistency.
- *Affordability* - Utilizes common servers hence cost effective.
### MemCached vs Redis
#### Memcached
- Stores key-value pairs as strings -> Values must be seriaizable.
- **Shared Nothing** - Each node operates on its own. No shared state or resources.
- Above architecture is of memcached -> Optimized for high throughput and fast reads.
- Scales horizontally easily.
#### Redis
- It is a data structure store used as a cache, DB and a message broker.
	- **DS Store** -> Redis is aware of internal DS implementation -> Values can be modified in place 
	  -> No need to retrieve, deserialize, modify, reserialize and store for update.
	- **Database** -> Redis can persist in-memory data to secondary store -> Uses Snapshots and WAL.
	- **Message Broker** -> Has a queue implementation or use pub/sub or use redis streams.
- **Single Threaded** - Works on a event loop -> Retrieval is fast -> Multitreading leads to locking overhead which slows down retrieval.
- **Pipelining** - Client can send requests in batch which reduces RTT significantly. Event Loop -> Sequencial Response.
#### Diff
- **Simplicity**- Memcached is simple but needs to be managed by developers. Redis automates everything.
- **Persistance**- Redis can persist data
- **DataType Support**- Redis can support all data.(Sets, arrays, Maps, Logs etc)
- **Multithreading** - Redis is single threaded. Memcached is multithreaded.
- **Replication** - Redis automates it. Memcached uses third party libraries.
- **Data size** - Redis works well with small size data. Memcached is better for simpler, read-heavy workloads with >100kb data size.
## Distributed Message Queue
- **Message Queue** - intermediate buffer between producers and consumers.
- Producers produce messages to the queue. Consumers consume messages from the queue.
- **Uses of Message Queue** - 
	- **Performace and Scalability**- Async communication -> improves performance and easy to scale.
	- **Decoupling**- Allows services to decouple preventing overload and allowing them to scale independently.
	- **Fault Tolerance**- Persistence, Retries and DLQs allow reliable delivery.
	- **Rate Limiting and Priority**- Queues absorb bursts and can process high priority events on demand.
- Eg- RabbitMQ, Apache Kafka and Amazon SQS.
#### Types of Message Queues
- **Point to Point** - Used to send messages to a single consumer.
	- Tracks acknowledgement from the consumer to ensure at-least once semantics.
	- Suited for Task Queues and worker-based systems that require isolated task processing.
- **Pub/Sub** - Bunch of consumers subscribe to a topic that a producer sends message to.(1-\*)
	- Decouples producers from consumers and reduces the need for polling.
- **Request/Reply** - Supports sync 2 way comms between 2 clients.
#### Message Idempotency
- Idempotency means processing a message several times yields the same result as processing it once.
- This helps prevent duplicate actions and data corruption.
#### Error Handling
- For transient errors, the message on a consumer.
- If the #retries > threshold, the message is moved to **Dead Letter Queue** and client decides what to do with it.
#### Single Server Queue
- A simple queue acts as a buffer between producer and consumers.
- Producers and consumers acquire a lock to update/read on the queue.
- Low Throughput(Locks), not Scalable, SPOF -> Not Available.
#### Ordering in Message Queues
- **Best Effort Ordering** - Messages are ordered in the order they arrive at the queue.
- **Strict Ordering** - Messages are ordered in the exact sequence they were produced.
	- *Monotonically Increasing Numbers* - Server assigns numbers. But still network delays break order.
	- *Causality based Ordering* - Messages are sorted by timestamps created by clients.
		- Timestamps cannot be synced due to clock skew -> Messages cannot be ordered across clients.
	- *Timestamps generated by server* - Unique sequencial timestamps are assigned to each request using sequencer
		- Allows server to identify and wait for delayed messages.
#### Sorting, Ordering and Tradeoffs
- Online sorting algorithm to maintain order as new messages arrive -> Order will be lost(Earlier msg processed before a later msg arrives).
- Strict Ordering requries system to perform online sorting or wait for delayed messages.
- We can also use time window ordering where order in a window is ensured.
- There is a trade off between strict ordering guarantee and throughput.
#### Concurrency
- Multiple messages arrive simultaneously or Multiple consumers try to consume messages simultaneously.
- **Locking Mechanism** - A process acquires a lock to place or consume messages -> Not Scalable.
- **Serialization** - System queues the requests in a buffer and a single threaded program processes them serially.
  Avoids race condition and gives higher throughput.
### Design
#### Components
##### Load Balancer
- Distributes incoming requests from producers and consumers to frontend servers.
##### Frontend Service
- Performs the following functions -
	- *Request Validation, Authentication*(Requester is valid, authenticated and authorized).
	- *Request Deduplication* - Prevents identical requests.
	- Caches the metadata service results here as well.
	- Fetches the required queue metadata from metadata service and queries the appropriate backend server.
##### Metadata Service
- Manages queue metadata in the DB and the cache. Updates when queue is added/deleted.
- Stores the leader/cluster id that houses the queue.
- *Caching Strategies* - 
	- **Small Metadata** - Fits in the one machine -> Replicate it across cache servers.
	  Simple. Fast reads but slow writes.
	- **Large Metadata** - Shard the cache based on a key. 
		- *Frontend-Mapping* - Frontend has (server-key) mapping. -> Efficient routing, no extra hop. Complex frontend servers.
		- *Host based Mapping* - Frontend forwards to any cache node -> routes to appropriate node.
		  Simple Frontend, Easy to manage mappings. Extra network hop and higher latency.
##### Backend Service
- Handles message storage, routing and replication.
- Each queue is managed by a cluster. A internal cluster manager maps queues to primary and secondary hosts.
- **Primary Secondary Model** - 
	- There is a single primary and a bunch of secondary nodes.
	- Metadata service stores the id of the primary of a cluster for writes. For reads any can serve.
- **Leaderless Model** - 
	- A cluster is responsible for handling a queue. Within the cluster any node can accept the request.
	- Introduces a external cluster manager which maps requests to different nodes.
- **Message Deletion** - 
	- **No message deletion** - Read messages are not deleted immediately to preserve the order.
		- Once expiration condition is met, the message is automatically deleted by a background job.
	- **Visibility Timeout** - A timeout is added to make the message invisible. The consumer is explicitly responsible for deleting the message else message becomes visible again and processed by another consumer.
## Pub/Sub
- Asynchronous communication method where you send messages to multiple subsystems simultaneously. 
- Services subscribed to a specific `topic` receive any message pushed to that topic.
- **Uses** - 
	- **Improved Performance** - Push based system reduces need for polling
	- **Handling ingestion and replication** - Used for log reading and ingestion -> replication.
#### Requirements
- **Functional Requirements** - Create Topic, Write Message, Read Message,Subscribe/Unsubscribe, Retention policy(TTL)/Deletion.
- **Non Functional Requirements** - Scalability, Availability, Durability, Fault Tolerance and Concurrency.
#### API Design
- create(topic_ID, topic_name)
- write(topic_ID, message)
- read(topic_ID)
- subscribe(topic_ID)
- unsubscribe(topic_ID)
- delete_topic(topic_ID)
#### First Design - Message Queues
- **Topic Queue** - Have a queue for each topic where the producers produce messages.
- **DB** - Store subscription details - which client should produce to which topic. Which client should recieve.
- **Message Director** - Comsumes message from topic queue and forwards messages to consumer queues.
- **Consumer Queues** - Dedicated queue for each consumer.
- **Subscriber** - Handles Subscription request.
- ==Maintaining so many queues is hard==
#### Second Design
##### Broker
- Responsible for storing messages and handling reads/writes.
- Each broker handles a partition of a topic(logical stream of messages).
- Segments(Message + metadata) are stored onto a append only log by each broker with a unique offset per broker.
##### Cluster Manager
- Tracks health of brokers. Handles failures and manages replication of brokers.
- Each partition is usually supported by a leader and there are followers where replication takes place.
##### Storage
- Manages consumer info, Subscriptions and Retention configs
##### Consumer Manager
- Manages the **consumer offsets tracking**(what was the last offset consumer consumed).
- Manages how consumer delivery options -> Push and Pull both supported.
- Authorization of the consumer.
- Enforces **Retention Time policies**
## Rate Limiter
- limits the number of requests to service within a specific timeframe. Throttles traffic > limit.
- Uses
	- *Prevents Resource Starvation*
	- *Prevents excessive costs* due to runaway processes exhausting external resources.
	- *Prevents overloading services* by controlling overflow.
- Types of Throttling
	- *Hard Throttling* - Enforces strict limits. Any request exceeding this is rejected.
	- *Soft Throttling* - Allows a buffer -> Can exceed limits by some %.
	- *Elastic/Dynamic Throttling* - Requests can exceed limits if system has resources.
- Where to place rate limiter - 
	- **On the client side** Easy to place. Hard to configure.
	- **On the server side**
	- **As middleware** intermediary service, throttling requests before they reach API servers.
- **Functional Requirements** - 
	- Limit no of requests in a timeframe.
	- Make this limit configurable.
	- Notify the client when the limit is breached.
- **Non Functional Requirements** - Availability, Scalability and Performance
### Design
- **Rules DB** - Stores rules defined by service owner.
- **Rules Retriever** - Checks rules DB for changes and updates the cache if changed.
- **Rules Cache** - Stores the limits for quick access. Checks requestID against cache.
- **Decision Maker** - Retrieves values from cache and checks against the rules.
- **ID Builder** - Assigns ID to each request to track number of requests for the Decision Maker.
#### Flow
- Request -> ID Builder extracts the ID -> Decision Maker (<-> Cache, get count and Rule) -> pass/fail
#### Race Condition
- Concurrent processing System with "read-modify-write" -> Race condition as 2 requests read counter.
- Using locks create a performance bottleneck.
- Use atomic increment operations to update counters.(CAS)
#### Optimizing counter updates
- **Online Check** - Check the cache and rules when the request arrives. Allow if pass, update cache.
- **Offline Update** - All other places than Cache(Backup log/DB or analytics) are updated async.
- Essentially you use Write-back caching.
### Rate Limiter Algos
#### Token Bucket Algorithm
- Container with predefined capacity(C). Tokens added at a constant rate(R).
- Requests try to consume a token from bucket. If bucket is empty, request is rejected.
- C- Bucket Capacity, R- Rate Limit => Refill Rate = 1/R N- Number of Requests.
- **Advantages** - 
	- Allows traffic bursts as long as tokens are available.
	- Memory Efficient
- **Disadvantages** - 
	- Difficult to configure C and R.
	- *Can Surpass limit at edge* -> Suppose C-3, R-3/min and bucket is full, 3 requests arrive and deplete the bucket. Now at t=0.33 another token arrives and request consumes it. -> 0 to 0.33s -> 4 requests alowed.
#### Leaking Bucket Algorithm
- Uses a queue to manage requests. Smooths traffic by processing requests at a steady rate.
- Requests -> Queue. If queue if full, request is discarded.
- **Bucket capacity(C)** - The maximum queue size. Requests are discarded if the queue reaches capacity CC.   
- **Inflow rate(Rin)** - The rate at which requests arrive.
- **Outflow rate(Rout​)** - The constant rate at which requests are processed.
- **Advantages**
	- The constant outflow rate Rout prevents traffic bursts.
	- Memory efficient
	- Suitable for applications requiring a stable outflow rate.
- **Disadvantages**
	- Bursts can fill the bucket quickly, potentially delaying recent requests.
	- Determining the optimal bucket size and outflow rate is challenging.
#### Fixed Window Counter Algorithm
- Count of requests within a fixed time frame < limit. If limit exceeds, requests are discarded.
- Potential for bursts at edges -> At edge, R before edge and R after edge. -> 2R within a short span.
- **Advantages** -
	- Memory-efficient due to constraints on request rate.    
	- Ensures new requests are processed as long as the window limit isn’t reached.
- **Disadvantages** -
	- Traffic bursts at window edges can temporarily exceed the rate limit.
#### Sliding Window Log Algorithm
- Tracks timestamp of every request in a **set**. 
- Request -> #requests within time frame of its timestamp < limit. Else discard
- Log Size(L - max requests in a timeframe), Arrival time and Time Range(duration of window)
- **Advantages** - 
	- Accurate rate limiting without boundary issues.
- **Disadvantages** - 
	- High memory consumption because it stores timestamps for every request, even those rejected.
#### Sliding window counter algorithm
- It combines the fixed-window counter and sliding-window log algorithms to smooth traffic flow.
- Rate=Rp​×(time frame−overlap time​)/(time frame) + Rc​.
- Rp - #requests in previous timeframe, 
	Rc - #requests in current timeframe
  Overlap time - time passed in current timeframe
- **Advantages** - 
	- Memory efficient
	- Smooths traffic bursts using a weighted average of the previous window.
- **Disadvantages** - 
	- Assumes requests are evenly distributed, which is an approximation.
## Blob Storage
- Storage solution for large unstructured data eg- audio, video etc.
- Ideal for applications with WORM(Write Once, Read Many) storage requirements.
- Usecases -
	- Serving images and documents directly to browsers.
	- Streaming video and audio.
	- Distributed file storage for multiple users.
	- Bakups and data lakes for cloud analysis.
- **Functional Requirements**
	- Get/Put/Delete Blob
	- List Blobs (- in container/related to user)
	- Create/Delete Container
	- List Containers
- **Non Functional Requirements** - Availability, Scalability, Durability, Throughput and Reliability.
##### Resource Estimation
- **Daily active users (DAU):** 5 million
- **Requests per second (RPS) per server:** 500
- **Average video size:** 50 MB
- **Average thumbnail size:** 20 KB
- **Daily video uploads:** 250,000
- **Daily read requests per user:** 20
- *No of servers* = DAU/RPS = 10K servers
- *Storage needed per day* = No of videos per day * ( storage for video + thumbnail) = 12.51 TB/day
- *Inbound Bandwidth Estimation* = (Storage per day)/24\*60\*60 = 1.16 GBps
- *Outbound Bandwidth Estimation* = (No of requests per server per sec * No of servers * Avg data size) = 462 GBps
##### API Design
- putBlob(containerPath, blobName, data)
  containerPath -> userId and containerId.
- getBlob(blobPath) -> userId, containerId and blobName.
- listBlob(containerPath)
- deleteContainer(containerPath)
- listContainers(userId)
### Components
- **Client**
- **Rate Limiter** - Limits the number of request a client can make.
- **Load Balancer** - Distributes the requests between frontend servers.
- **Frontend Servers** - Based on request, gets appropriate node based on Metadata service.
- **Data Nodes** - Stores the actual blobs.
- **Manager Node** - Manages the data nodes and directs where the writes should go to.
- **Metadata Service** - Stores metadata about the blobs, users and containers.
- **Monitor Service**
- **Administrator**
#### Workflows
##### Write a Blob
- Request -> Rate Limiter -> Load Balancer -> Frontend Server -> Manager Node. 
- Manager node has the list of data nodes and tracks the space available on each node.
- Manager node assigns a uniqueID to the blob, chucks the blob and assigns the chunks to different data nodes.
- Frontend server writes data to the assigned data nodes.
- Data nodes accept the write and replicate it synchronously.
- Manager node updates the metadata service about the chunks and their locations.
- Client receives the blobID.
##### Read a Blob
- Frontend Server requests blob metadata from the Manager node.
- Manager Node verifies the blobID and access of the user then fetches data from the metadata service.
- Metadata Service returns the list of chunks with their replicated nodes.
- Client reads chunks directly -> Blob construction is a bottleneck
##### Delete a blob
- Metadata is updated to mark the blob deleted. Garbage collector later cleans it.
#### Blob Metadata
- Blob Metadata is stored as - UserID - ContainerID - BlobID - ChunkID - \< Replica 1,2,3,4\>
- Blob size is a multiple of chunk size.
#### Partitioning Metadata
- Metadata can grow a lot in size so we partition the metadata DB.
- Partitions should be based on {UserID - ContainerID - BlobID - ChunkID}. 
- If based on BlobId, blobs in a container will be spread across partitions -> List operation is expensive.
#### Blob Indexing
- Users define key-value tags (e.g., upload date, media type, container name) during upload. 
- An indexing engine reads these tags and populates a searchable index.
#### Pagination for Listing
- Users can have 1000s of blobs -> Cannot list all. -> use **Continuation Token**
- If query has more than 5 results -> send **Continuation Token** with response listing the offset.
#### Garbage Collection
- Blobs marked for deletion but deleted later-> Helps fast deletion but space wastage.
- Garbage Collection cleans up the data.
## Distributed Search
- Accept a text from user and returns relevant documents having the text.
- **Functional Requirements** - Search
- **Non Functional Requirements** - Availability, Scalability, Fault Tolerance, Performance and Cost Effectiveness.
#### Calculations
- DAU - 150 millions
- #servers = #(requests per second at peak) / (RPS of server) = 150M / 64000 = 2350 servers
- Total Storage per doc = Storage of document(200) + (#terms in document(1000) * size of term(10B)) = 300KB
- Total Storage per day = #(uploads per day = 6000perday) * Storage per doc = 1.8GB
- Incoming Bandwidth = #(Requests per sec) * querySize = 150M/day * 100B = 1.39Mbps
- Outgoing Bandwidth = #(Requests per sec) * responseSize = 55.56Mbps
#### Types of Indices
- **Forward Index** - Store documents ordered by a unique ID.
	- Storing full document in tables leads to **very large data**.
	- **Expensive Searching** requires going through all documents.
- **Inverted Index** - Indexed by the terms appearing in the document.
	- Stores only unique words and discards common words.
	- Structure - *Term* : *Mapping*
	- Mapping - ( List\<docIds\>, List\<Frequency in doc\>, List\< List\<offset in doc1>,L<2>,L<3>..\> )
	![[Screenshot 2026-04-03 at 7.28.29 PM.png|400]]
	- **Advantages** - 
		- Facilitates full text search
		- Reduces search time by pre calculating occurences.
	- **Disadvantages** - 
		- Storage Overhead
		- Difficult to maintain - crud of docs leads to upheaval in index.
#### Indexing on a Centralized System
- Indexing process converts doc into index and stores index in binary file.
- Query process interprets binary file and finds all the docIds with term.
- **Disadvantages** - SPOF, Server Overload, Huge Index file exceeds RAM.
### Components
- **Crawler** - Crawls content collects text, metadata -> JSON file in a distributed storage.
- **Indexer** - Reads JSON file and constructs index using MapReduce.
- **Distributed Store** - Holds files and the index.
- **Searcher** - Parses query, maps terms to index and returns ranked results.
### Distributed Indexing and Searching
- Partitioning the data
	- **Term Partitioning** - Terms split into subsets handled by cluster.
		- Term Partitioning sends requests to a single cluster -> Load on single cluster
		- Multi-word requires inter node communication about docIds
	- **Document Partitioning** - Documents split into subsets handled by a cluster.
		- Sends request to all the clusters -> Network lag.
		- Lesser inter node communications.
- **Indexing**
	- Crawler collects documents
	- Cluster manager splits documents based on *docSize* and *space on node* based on **Hashing**
	- Mapper is run on the affected nodes -> Reducer is run on all nodes to update indices.
- **Searching**
	- Execute parallel search on each node -> Mappings with query terms
	- Merger merges and ranks documents -> Top k results returned.
- *Colocation* - Documents are stored in the same place as index -> Fast search and retrieval.
- **Benefits**
	- Eliminates SPOF
	- Low latency
	- Scalability -> Just add more nodes.
### Replication
- Each node is replicated across 3 partitions. One of these would be the primary.
- Each partition is placed globally so that searches can be faster. Placement is done as a group not node.
- Document is passed to each replica async and indexing is performed.
### Issues
- **Colocated Indexing and Storage** -
	- Storage/Retrieval and Indexing processes contend for same resources.
	- Prevents independent scaling of storage and indexing processes.
- **Index Recomputation** - 
	- Computing index on each replica is expensive and redundant.
#### Solution
- Compute the index on replica and just forward the binary file to the replicas async just like the docs.
#### Separating Indexing and Storage
- **Indexer** - Group of nodes that construct the index.
- **Distributed Store** - Stores the document
- **Searcher** - Group of nodes that fetch the docIds for a query.
- Indexer produces Index -> Stores binary files into storage for backup -> Searcher downloads the file when a request arrives. They may also cache the index files for frequent terms.
### Indexing Explained
- **Cluster manager:** Assigns partitions to Mappers. Triggers reducers after mappers.
- **Mappers:** Extracts terms from assigned partitions -> Inverted index -> exposed to reducers.
- **Reducers:** Combine mapper output to generate combined index -> stored in Distributed Store -> Partitioned across nodes to store index on each node.
=> Documents are partitioned by documents but index is now partitioned across terms.
## Distributed Logging
- Logs -> event flow in distributed systems. -> identify the root cause and reduce the mean time to repair.
- Uses - 
	- **Troubleshooting** application, node, or network issues.   
	- Adhering to internal security policies and external **compliance** regulations.
	- Detecting and responding to **data breaches**.
	- Analyzing **user actions** to inform features like recommender systems.
- Microservices -> distributed across machines -> Failure in one can cascade -> Logs required for root cause
- **Sampling** - 
	- System generates massive number of events. Logging and analysing all is difficult.
	- Sampling captures a fraction of the events to monitor and log.
- **Structuring Logs** - 
	- Logs should have a defined structure to promote **readability** and **downstream processing**.
- **Severity levels** -
	- **DEBUG**: Detailed information for diagnosing problems.
	- **INFO**: Confirmation that things are working as expected.
	- **WARNING**: Indication of a potential problem or unexpected situation.
	- **ERROR**: A serious issue preventing a specific function from executing.
	- **FATAL/CRITICAL**: A severe error causing the program to crash.
#### Considerations about Logging 
- **Protect PII data**
- **Secure secrets** - eg env variables, keys
- **Avoid Excessive Logging** - Logging is I/O heavy and slows down system.
- **Ensure Security** - Logs reveal flow and internal logic -> logging mechanism against expoloitation
#### Functional Requirements 
- Write Logs
- Search Logs
- Store Logs
- Centralized Logging Visualizer
#### Non Functional Requirements
- Low Latency, Scalability and Availability.
##### API
- write(unique_ID, message_to_be_logged)
- searching(keyword)
### Design
- **Log accumulator:** Collects logs from each node and dumps them into storage. 
- **Storage:** We use *blob storage* to save accumulated logs.
- **Log indexer:** Indexes log files to enable efficient distributed search.
- **Visualizer:** Provides a unified view of all logs.
#### Logging at various levels
- **Server Level** - 
	- Server hosts multiple apps and microservices -> Produce logs
	- UniqueID -> ApplicationID + serviceID + Timestamp -> Used for causality.
	- Service pushes logs to *Log accumulator* async(through queues). -> Stores in storage and pushes to **pub/sub system.**
- **Data center Level** - 
	- **Filterer:** Identifies application -> stores logs in the blob storage for that application.
	- **Error aggregator:** Identifies error messages -> notifies the client immediately.
	- **Alert aggregator:** Identifies alerts -> notifies monitoring tools of fatal errors.
- **expiration checker** component responsible for -
	- Verifying logs for deletion.
	- Moving logs to cold storage.
## Distributed Task Scheduler
- A **task** is a unit of computational work requiring resources.
- Some tasks require resources but are not of immediate importance -> Queue it and schedule later.
- Use Cases - 
	- **Single-OS nodes:** OS schedulers use multi-feedback queues to allocate CPU time to competing processes.
	- **Cloud computing services:** Manage billions of tasks from multiple tenants
	- **Large distributed systems:** Platforms like Facebook or Instagram generate billions of asynchronous requests from user interactions. Distributed schedulers to process tasks.
- **Functional Requirements** - 
	- *Submit Tasks*
	- *Allocate Resources*
	- *Remove Tasks* - when user cancels.
	- *Monitor Task Execution*
	- *Efficient Resource Utilization*
	- *Release Resources*
	- *Show Task Status*
- **Non Functional Requirements** - Scalability, Availability, Durability, Fault Tolerance and Bounded Wait Time.
### Design
- Incoming tasks are placed in a queue - 
	- Resource might be occupied.
	- Decouples client from scheduler.
	- Dependencies can be run before the actual task.
- Information required about a task - 
	- *Resource Requirements* - RAM, Disk, Ports CPU cores etc.
	- *Dependencies* - The tasks that this one depends on.
- Components - 
	- **Client**
	- **Rate Limiter** - Limits the number of requests that can pass through based on subscription.
	- **Task Submitter** 
		- Cluster of nodes that accepts requests coming from rate limiter.
		- *Cluster Manager* - monitors nodes via heartbeats and maintains **Task-node mappings**. If a node fails, reassigns all the tasks to a new node.
		- Assigns unique requestID, stores task details on DB
	- **Unique ID generator** - a sequencer
	- **Database** - 
		- **Relational DB** - Store task details like ID, userId, resource reqs, execution cap etc.
		- **Graph DB** - Stores a DAG about the dependencies of a task.
	- **Queue Manager** - 
		- Fetches batches of tasks from RDB based on priority and pushes to queue.
		- Priority is decided based on *delay tolerance* or *execution cap*.
	- **Distributed Queue** - Holds task while resources are busy. If a task fails, makes it visible again for retry
	- **Resource Manager** - 
		- Pulls in tasks from queue based on resource availability and priority.
		- Monitors execution status and terminates tasks that exceed their resource limits.
	- **Monitoring service:** 
		- Checks the health of resources and the resource manager.
#### Queueing
- Priority-Tiered queue structure where each queue follows FIFO.
- System assigns each task **Delay Tolerance** -Max amount of time each task can wait. Shortest tolerance goes first
	- **Urgent:** Tasks that cannot be delayed.
	- **Delayable:** Tasks that can wait for resources.
	- **Periodic:** Tasks executed on a schedule (e.g., every hour).
- System monitors non-urgent queues and moves tasks that have low tolerance to urgent queue.
#### Execution cap
- Large tasks can block small tasks from executing -> **Execution Cap** = Max time to execute.
- Post execution cap, task is declared failed.
- Clients define the execution cap.
- Long running tasks need to be paused and resumed -> **Checkpointing** - store state regularly to restore.
#### Resource capacity optimization
- Resources have a peak time(80% utilization) while remaining idle rest of the time.
- Schedule non urgent tasks when the system is sitting idle. At peak time, shift resources to serve urgent tasks
#### Idempotency
- If node fails or ack no recieved, task has to be retired => Duplicate transactions may occur.
- Idempotent Task -> Even executed multiple times end result is same.
- Assign a uniqueID to the task. When system sees a duplicate, it discards or overwrites.
#### Security of untrusted tasks
- Security is essential when providing infrastructure-as-a-service. Tasks cannot be trusted - 
- **Authentication and authorization:** Strictly control access to resources.
- **Sandboxing:** Isolate code execution using containers (Docker) or virtual machines.
- **Performance isolation:** Monitor resource utilization and cap or terminate tasks that exhibit atypical behaviour
## Sharded Counters
- 