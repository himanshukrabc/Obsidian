Database - Place where data ccan be stored in a way that is easily accessible manageable and updatable.
DBMS - collection of interrelated data and a set of programs to access them.
### DBMS vs File System:
- Data Redundacy
- Accessibility is difficult
- Data Isolation
- Integrity Issue ⇒ Same data may be stored in 2 places do not match.
- Atomicty ⇒ **the property of a database transaction where all the actions within the transaction are executed as a single, indivisible unit of work**

### 3 Schema Layers in DBMS
- Physical/Internal Layer ⇒ Describes how data must be stored(DB) and their retrival methos(Algo).
- Logical/Conceptual Layer(DB Schema) ⇒ What data is stored and the relation between them,(Table def.)
- View/External Layer ⇒ Define views on the tables to provide different shape of data to different users
### DB Schema
1. Attributes and tables  
2. Consistency constraints  
3. Reln b/w tables

- DB Schema can be visualized using **Data Models.**> 
  Data Models Examples ⇒ ER Model, Relational Model, Object oriented model, object relational model.
- Apps communicate with DB through interfaces like JDBC(Java) and ODBC(C/C++)  
### DBMS Application Architecture
- Tier 1 Architecture  
    DB, Client and Server all running in the same PC.
- Tier 2 Architecture
![[Image/Untitled 15.png|Untitled 15.png|250]]
- Tier 3 Architecture
	![[Image/Untitled 1 3.png|Untitled 1 3.png|250]]

**Adv** **⇒**  
	Scalability : DUE TO DISTRIBUTED APP SERVERS  
	Data Integrity : App servers act as filter to prevent data corruption  
	Security :  

### ER Model
- Entity Relational Model ⇒ high level data model based on collection of objects and relations between them.
- Entity ⇒ object which has physical existence and has attributes
    - Strong Entity ⇒ Uniquely Identifiable
    - Weak Entity ⇒ Depends on another entity for identification.
- Attributes
    - Simple
    - Composite - can be divided into subparts(Address)
    - Single Valued
    - Multi Valued
    - Derived
- Relationships ⇒ unary, binary, ternary
- Constraints
    - Mapping Constraint : 1:1, N:1, 1:N, M:N
    - Participation Constraint
        - Partial Participation
        - Total Participation
    
    ![[1.jpeg|800]]
    
    ![[12.pdf]]
    
### Specialization vs Generalization

**Generalization** ⇒ Suppose we have two entities ⇒ Employee, Customer. Here we will have a lot of common attributes. What we can do here is add another entitiy called person so that we dont have to store the same data multiple times.

**Specialization** ⇒ Suppose we have an entity called accounts with both savings interest rate and current a/c transaction charges. This can be specialized into 2 entities.⇒ Savings, Current

|                                                                                                                                                           |                                                                                                                                                            |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GENERALIZATION**                                                                                                                                        | **SPECIALIZATION**                                                                                                                                         |
| Generalization works in Bottom-Up approach.                                                                                                               | Specialization works in top-down approach.                                                                                                                 |
| In Generalization, size of schema gets reduced.                                                                                                           | In Specialization, size of schema gets increased.                                                                                                          |
| Generalization is normally applied to group of entities.                                                                                                  | We can apply Specialization to a single entity.                                                                                                            |
| Generalization can be defined as a process of creating groupings from various entity sets                                                                 | Specialization can be defined as process of creating subgrouping within an entity set                                                                      |
| In Generalization process, what actually happens is that it takes the union of two or more lower-level entity sets to produce a higher-level entity sets. | Specialization is reverse of Generalization. Specialization is a process of taking a subset of a higher level entity set to form a lower-level entity set. |
| Generalization process starts with the number of entity sets and it creates high-level entity with the help of some common features.                      | Specialization process starts from a single entity set and it creates a different entity set by using some different features.                             |
| In Generalization, the difference and similarities between lower entities are ignored to form a higher entity.                                            | In Specialization, a higher entity is split to form lower entities.                                                                                        |
| There is no inheritance in Generalization.                                                                                                                | There is inheritance in Specialization.                                                                                                                    |

### Aggregation
Suppose there is an entity manager that manager employees, branch, jobs. Now if these entities are not further divided, they can be clubbed to form a single entity.

### Relational Model
- Data is represented in the form of tables(called relation).  
    Degree of table,relation ⇒ num of cols  
    Cardinality of table,relation ⇒ num of rows
- Prepared from the ER Model.
- Keys in Relational Model :
    - Super Key ⇒ Any permutation of attributes that can uniquely identify the entry.
    - Candidate Key ⇒ Min subset of super keys which uniquely identify an entry but must not include any redundant atttribute. Eg Name is redundant as two entries may have same name.
    - Primary Key ⇒ a selected key used to identify the entry
    - Alternate key ⇒ Candidate Keys - primary Key
    - Foreign Key ⇒ used to define reln b/w tables.
    - Composite Key ⇒ PK with 2 attr
    - Compound Key ⇒ PK with >2 attr
    - Surrogate Key ⇒ When merging tables with diff PKs, we need a new key to merge them.

- Integrity Constraints  
    Constraints on attributes to ensure correctness.
    - Domain Constraint ⇒ Restricts datatype and value of attr.
    - Entity Constraint ⇒ Every table must have PK.
    - Referential Constraint ⇒ Value specified in FK can either be NULL or provided in the referenced table.Also when deleting a row in the referenced table ⇒ CASCADE(delete all referencing tuples) or NULL(set fk vals to NULL)
    - Key Constraints ⇒ NOT NULL, UNIQUE, DEFAULT() , CHECK, PRIMARY KEY, FOREIGN KEY
### ER Model to Relational Model
Transformation from ER Madal to Relational Model:
- Strong Entity
    - Becomes individual table.
    - FK added to establish reln b/w other tables
- Weak Entity
    - Form a table with FK of the Parent Entity.
    - PK of this table = composite Key {FK, partial discriminator key}
- Single Valued Attr ⇒ becomes col.
- Composite Attr- Each of the components become news column.
- Multivalued Attr-
    - New table created, with name = attr name. FK alt value }.
    - Columns = { PK of entity as FK, sttr value }
    - There may be multiple entries of same PK(of parent) to specify multiple values.
    - PK = { FK, att value }
- Derived Attr - Not considered.
- Generalization-
    - Method-1-
        - Create table for both higher-level lower-level entities.
        - PK for all tables: PK of higher level entity.
    - Methad-2 -
	    - If generalization is disjoint & complete ⇒ higher level entity is not a member of any other lower level entity. then, we can remove the higher level entity
	    - Draw backs-
	        - If there is an entity which is part of both lower lewel entities) ⇒ Redundancy
	        - If an entity is part of neither of the lower level ⇒ no way of storage.
- Aggregation - Create a new table with all the entities in the aggregated block.  
    PK = { Parent Entity, col : col is attr in the aggr }
- Unary Relation - Add foreing Keys

[[SQL]]

### Functional Dependency (A→B):
- If we can get the value B from a set of values A, it means B is functionally dependent on A.  
    Eg- If you have Id, you can get name from employee table. =⇒ Id → Name(Name depends on Id)
- Types :
    - **Trivial :** A→B, B is subset of A.  
        Eg- {Id,Name} → Name.
    - **Non Trivial :** A→B, B is not subset of A.  
        **Eg-** {Id,Name} → Address (through Table Query)
- Properties :
    - Reflexive - If B is subset of A ⇒ A→**B**
    - Augmentation - If A→B , X={ A ,Y,Z….} ⇒ X→B
    - Transitivity - A→B, B→C ⇒ A→C

### Normalization
- To avoid redundancy in Data stored.
- Anomalies due to redundancy
- Insertion ⇒ certain data cannot be inserted without presence of anouther data  
	Eg- The branch IT cant be inserted without a student enrolled in the dep.
    ![[2.png]]
- Deletion ⇒ Deletion of data may lead to loss of other important data  
	Eg- Deletion of id=6 leads to loss of any data on mech dept.
- Updation ⇒ Single updation requires the updation of multiple tuples.  
	Eg - Change of HOD leads to multiple updations.
- In Normalization, we decompose tables into multiple tables until _**Single Responsibility Principle(SRP)**_ is established.
- Types of Normalization :
    - 1NF :  
        Each attribute must have atomic values ⇒ No multivalued attribytes present.
    - 2NF : Requires a 1NF relation(table)  
        There should not be any partial dependencies, i.e. , Any non prime attribute cannot be dependent on a subset of PK.  
        Eg - Relation = R(A,B,C,D) , PK ={A,B}, Partial Dep⇒ B→C, Complete dependency ⇒ AB→D  
        Issue ⇒ As {A,B} is PK, B can be null ⇒ B cannot determine C.  
        Solution ⇒ R1(A,B,D), R2(B,C)
    - 3NF : Requires a 2NF relation(table)  
        No transitivity must exist in non-prime attributes.  
        Eg- Reln(A,B,C) ⇒ A→B, B→C  
        Due to B→C there is lot of redundant data.  
        Soln- R1(A,B) R2(B,C)
        ![[2 1.png]]
    
    - BCNF(Boyce Codd Normal Form) : Requires a 3NF relation(table)  
        Any prime attribute cannot be determined by any attribute ⇒ The primary key must be a super key.  
        Issue ⇒ Subject can be determined using the professor.  
        Solution ⇒ R1(Id,Professor) R2(Professor,Subject).
    ![[2 2.png]]
    
### Transactions in DBMS
A set of sql statements required to do a unit of work in a logical sequence.  
If any step generates an error, the transaction is rolled back.  
Eg- Bank transfer from A to B  
1. Read(A) ⇒ Data read and stored in a buffer  
2. A:=A-50  
3. Write(A) ⇒ Buffer to DB buffer. After commit, value is written to DB.  
4. Read(B)  
5. B:=B+50  
6. Write(B)

### ACID Properties
1. Atomicity  
    Either all the changes corres to a transaction are reflected in the DB or none are.  
    Eg - If failure occours after step 3, The changes due to first 3 steps must not be reflected.  
    Thus at anytime, the DB maintains an old state and a transition state. If failure happens, we revert to the old state.
2. Consistency  
    Data changes due to transactions must be consistent.  
    Eg- If failure occours after step 3, it will lead to loss of money. If the changes are not rolled back.
3. Isolation  
    Multiple transactions must be concurrent and sequential. For transaction T2 to start, all earlier transactions must finish executing. Transsactions are allowed to execute concurrently, each transaction should be independent and isolated of each other.
4. Durability  
    After transaction is completed, the changes made in DB must be persistent even if there is a system failure. This can be done by generating logs or by directltly doing steps on main DB.

### BASE Properties
- The acronym BASE stands for Basically Available, Soft State, and Eventual Consistency.
- **Basically Available**
    - DB system should always be available even if consistency is compromised. Minimize downtime and provide quick recovery from failures.
- **Soft State**
    - The state of the DB can change over time, even without any explicit user intervention. This can happen due to the effects of background processes because of eventual consistency.
- **Eventual Consistency**
    - The database should eventually converge to a consistent state, even if it takes some time for all updates to propagate and be reflected in the data.
- Uses of BASE Databases
    - BASE databases are used in modern, highly-available, and scalable systems that handle large amounts of data. Examples of such systems include online shopping websites, social media platforms, and cloud-based services.
      
### States/Lifecycle of Transaction
![[Image/Capture 2.png|Capture 2.png]]
1. Active state  
    The very first state of the life cycle of the transaction, all the read and write operations are being performed.
2. Partially committed state  
    After transaction is executed the changes are saved in the buffer in the main memory. This is called partially commited state.
3. Committed state  
    When updates are made permanent on the DB. Then the T is said to be in the committed state. Rollback can’t be done from the committed states. New consistent state is achieved at this stage.
4. Failed state  
    When T is being executed and some failure occurs. Due to this it is impossible to continue the execution of the T.
5. Aborted state  
    When T reaches the failed state, all the changes made in the buffer are reversed. After that the T rollback completely. T reaches abort state after rollback. DB’s state prior to the T is achieved.
6. Terminated state  
    A transaction is said to have terminated if has either committed or aborted.

### Implementing Atomicity and Durability
1. Shadow Copy Method
    - A DB pointer is present which points to the old DB storage
    - A new copy of the entire DB is created on RAM, where all transactions are performed.
    - If transaction suceeds, The new copy is written into memory and the pointer is updated which marks the commited state.  
        else the new copy is deleted. ⇒ _**ATOMICITY**_
    - As transaction is only completed after updation of DB pointer, if system crashes before that, it leads to no record of any changes. ⇒ _**DURABILITY**_
    - _**The writing of pointer is made to be a atomic process, by ensuring that the DB copy is written in memory in a individual block of memory. ⇒ ensures ATOMICITY**_
    - Inefficient
2. Log Bassed Recovery System
    - All logs are stored in a stable storage which ensures atomicity.
    - We generate logs for all the steps transactions made. Logs are generated before steps are performed.
        - **Defered DB Modification**
            - Logs for the entire transaction are generated before the actual changes in DB are made. Then the logs are converted to changes in DB.
            - If there is no commit log, the logs are ignored. ⇒ _**ATOMICITY**_
            - If failure occours while writing, we perform redo.
            ![[Image/Untitled 2 3.png|Untitled 2 3.png]]
        - **Immidiate DB Modification**
            - ==Uncommited modifiction :== write operation performed for each step just after its log is generated.
            - The logs generated here have an extra field with the old value written.
                ![[Image/Untitled 3 3.png|Untitled 3 3.png]]
            - In case of system crash(commit log not found), the old values are used to ensure atomicity.
            - In case the system crashes just after commit log is generated(but commit was not performed), we redo the entire transaction rom the logs. In such a case, last log found is the commit log.
> [!important] ==**Checkpoints ⇒**==
    > 
    > As processing multiple transactions is difficult, when we find a consistent state of DB(all transactions before this have been committed), we set this a checkpoint. This indicates that any checking should start after the last checkpoint.
### Indexing
- Data stored in DB is in the form of Blocks. Each block has multiple tuples, which can be ordered by the a key aslo called the search key.
- We create index file containing list of { search key ,base pointers} pair to the blocks which is sorted again. This helps in making the search faster.
![[Image/Untitled 4 2.png|Untitled 4 2.png]]
- Types of Indexing
    - Primary Indexing(Clustering Index) :
        - If the search key is also the key on which the DB file is sorted.
        - Note that it may be possible that the DB file is not sorted by primary key or ay key.
        - Dense Indexing :
            - All different search key values of the records appear in the index file.
            - This is generally used when search key is a non key attr.
            - The records with the same value of the search key are stored sequencially after first record.
            - We store the base pointer corres. to first occourance.
            - \#entries = \#unique attr. values
        - Sparse Indexing :
            - Some of the keys of the records are only present in the index file.
            - If the search key is actually a primary/candidate key, then we generally do sparse indexing.
            - \#entries = \#blocks.
        - Multi level Indexing
            - In case of large DBs, we need to create index files for index files.
                ![[Image/Untitled 5 2.png|Untitled 5 2.png]]
    - Secondary Indexing:
        - Data file is unsorted.(Eg if it is sorted bsed on another attr but we want to seach the other attr.)
        - You need to do dense indexing, store every value of the search key. This helps in doing BinSearch.
- Advantages of Indexing
    1. Faster access and retrieval of data.
    2. IO is less.
- Limitations of Indexing
    1. Additional space to store index table
    2. Indexing Decrease performance in INSERT, DELETE, and UPDATE query.
### NoSQL DBs
- “Not only SQL”
- Stores data in non relational format.
- Data can be non structured, i.e, There is no fixed schema. User can store structured, semi-dtrutured and unstructured data(eg files).
- **Data Modelling in NoSQL ⇒** Data is stored in the form of JSON object
- **Flexible Schema** ⇒ The data stored has no fixed schema. It allows multiple schemed data to be stored in same space.
- **Scaling ⇒** What do we do if the space available for DB is exhausted?
    - Vertical Scaling(Scale Up) ⇒ Improving hardware(CPU, memory, RAM)
    - Horizontal Scaling(Scale Out) ⇒ Distribution of load into multiple nodes. Thus the DB can exist in split state.
    

> [!important] **Why SQL does not support horizontal scaling?**
> 
>   
> During data retrival using joins, we need to retrieve tables from multiple nodes, into a common system via network and then we can use the query. This makes the query very slow.  
> NoSQL doe not require Joins.

> [!important]
> 
> **Why do you need NoSQL?** ⇒ Due to expensive memory, data was stored in structured way to reduce redundancy. But as memory became cheaper, there was need to optimise access time. Due to its non structured format, data can be stored in a format to make it more accessible.

- Advantages
    - Flexible Schema
    - Horizontal Scaling
    - High Availability ⇒ NoSQL DBs auto replicate to previous consistent stae on failure. Even if a server ails, the other nodes are still accessible.
    - Easy insert, read ⇒ Due to no join required and all data available in the same object, the queries of NoSQL DBs can be faster.
- Disadvantages
    - Updation,Deletion is difficult because the entire object needs to be updated.
  - When to use?
    - Fast Development required.(Planning is less)
    - Huge volume of data
    - Scale out requirements.
    - Microservices and streaming requiremens.
    - Storage of unstructured data.
    
> [!important] **==Note :==**
> 
>   
> NoSQL DBs can also store relational data. Relations here are stored as fields referencing id of another entry.  
> NoSQL DBs also support ACUD properties
- Types of NoSQL DBs :
    - Key-value stores: Data stored in { key, value } pairs.
    - Column-Oriented / Columnar / C-Store / Wide-Column:
        - Data is stored column wise.
            ![[Image/Untitled 6 2.png|Untitled 6 2.png]]
	    - In DB, data is stored as a linked list. When reading data stored in rowwise format, we need to jump a few nodes to get next required info. Thus, reading is easy in column stores. But insert is slow as it needs to be done in between the LL.
        - This helps in aggregation and analytics.
    - Document stores : Data is stored as json objects, **Supports ACID properties.**
    - Graph based stores : Data is stored as nodes and the relations are edges.
        - Stores the relations as edges along with nodes.
- Disadvantages:
    - Data Redundancy
    - Updation,Deletion Costly
    - Generally ACID properties are not supported.
    - Doesnot support data entry consistency constraints.
![[Image/Untitled 7 2.png|Untitled 7 2.png]]

### Object Oriented DB
- Every data is stored as an object. Data can be table, code, string etc.
- Objects communicate with each other via methods.
- Handles ⇒ Id of objects.
- This can be visualized as an implementation of ER Diagram.
- All data required is in the object/can be accessed via methods ⇒ Joins not requuired.
- Advantages :
    - Easy retieval
    - Can handle complex relations well.
    - Easy to model, supports OOPs languages.
- Disadvantages :
    - Slow CRUD operations.
    - Does not support views.
### Heirarchical DBs
- Data stored as a tree. Eg- File System.
- Can be used as physical models as disk storage system is also heirarchical.
- Easy to use, traversal is faster.
- Cannot be used when there are relations between siblings. Or when a node has multiple parents
### Network DB
- Heirarchical DB with nodes having multiple parents.
- This gives a graph like structure and allows all relations.
- Slow traversal
### Clustering / Replica Sets
- There are multiple sets of servers storing the same data.(Replicas) ⇒ The collection of these replicas is called Replica Sets or Cluster
- High Availability ⇒ This helps in processing queries when server is down.(Eg- crash, maintainence etc.)
- Load Balancing ⇒ Used to handle multiple requests by splitting requests across servers.
- CDN ⇒ group of servers for fast delivery of data.
### Sharding / Partitioning
- If large amt of data, \#requests ⇒ data can be distributed. ⇒Distributed DB
- To increase performance ⇒ vertical scaling, clustering+load balancing, partitioning
- Partitioning can be done in horizontal(splitting by row) or vertical(splitting of columns) way.
- Advantages
    - Parallelism ⇒ 2 request which can run by using a mutually exclusive list of serverscan be run simultaneously.
    - Availability ⇒ Even if one partitionn crashes, queries not involving this will still work.
    - Performance ⇒ less loading
    - Managability ⇒ Updation of data iis alos easy.
    - Scaling is easy.
### Sharding
- It is a way to implement horizontal partitioning based on an attr called shard key.
- Instead of having all the data sit on one DB instance, we split it up and introduce a routing layer so that we can forward the request to the right instances that actually contain the data.
- Pros
    - Scalability
    - Availability
- Cons
    - Complexity, making partition mapping, Routing layer to be implemented in the system,
    - Non-uniformity that creates the necessity of Re-Sharding
    - Not well suited for Analytical type of queries, as the data is spread across different DB instances. (Scatter-Gather problem)
### Scaling Patterns
- As the number of transactions increase, it is possible for the performance of the DB to reduce.
- **Pattern 1 : Query Optimization and Collection Pool Implementation**
    - Cache frequently used non dynamic(data fixed for user) data
    - Introduce data redundancy(to reduce joins) or use NoSQL
    - **==Cache DB Connections/ Collection Pool Implementation==** ⇒ When DB calls are made, it leads to establishment of new DB connections. It is better to create a pool of connections to reduce latency.
- **Pattern 2 : Scale Up**
- **Pattern 3: Command Query Responsibility Segragation / Master Slave Architecture**
    - Seperate read and write request physical machine wise.
    - There will be a single machine for write request and replicas for read. The replicas constantly replicate the primary to ensure consistency.
    - There is usually a lag in read request to ensure the replication takes place.
        ![[Image/Untitled 8 2.png|Untitled 8 2.png|500]]
 - **Pattern 4: Multi Primary Repliation**
    - More write requests than server can handle ⇒ read requests also slows down due to multiple replication.
    - There is no primary/replica segregation.
    - All replicas arranged in circle and replicate the data of the previous server(if required)
    - Thus any request can be made to any node. Thus the load of write request is distribtued
    - The read request is broadcasted and the node replying first is sent. There is still some lag in the read request.
 - **Pattern 5: Partition of Data by functionality(~50 req/s)**
    - Create seperate DBs for specefic functionalities having all the above functionalities.
    - Application/Backend must handle the aggregation of data from the 2 DBs.
 - **Pattern 6: Horizontal Scaling**
    - Sharding, Replica Sets, etc.
 - **Pattern 7: Data Centre Partitioning**
    - Distribute requests across multiple data centres.
    - There is also data replication to other data centres to ensure availability on failure.
### CAP Theorem for Distributed Storage
- CAP ⇒ Consistency, Availability, Partition Tolerance
- Consistency ⇒ all systems must see the same simultaneously.
- Availability ⇒ In case of crash, there must be a backup to supply data.
- Partition Tolerance ⇒ In case of communication breakdown between primary and replica(Partition), the system must still be up. There can be delays but it should till provide response.
- CAP Theorem states, that in a distributed system, all three properties can not be achieved.  
    ⇒ In case of partition, either availability or consistency can be achieved.
- **CA(P) system :** only possible with a single node.
- **AP system :** If you provide availability(write and read both requests will be processed), due to communication breakdown, consistency is hampered
- **CP system :** If you want the system to be consistent, you need to stop either write or read request and compromise availability. Eg- MongoDB. Used in banking systems.
### Master Slave Architecture
- If we have a single DB serving mltiple servers, it can be a single point of failure.
- Instead, we create replicas and do DB replication from the original DB.
- This is called the master-slave architecture.
- All write operations are made on the master and the replica is used for read requests
- **Replication** ⇒
    - Async ⇒ This leads to read requests giving inconsistent data.
    - Sync ⇒ This leads unavailability of the DB to read requests.
- Advantages :
    - If primary aails, system still supports read requests
    - Scaling out is easy
    - Latency is reduced to paallelism.