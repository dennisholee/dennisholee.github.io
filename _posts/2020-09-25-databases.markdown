title:  "Database Options!"
date:   2020-09-26 12:00:00 +0000
categories: jekyll update
---

# Comparison Matrix
|Databases|Description|Relational|NoSQL|NewSQL|Remarks|
|-|-|-|-|-|-|
|Data||Relational|Key-Value, Document, Column, Graph|Relational|
|Schema||Static|Dynamic||
|Scalability||Vertical|Horizontal||Relational DB is strongly consistent hence there may be delay to synchronize the DB nodes. |
|Language||SQL|||
|Joins|||||
|OLTP|||||
|Integrated Caching|||||
|Transaction||ACID|CAP||
|Auto Elasticity|||||
|Failover|||||
|Example||MySQL, Oracle, PostgreSQL, and Microsoft SQL Server.|MongoDB, BigTable, Redis, RavenDB, Cassandra, HBase, Neo4j and CouchDB.|GCP Spanner, ClustrixDB, NuoDB, CockroachDB, Pivotal GemFire XD, Altibase, MemSQL, VoltDB, c-treeACE, Percona TokuDB, Apache Trafodion, TIBCO ActiveSpaces, ActorDB|

## NoSQL Comparison Matrix
|Database|MongoDB|Cassandra|Google Cloud BigTable|
|-|-|-|-|
|ACID Compliant|No|
|Scalability|Horizontal|Masterless design (All nodes are identical)|
|Language||SQL Like|
|Sharding|
|Partitioning|

## High Availability

|DBMS|Model|Fail Over|
|-|-|-|
|MongoDB|Master-Slave||
|Cassandra|Ring Architecture (peer communication)||
|Redis|||



# Theories
## ACID
[https://www.quora.com/What-is-the-relation-between-SQL-NoSQL-the-CAP-theorem-and-ACID](https://www.quora.com/What-is-the-relation-between-SQL-NoSQL-the-CAP-theorem-and-ACID)
> * **Atomicity** - Everything in a transaction must happen successfully or none of the changes are committed. This avoids a transaction that changes multiple pieces of data from failing halfway and only making a few changes.
* **Consistency** - The data will only be committed if it passes all the rules in place in the database (ie: data types, triggers, constraints, etc).
* **Isolation** - Transactions won't affect other transactions by changing data that another operation is counting on; and other users won't see partial results of a transaction in progress (depending on isolation mode).
* **Durability** - Once data is committed, it is durably stored and safe against errors, crashes or any other (software) malfunctions within the database.

## CAP
[https://www.quora.com/What-is-the-relation-between-SQL-NoSQL-the-CAP-theorem-and-ACID](https://www.quora.com/What-is-the-relation-between-SQL-NoSQL-the-CAP-theorem-and-ACID)
* **Consistency** - All the servers in the system will have the same data so users will get the same copy regardless of which server answers their request.
* **Availability** - The system will always respond to a request (even if it's not the latest data or consistent across the system or just a message saying the system isn't working).
* **Partition Tolerance** - The system continues to operate as a whole even if individual servers fail or can't be reached.

# Considerations for relational vs. NoSQL systems 

Src: [https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data)

|Consider a NoSQL datastore when: |	Consider a relational database when:|
|-|-|
|You have high volume workloads that require large scale|Your workload volume is consistent and requires medium to large scale|
|Your workloads don't require ACID guarantees|ACID guarantees are required|
|Your data is dynamic and frequently changes|Your data is predictable and highly structured|
|Data can be expressed without relationships|Data is best expressed relationally|
|You need fast writes and write safety isn't critical|Write safety is a requirement|
|Data retrieval is simple and tends to be flat|You work with complex queries and reports|
|Your data requires a wide geographic distribution|Your users are more centralized|
|Your application will be deployed to commodity hardware, such as with public clouds|Your application will be deployed to large, high-end hardware|

# NoSQL Decision Tree
[https://medium.baqend.com/nosql-databases-a-survey-and-decision-guidance-ea7823a822d](https://medium.baqend.com/nosql-databases-a-survey-and-decision-guidance-ea7823a822d)

<img src="{{site.url}}/assets/nosql_decision_tree.png" />
