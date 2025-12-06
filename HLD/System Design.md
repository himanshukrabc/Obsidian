- Readings
    - [Scalable system design patterns](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)

## Hosting
1. Using webhosts:  
    These store all data of their clients in a shared space.  
    There is no data isolation.  
    They bank on the idea that most users will not require a lot of space. Thus larger projects will lead to resource scarcity.
2. Using VPSs(Virtual Private Servers):  
    VPS are virtual machines running of host servers. This enables isolation of client data.  
    However the VPS server hosts may have access to your data.
## Scaling
1. Vertical Scaling:
    - Add more/better resources to the server in the form of RAMs, processors etc.
    - Problem - Price of resources increase exponentially.
    - Inter Process Communication
2. Horizontal Scaling:
    - Add more servers and implement a load balancer to distribute requests.
    - The clients will have access to the IP of load balancer and the server IPs will be private.
    - Network calls(RPC) required
## Scalability
1. **Clones (Horizontal Scaling):**
- As a user may be served by different servers it is essential to store same information on each database.
- Also, it is essential not to store user specific information like sessions, profile pics etc. in servers as the used server may change.
- Sessions need to be stored in a centralized data store which is accessible to all your application servers.
- It can be an external database or an external persistent cache, like Redis. An external persistent cache will have better performance than an external database.
1. **Scaling in Databases:**  
    As number of customer grows, SQL queries become slower.
    - Path 1 - Stick with SQL :  
        Deploy the master slave architecture and perform replication b/w the DB copies.  
        Eventually will have to move to sharding, denormalization and SQL tuning.
    - Path 2 - Move to NoSQL :  
        Joins will now be a part of the application logic which increases scalability.
2. **Caching :**
    - A cache is a simple key-value store. It is a buffering layer between your application and your data storage.  
        For every request, the cache is checked. **Cache Hit -** if data is found else **Cache Miss.**
    - **Caching Database Queries -**  
        Whenever you do a query to your database, you store the result dataset in cache. A hashed version of your query is the cache key.**Issue ⇒**  
        When one piece of data changes (for example a table cell) you need to delete all cached queries who may include that table cell.
    - **Caching Objects -**  
        Each table can be seen as a class and every query will result in a set of objects. Thus we can store the objects in a cache.  
        This way if a row is altered, only the corresponding object needs to deleted.
3. **Asynchronism :**  
    Used to avoid wait times in an app.
    - Async - 1 :  
        a web app con do time-consuming work in advance and serve the finished work with a low request time.  
        Very often this paradigm is used to turn dynamic content into static content.  
        Pages of a website, are pre-rendered and locally stored as static HTML files on every change.  
        This pre-computing of overall general data can extremely improve websites and web apps and makes them very scalable and performant.
    - Async - 2 :  
        A user starts a very computing intensive task which would take several minutes to finish.  
        So the frontend of your website sends a job onto a job queue and immediately signals back to the user: your job is in work, please continue to the browse the page.
## Performance vs Scalability
- A service is said to be scalable if -  
    1. increase in the resources results in proportionally increased performance.  
    2. adding resource for redundancy does not adversely affect performance.
- If you have a **performance** problem, your system is slow for a single user.
- If you have a **scalability** problem, your system is fast for a single user but slow under heavy load.
- Why is scalability hard?
    1. Applications need to be designed keeping scalability in mind. Many algos may break under high load.
    2. Scaling leads to heterogeneity - some nodes will be capable of faster processing, more data storage. ⇒ due to lowering of costs.
- Carefully inspect along which axis we expect the system to grow, where redundancy is required, and how one should handle heterogeneity in this system, and make sure that architects are aware of which tools they can use for under which conditions, and what the common pitfalls are.
## Latency vs throughput
- **Latency** is the time to perform some action or to produce some result.**Throughput** is the number of such actions or results per unit of time.
- Generally, you should aim for **maximal throughput** with **acceptable latency**.
## Availability vs Consistency - CAP Theorem
The CAP theorem comes into effect when a partition occurs. This is when a node is unable to communicate with other nodes in the network.
In a distributed computer system, you can only support two of the following guarantees:
- **Consistency** - Every read receives the most recent write or an error
- **Availability** - Every request receives a response, without guarantee that it contains the most recent version of the information
- **Partition Tolerance** - The system continues to operate despite arbitrary partitioning due to network failures
### CP - consistency and partition tolerance
- Waiting for a response from the partitioned node might result in a timeout error. CP is a good choice if your business needs require atomic reads and writes.
### AP - availability and partition tolerance
- Responses return the most readily available version of the data available on any node, which might not be the latest. Writes might take some time to propagate when the partition is resolved.
- AP is a good choice if the business needs to allow for eventual consistency or when the system needs to continue working despite external errors.
## Consistency Patterns
1. **Weak Consistency -**  
    After a write a read may not see it. It may lead to stale results and loss of data. Eg- in a video chat due to network error, some video will never be seen.
2. **Eventual Consistency -**  
    The read will eventually see all effects of write.  
    Results of asynchronous processing. It leads to stale results but no loss in data.  
    Eg- social media apps.
3. **Strong consistency -**  
    After a write, reads will see it. Data is replicated synchronously.  
    This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.
## Availability Patterns
### 1. Fail-over
- **Active-passive**
	- With active-passive fail-over, heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.
    - The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby. Only the active server handles traffic.
    - Active-passive failover can also be referred to as master-slave failover.
- **Active-active**
    - In active-active, both servers are managing traffic, spreading the load between them.
    - If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.
    - Active-active failover can also be referred to as master-master failover.
#### Disadvantages
- Fail-over adds more hardware and additional complexity.
- There is a potential for loss of data if the active system fails before any newly written data can be replicated to the passive.
### 2. Replication
This topic is further discussed in the Database section:
- Master-slave replication
- Master-master replication
## Availability in numbers
Availability is often quantified by uptime (or downtime) as a percentage of time the service is available. Availability is generally measured in number of 9s--a service with 99.99% availability is described as having four 9s.
### 99.9% availability - three 9s

| Duration           | Acceptable downtime |
| ------------------ | ------------------- |
| Downtime per year  | 8h 45min 57s        |
| Downtime per month | 43m 49.7s           |
| Downtime per week  | 10m 4.8s            |
| Downtime per day   | 1m 26.4s            |
### 99.99% availability - four 9s

| Duration           | Acceptable downtime |
| ------------------ | ------------------- |
| Downtime per year  | 52min 35.7s         |
| Downtime per month | 4m 23s              |
| Downtime per week  | 1m 5s               |
| Downtime per day   | 8.6s                |
### Availability in parallel vs in sequence
If a service consists of multiple components prone to failure, the service's overall availability depends on whether the components are in sequence or in parallel.
- In sequence, Availability (Total) = Availability (Foo) * Availability (Bar)
- In parallel, Availability (Total) = 1 - (1 - Availability (Foo)) * (1 - Availability (Bar))  
## Domain Name System(DNS)
- Domain Name ⇒ ISP ⇒ Local(Low level) DNS server ⇒ Root DNS server
- Local DNS server acts as a cache stores the domain name IP hash. If cache miss, we go to the root DNS server which contains another cache.
![[Image/Untitled 2.png|Untitled 2.png|500]]
- **NS record (name server)** - Specifies the DNS servers for your domain/subdomain.
- **MX record (mail exchange)** - Specifies the mail servers for accepting messages.
- **A record (address)** - Points a name to an IP address.
- **CNAME (canonical)** - Points a name to another name or `CNAME` (example.com to [www.example.com](http://www.example.com/)) or to an `A` record.
DNS services can route traffic through various methods:
- Weighted round robin - distribution based on serving capacity of servers.
    - Prevent traffic from going to servers under maintenance
    - Balance between varying cluster sizes
    - A/B testing
- [Latency-based](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-latency)
- [Geolocation-based](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-geo)
### Disadvantage(s): DNS
- Accessing a DNS server introduces a slight delay, although mitigated by caching described above.
- DNS server management could be complex and is generally managed by [governments, ISPs, and large companies](http://superuser.com/questions/472695/who-controls-the-dns-servers/472729).
- DNS services have recently come under [DDoS attack](http://dyn.com/blog/dyn-analysis-summary-of-friday-october-21-attack/), preventing users from accessing websites such as Twitter without knowing Twitter's IP address(es).
- [DNS architecture](https://technet.microsoft.com/en-us/library/dd197427\(v=ws.10\).aspx)
- [Wikipedia](https://en.wikipedia.org/wiki/Domain_Name_System)
- [DNS articles](https://support.dnsimple.com/categories/dns/)
## Content Delivery Network
- A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user.
- Generally, static files such as HTML/CSS/JS, photos, and videos are served from CDN. The site's DNS resolution will tell clients which server to contact.
- Serving content from CDNs can significantly improve performance in two ways:
- Users receive content from data centers close to them
- Your servers do not have to serve requests that the CDN fulfills
### Push CDNs
- Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.
- Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.
### Pull CDNs
- Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.
- A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.
- Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.
#### Disadvantage(s): CDN
- CDN costs could be significant depending on traffic, although this should be weighed with additional costs you would incur not using a CDN.
- Content might be stale if it is updated before the TTL expires it ⇒ pull CDN.
- CDNs require changing URLs for static content to point to the CDN.
## Load Balancer
Functions of load balancer -
- Help in horizontal scaling
- Preventing requests from going to unhealthy servers
- Preventing overloading resources
- Helping to eliminate a single point of failure
- **Session persistence** - Issue cookies and route a specific client's requests to same instance if the web apps do not keep track of sessions. Cookies will contaain hashes to identify the server.
- **SSL termination** - Decrypt incoming requests and encrypt server responses ⇒ less stress on servers.
Load balancers can route traffic based on various metrics, including:
- Random
- Least loaded
- Session/cookies
- Round robin or weighted round robin
    - **Load Balancer using Hashes:**
        - The incoming requests have a request Id which are hashed, Then the hash value is moded with the number of servers and the remainder decides the server which will process the request
        - serverId = hash(req_id) % N, N=num of servers
        - If the hash function is uniformly random, each server will recieve X/n requests, thus load factor = 1/n.
        - **Issue -**  
            The requestId coming from user usually remain the same. This helps in storing some info as cache in the servers.
        ![[Untitled.jpeg]]
        - However, when the number of servers change, the following happens. 50% of the requests will be directed to a new server and the cache will be destroyed. This is the case with mod operator is deciding the server.
    In weighted round robin, the server capacity is also taken into account.
- **Consistent Hashing:**
    - We need a hashing technique that takes request from each server equally.
    - In consistent hashing, hash both the req_id and the server_id. Then for a particular request, we move clockwise to find the first server to which we can send the request.
    - Now if we add a new server only one of the server’s request will be hampered.
    - There is another problem, the load is not equally distributed. Also if a server crashes, there will be more load on a single server.
    - So, now we use multiple hashing functions to generate multiple positions for the same server. This helps in more equal distribution and load balancing in case of failure.(ABC are servers)
    
    ![[download.png|500]]
    
    ![[Untitled 1.png|500]]
- Layer 4  
    based on the info at the transport layer to decide how to distribute requests.  
    Generally, this involves the source, destination IP addresses, and ports in the header, but not the contents of the packet.  
    Layer 4 load balancers perform Network Address Translation (NAT). ⇒ in incoming request destination IP changed to server address, outgoing → source IP
- Layer 7  
    based on the application layer ⇒ contents of the header, message, and cookies.  
    Layer 7 load balancers terminate network traffic, reads the message, makes a load-balancing decision, then opens a connection to the selected server.  
    For example, a layer 7 load balancer can direct video traffic to servers that host videos while directing more sensitive user billing traffic to security-hardened servers.
At the cost of flexibility, layer 4 load balancing requires less time and computing resources than Layer 7, although the performance impact can be minimal on modern commodity hardware.
### Disadvantage(s): load balancer
- The load balancer can become a performance bottleneck.
- Increased complexity.
- Multiple load balancers required=>, configuring multiple load balancers further increases complexity.
### Types of load balancing:
- active-passive mode:  
    there are 2 load balancers handling requests simultaneously.  
    There is constant communication b/w them. If one of them stops getting the heartbeat from the other one, it takes over as the sole load balancer.
- active-active mode:  
    there are 2 load balancers where only 1 is handling requests.  
    There is constant communication b/w them. If passive one stops getting the heartbeat from the other one, it takes over as the sole load balancer.
### Disk Striping using RAID(Redundant Array of Independent Disks) -
- How do we manage sessions in multi-server systems ⇒ user having session on one server will be asked to create a new session when switched to a new server.
- We need to store the sessions attached to each server until it expires.
- This database becomes a single point of failure.
- RAID uses multiple disks to provide 2 features -
    1. RAID1 ⇒ It writes in 2 drives, so if 1 drive fails the other one is still active.
    2. RAID0 ⇒ writes the data simultaneously on 2 drives(doubling the writing speed) ⇒ Striping.
- RAID10 ⇒ combination of 0,1 , with 4 drives.  
    RAID5 ⇒ 1 redundancy drive and rest striping.  
    RAID6 ⇒ 2 redundancy drives and rest striping.
Read about HAProxy and Nginx
> [!info] Listeners for your Classic Load Balancer - Elastic Load Balancing  
> Use Elastic Load Balancing to distribute your incoming application traffic across multiple EC2 instances.  
> [https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-listener-config.html](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-listener-config.html)  
## Proxy, Reverse Proxy
Proxy - A dedicated device that routes all the requests of a intranet. It helps in blocking malicious responses, caches the requests, hides internaal IPs of nodes etc.
Reverse Proxy -
- It is a proxy server for all the internal servers. It provides unified interfaces to the public.  
    Requests from clients arrive first to proxy server, are forwarded to an internal server that can fulfill it before the reverse proxy returns the server's response to the client.
Additional benefits include:
- **Increased security** - Hide information about backend servers, blacklist IPs, limit number of connections per client
- **Increased scalability and flexibility** - Clients only see the reverse proxy's IP, allowing you to scale servers or change their configuration
- **SSL termination** - Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
    - Removes the need to install [X.509 certificates](https://en.wikipedia.org/wiki/X.509) on each server
- **Compression** - Compress server responses
- **Caching** - Return the response for cached requests
- **Static content** - Serve static content directly eg - HTML/CSS/JS, Photos, Videos
### Load balancer vs reverse proxy
- Deploying a load balancer is useful when you have multiple servers. Often, load balancers route traffic to a set of servers serving the same function.
- Reverse proxies can be useful even with just one web server or application server, opening up the benefits described in the previous section.
### Disadvantage(s): reverse proxy
- Introducing a reverse proxy results in increased complexity.
- A single reverse proxy is a single point of failure, configuring multiple reverse proxies further increases complexity.
## Application layer
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/yB5SYwm.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/yB5SYwm.png)
Separating out the web layer from the application layer (also known as platform layer) allows you to scale and configure both layers independently. Adding a new API results in adding application servers without necessarily adding additional web servers. The **single responsibility principle** advocates for small and autonomous services that work together. Small teams with small services can plan more aggressively for rapid growth.
Workers in the application layer also help enable asynchronism.
### Microservices
A suite of independently deployable, small, modular services. Each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal called gRPC.
Pinterest, for example, could have the following microservices: user profile, follower, feed, search, photo upload, etc.
### Service Discovery
Basically all services register themselves on this app and the app matches requests from various services.  
Systems such as Consul, Etcd, and Zookeeper can help services find each other by keeping track of registered names, addresses, and ports.   
Health checks help verify service integrity and are often done using an HTTP endpoint.  
Both Consul and Etcd have a built in key-value store that can be useful for storing config values and other shared data.
### Disadvantage(s): application layer
- Adding an application layer with loosely coupled services requires a different approach from an architectural, operations, and process viewpoint.
- Microservices can add complexity in terms of deployments and operations.
### Source(s) and further reading
- [Intro to architecting systems for scale](http://lethain.com/introduction-to-architecting-systems-for-scale)
- [Crack the system design interview](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)
- What are service discovery options.
- [Here's what you need to know about building microservices](https://cloudncode.wordpress.com/2016/07/22/msa-getting-started/)
## Database
**ACID** is a set of properties of relational database transactions.
- **Atomicity** - Each transaction is all or nothing
- **Consistency** - Any transaction will bring the database from one valid state to another
- **Isolation** - Executing transactions concurrently has the same results as if the transactions were executed serially
- **Durability** - Once a transaction has been committed, it will remain so
There are many techniques to scale a relational database: **master-slave replication**, **master-master replication**, **federation**, **sharding**, **denormalization**, and **SQL tuning**.
![[Pasted image 20251206141431.png|500]]

### Replication
### Master-slave replication
The master serves reads and writes, replicating writes to one or more slaves, which serve only reads. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.
#### Disadvantage(s): master-slave replication
- Additional logic is needed to promote a slave to a master.
![[Screenshot 2025-12-06 at 2.21.11 PM.png|500]]
### Master-master replication
Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.
### Disadvantage(s): master-master replication
- a load balancer required.
- Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
- Conflict resolution comes more into play as more write nodes are added.
![[Screenshot 2025-12-06 at 2.22.12 PM.png|500]]
### Disadvantage(s): replication
- There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
- Writes are replicated to the read replicas ⇒ reads are slow.
- The more read slaves, the more you have to replicate, which leads to greater replication lag.
- On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
- Replication adds more hardware and additional complexity.
### Federation
- Federation (or functional partitioning) splits up databases by function.
- For example, instead of a single, monolithic database, you could have three databases: **forums**, **users**, and **products**
- resulting in less read and write traffic to each database and therefore less replication lag.
- Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality.
- With no single central master serializing writes you can write in parallel, increasing throughput.
### Disadvantage(s): federation
- Federation is not effective if your schema requires huge functions or tables.
- You'll need to update your application logic to determine which database to read and write.
- Joining data from two databases is more complex with a [server link](http://stackoverflow.com/questions/5145637/querying-data-by-joining-two-tables-in-two-database-on-different-servers).
- Federation adds more hardware and additional complexity.  
### Source(s) and further reading: federation
- [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=kKjm4ehYiMs)
  
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/U3qV33e.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/U3qV33e.png)
### Sharding
- distributes data across different databases based on a shard key.
- Taking a users database as an example, as the number of users increases, more shards are added.
- results in less read and write traffic, less replication, and more cache hits.
- Index size is also reduced, which generally improves performance with faster queries.
- If one shard goes down, the other shards are still operational,⇒ high availability.
- there is no single central master serializing writes, allowing you to write in parallel with increased throughput.
Common ways to shard a table of users is either through the user's last name initial or the user's geographic location.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/wU8x5Id.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/wU8x5Id.png)
#### Disadvantage(s): sharding
- could result in complex SQL queries.
- Data distribution can become lopsided in a shard. For example, a set of power users on a shard could result in increased load to that shard.
    - Rebalancing adds additional complexity. A sharding function based on [consistent hashing](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html) can reduce the amount of transferred data.
- Joining data from multiple shards is more complex.
- Sharding adds more hardware and additional complexity.
### Denormalization
- Denormalization attempts to improve read performance at the expense of some write performance.  
- Redundant copies of the data are written in multiple tables to avoid expensive joins.  
- Some RDBMS such as PostgreSQL and Oracle support materialized views which handle the work of storing redundant information and keeping redundant copies consistent.
- Once data becomes distributed with techniques such as federation and sharding, managing joins across data centers further increases complexity. Denormalization might circumvent the need for such complex joins.
- In most systems, reads can heavily outnumber writes 100:1 or even 1000:1. A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.
#### Disadvantage(s): denormalization
- Data is duplicated.
- Constraints can help redundant copies of information stay in sync, which increases complexity of the database design.
- A denormalized database under heavy write load might perform worse than its normalized counterpart.
### Source(s) and further reading: denormalization
- [Denormalization](https://en.wikipedia.org/wiki/Denormalization)
### SQL tuning
- SQL tuning is a broad topic and many [books](https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Daps&field-keywords=sql+tuning) have been written as reference.
- It's important to **benchmark** and **profile** to simulate and uncover bottlenecks.
- **Benchmark** - Simulate high-load situations with tools such as [ab](http://httpd.apache.org/docs/2.2/programs/ab.html).
- **Profile** - Enable tools such as the [slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) to help track performance issues.
Benchmarking and profiling might point you to the following optimizations.
### Tighten up the schema
- MySQL dumps to disk in contiguous blocks for fast access.
- Use `CHAR` instead of `VARCHAR` for fixed-length fields.
    - `CHAR` effectively allows for fast, random access, whereas with `VARCHAR`, you must find the end of a string before moving onto the next one.
- Use `TEXT` for large blocks of text such as blog posts. `TEXT` also allows for boolean searches. Using a `TEXT` field results in storing a pointer on disk that is used to locate the text block.
- Use `INT` for larger numbers up to 2^32 or 4 billion.
- Use `DECIMAL` for currency to avoid floating point representation errors.
- Avoid storing large `BLOBS`, store the location of where to get the object instead.
- `VARCHAR(255)` is the largest number of characters that can be counted in an 8 bit number, often maximizing the use of a byte in some RDBMS.
- Set the `NOT NULL` constraint where applicable to [improve search performance](http://stackoverflow.com/questions/1017239/how-do-null-values-affect-performance-in-a-database-search).
### Use good indices
- Columns that you are querying (`SELECT`, `GROUP BY`, `ORDER BY`, `JOIN`) could be faster with indices.
- Indices are usually represented as self-balancing [B-tree](https://en.wikipedia.org/wiki/B-tree) that keeps data sorted and allows searches, sequential access, insertions, and deletions in logarithmic time.
- Placing an index can keep the data in memory, requiring more space.
- Writes could also be slower since the index also needs to be updated.
- When loading large amounts of data, it might be faster to disable indices, load the data, then rebuild the indices.
### Source(s) and further reading: SQL tuning
- How do null values affect performance?  
    ⇒ Null values are not indexed. Thus if a search has to be done involving null a full table search will be made.  
    ⇒ Also when NULL values are filled, it is possible that the row size increases the bounds of the memory block where the table is stored. This requires a pointer to the memory location where the remaining data is stored. This leads to an extra lookup when searching data.
- [Slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)-  
    The slow query log consists of SQL statements that take more than [`long_query_time`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_long_query_time) seconds to execute and require at least [`min_examined_row_limit`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_min_examined_row_limit) rows to be examined. The slow query log can be used to find queries that take a long time to execute and are therefore candidates for optimization.  
    You can use the [**mysqldumpslow**](https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html) command to process a slow query log file and summarize its contents.
### NoSQL
- NoSQL is a collection of data items represented in a **key-value store**, **document store**, **wide column store**, or a **graph database**. 
- Data is denormalized, and joins are generally done in the application code.  
- Most NoSQL stores lack true ACID transactions and favor [eventual consistency](https://github.com/donnemartin/system-design-primer?tab=readme-ov-file#eventual-consistency).
- **BASE** is often used to describe the properties of NoSQL databases. BASE chooses availability over consistency.
	- **Basically available** - the system guarantees availability.
	- **Soft state** - the state of the system may change over time, even without input.
	- **Eventual consistency** - the system will become consistent over a period of time, given that the system doesn't receive input during that period.
#### Key-value store
> Abstraction: hash table
- A key-value store generally allows for O(1) reads and writes and is often backed by memory or SSD.
- They consider the data stored as opaque. ⇒ No idea about what is stored
- Key-value stores can allow for storing of metadata with a value. ⇒ somewhere b/w key-value and document store.
- Mostly application layer handles joins.
#### Document store
- A document store is centered around documents (XML, JSON, binary, etc), where a document stores all information for a given object.
- Document stores provide APIs or a query language to query based on the internal structure of the document itself. 
- Based on the underlying implementation, documents are organized by collections, tags, metadata, or directories.  
    Although documents can be organized or grouped together, documents may have fields that are completely different from each other.
##### Source(s) and further reading: document store
- [CouchDB architecture](https://blog.couchdb.org/2016/08/01/couchdb-2-0-architecture/) ⇒  
    database is divided into a number of equal shards based on its ID (and only its ID).  
    When a document is CRUD the node that processes the HTTP request spawns N processes that run in parallel, to attempt the desired operation at every copy of the document. The coordinating node will wait for N/2+1 responses before merging those responses as the HTTP response.
- [Elasticsearch architecture](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up) ⇒ Good article to read.
    - Inverted index ⇒ corres to each word, we construct a mapping reflecting the ids of documents in which it appears.
    - A simple search with multiple terms is then done by looking up all the terms and their occurrences, and take the intersection (for AND searches) or the union (for OR searches).
    - By using transformations(see doc), we turn the search into prefix matching problem ⇒ O(logn) using tries
#### Wide column store
- A wide column store's basic unit of data is a column (name/value pair). A column can be grouped in column families (analogous to a SQL table). column families ⇒ Super column families.
- You can access each column independently with a row key, and columns with the same row key form a row. Each value contains a timestamp for versioning and for conflict resolution.
- Eg- Google introduced Bigtable ,the open-source HBase often-used in the Hadoop ecosystem, and Cassandra from Facebook.
- Maintain keys in lexicographic order, allowing efficient retrieval of selective key ranges.
- Wide column stores offer high availability and high scalability. They are often used for very large data sets.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/n16iOGk.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/n16iOGk.png)
##### Source(s) and further reading: wide column store
- [Bigtable architecture](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/chang06bigtable.pdf)
- [HBase architecture](https://www.edureka.co/blog/hbase-architecture/)
- [Cassandra architecture](http://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archIntro.html)
#### Graph database
- In a graph database, each node is a record and each arc is a relationship between two nodes. Graph databases are optimized to represent complex relationships with many foreign keys or many-to-many relationships.
- Graphs databases offer high performance for data models with complex relationships, such as a social network. They are relatively new and are not yet widely-used; it might be more difficult to find development tools and resources. Many graphs can only be accessed with [REST APIs](https://github.com/donnemartin/system-design-primer?tab=readme-ov-file#representational-state-transfer-rest).
### Source(s) and further reading: graph
- [Graph database](https://en.wikipedia.org/wiki/Graph_database)
- [Neo4j](https://neo4j.com/)
- [FlockDB](https://blog.twitter.com/2010/introducing-flockdb)
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/fNcl65g.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/fNcl65g.png)
### Source(s) and further reading: NoSQL
- [NoSQL databases a survey and decision guidance](https://medium.com/baqend-blog/nosql-databases-a-survey-and-decision-guidance-ea7823a822d#.wskogqenq)
- [NoSQL patterns](http://horicky.blogspot.com/2009/11/nosql-patterns.html)
### SQL or NoSQL
Reasons for **SQL**:
- Structured data
- Strict schema
- Relational data
- Need for complex joins
- Transactions
- Clear patterns for scaling
- More established: developers, community, code, tools, etc
- Lookups by index are very fast
Reasons for **NoSQL**:
- Semi-structured data
- Dynamic or flexible schema
- Non-relational data
- No need for complex joins
- Store many TB (or PB) of data
- Very data intensive workload
- Very high throughput for IOPS
Sample data well-suited for NoSQL:
- Rapid ingest of clickstream and log data
- Leaderboard or scoring data
- Temporary data, such as a shopping cart
- Frequently accessed ('hot') tables
- Metadata/lookup tables
### Source(s) and further reading: SQL or NoSQL
- [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=kKjm4ehYiMs)
- [SQL vs NoSQL differences](https://www.sitepoint.com/sql-vs-nosql-differences/)
## Caching
- The dispatcher will first lookup if the request has been made before and try to find the previous result to return, in order to save the actual execution.
- Databases often benefit from a uniform distribution of reads and writes across its partitions. Popular items can skew the distribution, causing bottlenecks. Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.
- **Client caching -** Caches can be located on the client side (OS or browser).
- **CDN caching -** CDNs are considered a type of cache.
- **Web server caching -**  
	-  Reverse proxies can serve static and dynamic content directly.  
	- Web servers can also cache requests, returning responses without having to contact application servers.
- **Database caching -**  
    - Your database usually includes some level of caching in a default configuration, optimized for a generic use case. 
	- Tweaking these settings for specific usage patterns can further boost performance.
![[Pasted image 20251206143654.png|500]]
- **Application caching -**
    - In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage.
    - Since the data is held in RAM, it is much faster than typical databases where data is stored on disk.  
        RAM is more limited than disk, so [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms) algorithms such as [least recently used (LRU)](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_\(LRU\)) can help invalidate 'cold' entries and keep 'hot' data in RAM.

Redis has the following additional features:
- Persistence option
- Built-in data structures such as sorted sets and lists
There are multiple levels you can cache that fall into two general categories: **database queries** and **objects**:
- Row level
- Query-level
- Fully-formed serializable objects
- Fully-rendered HTML
### Caching at the database query level
hash the query as a key and store the result to the cache. This approach has expiration issues:
- Hard to delete a cached result with complex queries
- If one piece of data changes such as a table cell, you need to delete all cached queries that might include the changed cell
### Caching at the object level
- Data is visualized as a class in the application layer, rows are objects which is then cached.
- Remove the object from cache if its underlying data has changed
- Allows for asynchronous processing: workers assemble objects by consuming the latest cached object
### Suggestions of what to cache:
- User sessions
- Fully rendered web pages
- Activity streams
- User graph data
### Types of caching based on update principle-
#### Cache-aside
The application is responsible for reading and writing from storage. The cache does not interact with storage directly. The application does the following:
- Look for entry in cache, resulting in a cache miss
- Load entry from the database
- Add entry to cache
- Return entry
```Python
def get_user(self, user_id):
    user = cache.get("user.{0}", user_id)
    if user is None:
        user = db.query("SELECT * FROM users WHERE user_id = {0}", user_id)
        if user is not None:
            key = "user.{0}".format(user_id)
            cache.set(key, json.dumps(user))
    return user
```
![[Pasted image 20251206143751.png|500]]

[Memcached](https://memcached.org/) is generally used in this manner.
Subsequent reads of data added to cache are fast. Cache-aside is also referred to as lazy loading. Only requested data is cached, which avoids filling up the cache with data that isn't requested.
### Disadvantage(s): cache-aside
- Each cache miss results in three trips, which can cause a noticeable delay.
- In case of updates, data could become stale.  
    ⇒ set a time-to-live (TTL) which forces an update of the cache entry  
    ⇒ by using write-through.
- When a node fails, it is replaced by a new, empty node, increasing latency.
### Write-through
The application uses the cache as the main data store, reading and writing data to it, while the cache is responsible for reading and writing to the database:
- Application adds/updates entry in cache
- Cache synchronously writes entry to data store
- Return
Application code:
```Python
set_user(12345, {"foo":"bar"})
```
Cache code:
```Python
def set_user(user_id, values):
    user = db.query("UPDATE Users WHERE id = {0}", user_id, values)
    cache.set(user_id, user)
```
Write-through is a slow overall operation due to the write operation, but subsequent reads of just written data are fast. Users are generally more tolerant of latency when updating data than reading data. Data in the cache is not stale.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/0vBc0hN.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/0vBc0hN.png)
### Disadvantage(s): write through
- Caching cannot be done until the DB is updated.  
    When a new node is created due to failure or scaling, the new node will not cache entries until the entry is updated in the database.  
    Cache-aside in conjunction with write through can mitigate this issue.
- Most data written might never be read, which can be minimized with a TTL.
### Write-behind (write-back)
In write-behind, the application does the following:
- Add/update entry in cache
- Asynchronously write entry to the data store, improving write performance
### Disadvantage(s): write-behind
- There could be data loss if the cache goes down prior to its contents hitting the data store.
- It is more complex to implement write-behind than it is to implement cache-aside or write-through.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/rgSrvjG.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/rgSrvjG.png)

  
### Refresh-ahead
- You can configure the cache to automatically refresh any recently accessed cache entry prior to its expiration.
- Refresh-ahead can result in reduced latency vs read-through if the cache can accurately predict which items are likely to be needed in the future.
#### Disadvantage(s): refresh-ahead
- Not accurately predicting which items are likely to be needed in the future can result in reduced performance than without refresh-ahead.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/kxtjqgE.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/kxtjqgE.png)
#### Disadvantage(s): cache
- Need to maintain consistency between caches and the source of truth such as the database through [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms).
- Cache invalidation is a difficult problem, there is additional complexity associated with when to update the cache.
- Need to make application changes such as adding Redis or memcached.
#### Source(s) and further reading
- [AWS ElastiCache strategies](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Strategies.html)
## Asynchronism
Asynchronous workflows help reduce request times for expensive operations that would otherwise be performed in-line.
They can also help by doing time-consuming work in advance, such as periodic aggregation of data.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/54GYsSx.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/54GYsSx.png)
### Message queues
Message queues receive, hold, and deliver messages. If an operation is too slow to perform inline, you can use a message queue with the following workflow:
- An application publishes a job to the queue, then notifies the user of job status
- A worker picks up the job from the queue, processes it, then signals the job is complete
- The user is not blocked and the job is processed in the background.
[**Redis**](https://redis.io/) is useful as a simple message broker but messages can be lost.
[**RabbitMQ**](https://www.rabbitmq.com/) is popular but requires you to adapt to the 'AMQP' protocol and manage your own nodes.
[**Amazon SQS**](https://aws.amazon.com/sqs/) is hosted but can have high latency and has the possibility of messages being delivered twice.
### Task queues
Tasks queues receive tasks and their related data, runs them, then delivers their results. basically they are message queues only.
### Message Broker
If multiple workers are present it may lead to race condition. Message broker is responsible for distribution of the data between multiple workers(Basically a load balancer).
### Back pressure
- If queues start to grow significantly, the queue size can become larger than memory, resulting in cache misses, disk reads, and even slower performance. 
- [Back pressure](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html) ⇒ limiting the queue size, thereby maintaining a high throughput rate and good response times for jobs already in the queue.
- Once the queue fills up, clients get a server busy or HTTP 503 status code to try again later.
- Clients can retry the request at a later time, perhaps with [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff).
### Disadvantage(s): asynchronism
- Use cases such as inexpensive calculations and real time workflows might be better suited for synchronous operations, as introducing queues can add delays and complexity.
### Source(s) and further reading
- [It's all a numbers game](https://www.youtube.com/watch?v=1KRYH75wgy4)
- [Applying back pressure when overloaded](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html)
- [Little's law](https://en.wikipedia.org/wiki/Little%27s_law) -**Little's law** states that the long-term average number _L_ of customers in a [stationary](https://en.wikipedia.org/wiki/Stationary_process) queue is equal to the long-term average effective arrival rate _λ_ multiplied by the average time _W_ that a customer spends in the system. Expressed algebraically the law is **L= _λ*W_**
## Communication
### Hypertext transfer protocol (HTTP)
- HTTP is a method for encoding and transporting data between a client and a server.
- It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request.
- HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression.
A basic HTTP request consists of a verb (method) and a resource (endpoint). Below are common HTTP verbs:

| Verb   | Description                                               | Idempotent* | Safe | Cacheable                               |
| ------ | --------------------------------------------------------- | ----------- | ---- | --------------------------------------- |
| GET    | Reads a resource                                          | Yes         | Yes  | Yes                                     |
| POST   | Creates a resource or trigger a process that handles data | No          | No   | Yes if response contains freshness info |
| PUT    | Creates or replace a resource                             | Yes         | No   | No                                      |
| PATCH  | Partially updates a resource                              | No          | No   | Yes if response contains freshness info |
| DELETE | Deletes a resource                                        | Yes         | No   | No                                      |
- Can be called many times without different outcomes.
- HTTP is an application layer protocol relying on lower-level protocols such as **TCP** and **UDP**.
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/5KeocQs.jpg)](https://github.com/donnemartin/system-design-primer/raw/master/images/5KeocQs.jpg)
### Transmission control protocol (TCP)
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/JdAsdvG.jpg)](https://github.com/donnemartin/system-design-primer/raw/master/images/JdAsdvG.jpg)
- TCP is a connection-oriented protocol over an IP network ⇒ It is a communication protocol which delivers data packets across IP addresses.
- Connection is established and terminated using a [handshake](https://en.wikipedia.org/wiki/Handshaking).
- All packets sent are guaranteed to reach the destination in the original order and without corruption through:
    - Sequence numbers and [checksum fields](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Checksum_computation) for each packet
    - [Acknowledgement](https://en.wikipedia.org/wiki/Acknowledgement_\(data_networks\)) packets and automatic retransmission
- If the sender does not receive a correct response, it will resend the packets. If there are multiple timeouts, the connection is dropped. TCP also implements [flow control](https://en.wikipedia.org/wiki/Flow_control_\(data\)) and [congestion control](https://en.wikipedia.org/wiki/Network_congestion#Congestion_control). These guarantees cause delays and generally result in less efficient transmission than UDP.
- To ensure high throughput, web servers can keep a large number of TCP connections open, resulting in high memory usage. It can be expensive to have a large number of open connections between web server threads and say, a [memcached](https://memcached.org/) server. [Connection pooling](https://en.wikipedia.org/wiki/Connection_pool) can help in addition to switching to UDP where applicable.
- TCP is useful for applications that require high reliability but are less time critical. Some examples include web servers, database info, SMTP, FTP, and SSH.
- Use TCP over UDP when:
	- You need all of the data to arrive intact
	- You want to automatically make a best estimate use of the network throughput
### User datagram protocol (UDP)
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/yzDrJtA.jpg)](https://github.com/donnemartin/system-design-primer/raw/master/images/yzDrJtA.jpg)
[_Source: How to make a multiplayer game_](http://www.wildbunny.co.uk/blog/2012/10/09/how-to-make-a-multi-player-game-part-1/)
- UDP is connectionless. Datagrams (analogous to packets) are guaranteed only at the datagram level. Datagrams might reach their destination out of order or not at all. UDP does not support congestion control. Without the guarantees that TCP support, UDP is generally more efficient.
- UDP can broadcast, sending datagrams to all devices on the subnet. This is useful with [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) because the client has not yet received an IP address, thus preventing a way for TCP to stream without the IP address.
- UDP is less reliable but works well in real time use cases such as VoIP, video chat, streaming, and realtime multiplayer games.
- Use UDP over TCP when:
	- You need the lowest latency
	- Late data is worse than loss of data
	- You want to implement your own error correction
### Source(s) and further reading: TCP and UDP
- [Networking for game programming](http://gafferongames.com/networking-for-game-programmers/udp-vs-tcp/)
- [Key differences between TCP and UDP protocols](http://www.cyberciti.biz/faq/key-differences-between-tcp-and-udp-protocols/)
- [Difference between TCP and UDP](http://stackoverflow.com/questions/5970383/difference-between-tcp-and-udp)
- [Transmission control protocol](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)
- [User datagram protocol](https://en.wikipedia.org/wiki/User_Datagram_Protocol)
- [Scaling memcache at Facebook](http://www.cs.bu.edu/~jappavoo/jappavoo.github.com/451/papers/memcache-fb.pdf)
### Remote procedure call (RPC)
[![](https://github.com/donnemartin/system-design-primer/raw/master/images/iF4Mkb5.png)](https://github.com/donnemartin/system-design-primer/raw/master/images/iF4Mkb5.png)
[_Source: Crack the system design interview_](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)
In an RPC, a client causes a procedure to execute on a different address space, usually a remote server. The procedure is coded as if it were a local procedure call, abstracting away the details of how to communicate with the server from the client program. Remote calls are usually slower and less reliable than local calls so it is helpful to distinguish RPC calls from local calls. Popular RPC frameworks include [Protobuf](https://developers.google.com/protocol-buffers/), [Thrift](https://thrift.apache.org/), and [Avro](https://avro.apache.org/docs/current/).
- RPC is a request-response protocol:
	- **Client program** - Calls the client stub procedure. The parameters are pushed onto the stack like a local procedure call.
	- **Client stub procedure** - Marshals (packs) procedure id and arguments into a request message.
	- **Client communication module** - OS sends the message from the client to the server.
	- **Server communication module** - OS passes the incoming packets to the server stub procedure.
	- **Server stub procedure** - Unmarshalls the results, calls the server procedure matching the procedure id and passes the given arguments.
	- The server response repeats the steps above in reverse order.
Sample RPC calls:

```Plain
GET /someoperation?data=anId

POST /anotheroperation
{
  "data":"anId";
  "anotherdata": "another value"
}
```
- RPC is focused on exposing behaviors. RPCs are often used for performance reasons with internal communications, as you can hand-craft native calls to better fit your use cases.
Choose a native library (aka SDK) when:
- You know your target platform.
- You want to control how your "logic" is accessed.
- You want to control how error control happens off your library.
- Performance and end user experience is your primary concern.
HTTP APIs following **REST** tend to be used more often for public APIs.
#### Disadvantage(s): RPC
- RPC clients become tightly coupled to the service implementation.
- A new API must be defined for every new operation or use case.
- It can be difficult to debug RPC.
- You might not be able to leverage existing technologies out of the box. For example, it might require additional effort to ensure [RPC calls are properly cached](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/) on caching servers such as [Squid](http://www.squid-cache.org/).
### Representational state transfer (REST)
REST is an architectural style enforcing a client/server model where the client acts on a set of resources managed by the server. The server provides a representation of resources and actions that can either manipulate or get a new representation of resources. All communication must be stateless and cacheable.
There are four qualities of a RESTful interface:
- **Identify resources (URI in HTTP)** - use the same URI regardless of any operation.
- **Change with representations (Verbs in HTTP)** - use verbs, headers, and body.
- **Self-descriptive error message (status response in HTTP)** - Use status codes, don't reinvent the wheel.
- [**HATEOAS**](http://restcookbook.com/Basics/hateoas/) **(HTML interface for HTTP)** - your web service should be fully accessible in a browser.
Sample REST calls:
```Plain
GET /someresources/anId

PUT /someresources/anId
{"anotherdata": "another value"}
```
REST is focused on exposing data. It minimizes the coupling between client/server and is often used for public HTTP APIs. REST uses a more generic and uniform method of exposing resources through URIs, [representation through headers](https://github.com/for-GET/know-your-http-well/blob/master/headers.md), and actions through verbs such as GET, POST, PUT, DELETE, and PATCH. Being stateless, REST is great for horizontal scaling and partitioning.
#### Disadvantage(s): REST
- With REST being focused on exposing data, it might not be a good fit if resources are not naturally organized or accessed in a simple hierarchy. For example, returning all updated records from the past hour matching a particular set of events is not easily expressed as a path. With REST, it is likely to be implemented with a combination of URI path, query parameters, and possibly the request body.
- REST typically relies on a few verbs (GET, POST, PUT, DELETE, and PATCH) which sometimes doesn't fit your use case. For example, moving expired documents to the archive folder might not cleanly fit within these verbs.
- Fetching complicated resources with nested hierarchies requires multiple round trips between the client and server to render single views, e.g. fetching content of a blog entry and the comments on that entry. For mobile applications operating in variable network conditions, these multiple roundtrips are highly undesirable.
- Over time, more fields might be added to an API response and older clients will receive all new data fields, even those that they do not need, as a result, it bloats the payload size and leads to larger latencies.
### RPC and REST calls comparison

| Operation                       | RPC                                                                   | REST                                          |
| ------------------------------- | --------------------------------------------------------------------- | --------------------------------------------- |
| Signup                          | **POST** /signup                                                      | **POST** /persons                             |
| Resign                          | **POST** /resign{"personid": "1234"}                                  | **DELETE** /persons/1234                      |
| Read a person                   | **GET** /readPerson?personid=1234                                     | **GET** /persons/1234                         |
| Read a person’s items list      | **GET** /readUsersItemsList?personid=1234                             | **GET** /persons/1234/items                   |
| Add an item to a person’s items | **POST** /addItemToUsersItemsList{"personid": "1234";"itemid": "456"} | **POST** /persons/1234/items{"itemid": "456"} |
| Update an item                  | **POST** /modifyItem{"itemid": "456";"key": "value"}                  | **PUT** /items/456{"key": "value"}            |
| Delete an item                  | **POST** /removeItem{"itemid": "456"}                                 | **DELETE** /items/456                         |
[_Source: Do you really know why you prefer REST over RPC_](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)
### Source(s) and further reading: REST and RPC
- [Do you really know why you prefer REST over RPC](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)
- [When are RPC-ish approaches more appropriate than REST?](http://programmers.stackexchange.com/a/181186)
- [REST vs JSON-RPC](http://stackoverflow.com/questions/15056878/rest-vs-json-rpc)
- [Debunking the myths of RPC and REST](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/)
- [What are the drawbacks of using REST](https://www.quora.com/What-are-the-drawbacks-of-using-RESTful-APIs)
- [Crack the system design interview](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)
- [Thrift](https://code.facebook.com/posts/1468950976659943/)
- [Why REST for internal use and not RPC](http://arstechnica.com/civis/viewtopic.php?t=1190508)
## Security
This section could use some updates. Consider [contributing](https://github.com/donnemartin/system-design-primer?tab=readme-ov-file#contributing)!
Security is a broad topic. Unless you have considerable experience, a security background, or are applying for a position that requires knowledge of security, you probably won't need to know more than the basics:
- Encrypt in transit and at rest.
- Sanitize all user inputs or any input parameters exposed to user to prevent [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) and [SQL injection](https://en.wikipedia.org/wiki/SQL_injection).
- Use parameterized queries to prevent SQL injection.
- Use the principle of [least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege).
#### Source(s) and further reading
- [API security checklist](https://github.com/shieldfy/API-Security-Checklist)
- [Security guide for developers](https://github.com/FallibleInc/security-guide-for-developers)
- [OWASP top ten](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet)

## Back-of-envelope estimates

You'll sometimes be asked to do 'back-of-the-envelope' estimates. For example, you might need to determine how long it will take to generate 100 image thumbnails from disk or how much memory a data structure will take. The **Powers of two table** and **Latency numbers every programmer should know** are handy references.

### Powers of two table

```Plain
Power           Exact Value         Approx Value        Bytes
---------------------------------------------------------------
7                             128
8                             256
10                           1024   1 thousand           1 KB
16                         65,536                       64 KB
20                      1,048,576   1 million            1 MB
30                  1,073,741,824   1 billion            1 GB
32                  4,294,967,296                        4 GB
40              1,099,511,627,776   1 trillion           1 TB
```
###### Source(s) and further reading
- [Powers of two](https://en.wikipedia.org/wiki/Power_of_two)
### Latency numbers every programmer should know
```Plain
Latency Comparison Numbers
--------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy            10,000   ns       10 us
Send 1 KB bytes over 1 Gbps network     10,000   ns       10 us
Read 4 KB randomly from SSD*           150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
HDD seek                            10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from 1 Gbps  10,000,000   ns   10,000 us   10 ms  40x memory, 10X SSD
Read 1 MB sequentially from HDD     30,000,000   ns   30,000 us   30 ms 120x memory, 30X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```
Handy metrics based on numbers above:
- Read sequentially from HDD at 30 MB/s
- Read sequentially from 1 Gbps Ethernet at 100 MB/s
- Read sequentially from SSD at 1 GB/s
- Read sequentially from main memory at 4 GB/s
- 6-7 world-wide round trips per second
- 2,000 round trips per second within a data center
### Latency numbers visualized
[![](https://camo.githubusercontent.com/77f72259e1eb58596b564d1ad823af1853bc60a3/687474703a2f2f692e696d6775722e636f6d2f6b307431652e706e67)](https://camo.githubusercontent.com/77f72259e1eb58596b564d1ad823af1853bc60a3/687474703a2f2f692e696d6775722e636f6d2f6b307431652e706e67)
### Source(s) and further reading
- [Latency numbers every programmer should know - 1](https://gist.github.com/jboner/2841832)
- [Latency numbers every programmer should know - 2](https://gist.github.com/hellerbarde/2843375)
- [Designs, lessons, and advice from building large distributed systems](http://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf)
- [Software Engineering Advice from Building Large-Scale Distributed Systems](https://static.googleusercontent.com/media/research.google.com/en//people/jeff/stanford-295-talk.pdf)