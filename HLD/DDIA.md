- Problems with modern software - **Data Intensive** not **Compute Intensive**.
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
- Mismatch between the application objects structure and table data. 
- **ORM(Object Relational Mapping)** used for the translation as it reduces boilerplate code.
#### Used when
- Data has clear structure.
- Relationships matter and are frequently queried.
- Strong consistency and transactions are required.
### Document Store
- Stored as objects(JSON/BSON) or XML.
- **Flexible or schema-on-read** design.
- **Better locality of data** - Similar data stored in the same object so **fewer joins** are required.
- **Limited support for joins** - Application is responsible for maintaining consistency.
- **Writes are harder** - Inplace updates can only happen if encoded document's size does not change. Also, consistency issues.
#### Use when
- Variable schema/ Hetrogenous data.
- **Scalability for read heavy loads** -> *Maintain small documents else slow reads(as small parts of documents are required)*
- Your data is a tree of one to many relationships.
>**Data Locality** - Can do wonders for reads.
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
