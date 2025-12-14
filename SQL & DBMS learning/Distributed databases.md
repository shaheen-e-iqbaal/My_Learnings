

When should we move to distributed databases :

-> when we encounter that, our db server is taking longer than expected time to return result of a query. we should first optimize our query, improve indexing (remove unnecessary indexes and create useful indexes). but when our table sizes grow to the extend of 2 GB or more, then index tree height will also grow significantly. so we can't optimize index more. now, what we can do is, we can partition our large table into smaller segments. now for each of this segment, we can have it's separate index. so this way, we can reduce the index tree size. but now when our tables grow to the extent that, this partition also does not help, then we can replicate our db so that, we can reduce the load on single db node and scale our read across multiple replicas. now even if the replication doesn't help, then we should shard/partition our database across multiple nodes. i.e. make our database distributed.

-> So, distributed database should be the last option in scaling journey. 


High level architecture of Distributed Databases :

1. Shared Nothing
2. Shared Disk
3. Shared Memory

-> When we have single node database ex : we run postgreSQL, MySQL, it is called shared everything architecture. we run database on single node which has it's own disk, memory, CPU. 

Shared Nothing : 
-> In this architecture, each node of database has it's own storage (disk), compute power (CPU, memory). Each node contains some portion of data. to perform single query, we (coordinator) might need to refer to multiple nodes to get data stored in them.
-> This architecture offers great horizontal scalability. but it also gives challenges like managing transaction...
ex : Cassandra, Redis, CockroachDB
-> The other challenge this architecture gives is that, when we want to scale in or out (i.e. add new node or remove existing node), then we have to redistribute the data on  available nodes have balance portion of data on each node. we can use consistent hashing to optimize the redistribution of data but still it poses as challenge. 


Shared Disk :

-> In this architecture, we have one single infinite size disk (ex : AWS S3) and multiple nodes with their own CPU, memory which shares this disk. so disk (storage) is common for all nodes but CPU, memory (RAM) is personalized for each node.

-> the major benefit of this architecture is that, both level (i.e. disk and nodes) can be scaled independent of each other. if we want to add new node or remove existing one, we don't need to redistribute the data unlike we do in Shared Nothing architecture. 
-> as each node is stateless (unlike nodes of shared nothing, where to fetch specific data, we must go to that specific node containing that data), We can balance load across nodes based on individuals present health(i.e. less current load).

-> each node (dbms) must be capable of how to fetch data from shared disk. it is not like that, we run PostgreSQL on multiple nodes and share S3 between them. because PostgreSQL doesn't know how it can get data S3 disk.

-> The major challenge of this architecture is to handle concurrency due to which we might need to resolve conflicts which may occur when multiple users hit request to update same database object and that requests goes to different nodes. similar challenge we face when we have do multi-primary replication of database.

-> One other challenge we face in this approach is, suppose one node n1 has some db pages in it's memory (note that not in disk) due to some old read/write query on those pages. now suppose node n2 receives a write query on same db page. now when n2's transaction commits, those pages in n1 remains same. means that data gets stalled. so how we ensure that pages on n1 memory also get updated? there are multiple ways. 1. upon n2's txn commit, it tell all nodes to update those data. it can be done in synchronous or async way based on our applications data consistency requirements(social media apps prefers async way, banking apps prefer sync way. 

ex : Google Spanner, yugabyteDB, databricks, snowflake, FIREBOLT, Google Big Query, ORACLE RAC,  

-> Google Spanner use Distributed file system (created by google itself) as storage (instead of S3).

-> This architecture is most famous in recent decades due to it's ability of scaling compute and storage independently. 


==***Major Design Decisions we have to make while doing Distributed Databases :***==

-> How does application find data
-> Where does the application send Query
-> How to execute query on distributed data (in shared nothing architecture) 
	-> send query to data Or
	-> send data to query
-> How does databases ensure correctness 
-> How to divide data across available nodes (in shared nothing architecture)


Homogeneous vs Heterogeneous nodes
Homogeneous : in this type of systems, all nodes are capable of doing all work. i.e. if we send query to any node in cluster, it will have capability to find out which data is stored at which node. means it can work as coordinator.

Heterogeneous : in this type, each node in cluster is assigned a specific task. like there is specific node as coordinator which keeps all info about which data is stored in which node....


**==Ways to partition data :==**

1.  Naive table partitioning : in this way, each table is stored on separate node. each node can contain data of at max one table. 

2.  Vertical partitioning : the idea here is that, we do something like column oriented partitioning. suppose, the values of first 3 columns of all tuples is stored on one node, and remaining 3 columns of all tuples is stored on second node. in gujarati, ham table to vertically cheer dete he (it can be single partition, two or more) 

-> it can be useful suppose we have first 3 column as integer, which we query most. and fourth column is TEXT, which can have any size. so we can store first three column on single partition and fourth on other one.

3.  Horizontal partitioning : in this way, we partition our table rows across multiple nodes. we will select one/more column(s) from table, and based on this column(s) (key), we will decide where we should place this tuple. this tuple/key is called partitioning key. 

-> choose a column(s) which divides database equally in terms of size, load...

-> There are multiple scheme using which we can do horizontal partitioning : 1. Hashing 2. Range 3. Predicate
-> in hash based partition, we find the hash of partition key, and based on it's output, we decide where to put that tuple. suppose we got hash = 10 and # of nodes = 4. then 10 % 4 = 2, so this tuple will go to node 2. this scheme is most common one in real world.
-> in range based scheme, we partition rows based on range. like all rows with age between 1 to 10 goes to this node and 11-20 goes to this.... like that. 
-> in predicate based scheme, we define one predicate. and for each row, we pass it to that predicate and based on it's result, we choose node where to put this tuple. 

-> All above thing we discussed is where we are doing physical partitioning. means we are actually partitioning data physically across multiple nodes. this is what we are doing when we have shared nothing architecture. but when we have shared disk architecture, in that case our data is stored at single place. so in that case, we do logical partitioning. 

-> In logical partition (shared disk architecture), we do partition at node level (node = CPU + RAM). we fix each node which tuple it is going to handle. so queries involving those tuples will go to that specific node.
-> the benefits this approach offers are. 1. it reduces write conflicts across nodes as each node has it's own tuples to which it can only do write/read. 2. it increases cache hit 


-> Now, let come back to horizontal partitioning using hash based scheme (which is most common).  in this scheme, the major challenge which occurs when we want to do scale in/out. ex : initially we have 4 nodes. so to distribute rows, we do hash(key) % 4. but now we want to add new node i.e. 5th node. now we have to recalculate destination node for each node. i.e. hash(key) % 5. so this process can take much time for large dataset due to which we can face downtime. to solve it, we have consistent hashing algorithm. see consistent hashing notes for it.