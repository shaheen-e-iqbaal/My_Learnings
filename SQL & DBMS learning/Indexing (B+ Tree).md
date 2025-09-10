
-> Indexing can be implemented using balanced binary search tree data structures like Red-Black Tree, B-Tree. but B+ Tree is non binary. each node can have more that one two child node. we want to maximize the number of child node each node can have.

-> Most of the DBMS use B+ Trees. B+ Trees is one of the data structures of B Tree family. It was first discovered in 1979. the modern B+ Trees used by databases is not same of the 1979 one but have multiple modification/improvements. 

-> ***Each node of B/B+ Tree is stored as separate page in heap file of that index.***

B-Tree vs B+ Tree :

-> A **B-Tree** stores data in both its inner nodes and leaf nodes, whereas a **B+ Tree** stores actual data only in the leaf nodes. The inner nodes in a B+ Tree act only as guides to help navigate quickly to the right leaf node. This difference has an important impact: since B-Trees keep both keys and data in inner nodes, they can store fewer child pointers per node, which makes the tree taller. In contrast, B+ Trees store only keys in inner nodes, which allows them to hold more child pointers and keep the tree shorter. A shorter tree means fewer levels to traverse, so queries require fewer disk page reads.

-> Another advantage of B+ Trees is that their leaf nodes are linked together like a doubly linked list. This makes **range queries** (e.g., `SELECT name FROM table WHERE id BETWEEN 100 AND 200`) very efficient. The database first finds the leaf node containing the starting key (100), and then it can simply follow the linked leaf nodes sequentially until it reaches 200. It doesn’t need to go back up to inner nodes to find the next key, as would be required in a B-Tree. Because the leaf nodes are stored in order and linked, the database can read large chunks of them in sequence, which reduces **random I/O** on disk and turns it into mostly **sequential I/O**. This is a major reason why B+ Trees are the preferred structure for indexes in databases.

 P. S. :  `When we say **B+ Trees reduce random I/O**, we are mostly talking about **I/O to fetch index pages (leaf nodes of the index)**, not the data pages themselves.`

- `In a **clustered index** (like in InnoDB), the leaf nodes of the B+ Tree are actually the data pages themselves. So in that case, reducing random I/O applies directly to fetching data.`
- `In a **non-clustered/secondary index**, the leaf nodes contain key + row pointer (or TID in Postgres). So the sequential I/O benefit is mainly at the **index leaf level**, while fetching the actual data pages may still involve random I/O.`


-> Read [This stack over flow thread](https://stackoverflow.com/questions/870218/what-are-the-differences-between-b-trees-and-b-trees) to understand difference between B/B+ Tree.


Properties of M-Way B+ Tree :

1. Perfectly Balanced Tree : All leaf nodes must be at same depth from the root node.
2. Each node can have up to M child node.
3. each node except root node must have at least M/2 keys. i.e. keys present in each node must be M/2 - 1 <= #keys <= M

-> Inner nodes content will be like key -> pointer to child node. this pointer is nothing but actually a page_id of the child node page. Leaf nodes content will be like key -> pointer to tuple in head file i.e. TID or ctid (in case of postgres).

-> in case of MySQL (InnoDB) or more generally Index oriented storage databases, the content of leaf node will be key -> Actual tuple data. this will be for primary index. in secondary index, the content of leaf node will be key -> page_id (page_id will be the id of page of primary index leaf node where this key's data is stored).


-> In case of postgres, as page size is of 8 KB, multiple entries can be stored in single B+ Tree node which will have it's own page in heap file. for inner node, the entries looks like Key -> Pointer to child node (i.e. page number of child node page). for leaf node, the entries looks like Key -> TID (or ctid in case of postgres, rowId in case of oracle). so the main point is, in each node, multiple such entries can be stored. the maximum entries stored can be called M or degree of tree.

---------------------------------------<<<<<<<<<<<>>>>>>>>---------------------------------

When will index gets updated? 

-> Due to insertion of new tuple in table, DBMS have to insert that key in index also. so during that, it corresponding node is already full, then it will lead to page splitting.

-> Due to deletion of some tuple from table, DBMS also have to remove it's key from index. so during that operation, if the #keys  present in node becomes less than half, then it will lead to node merging.

-> Due to updation of one or more attributes of tuple, if there is any index on any of this attributes, then all the indexes of attributes which are updated will also gets updated. 


Q. Will these indexes get updated after transaction commits or instantly when it performs operation which needs index to be updated?
ans : When a transaction performs **INSERT, UPDATE, or DELETE** on a table, the indexes are updated **immediately as part of that same operation**, not delayed until commit. The backend process (the transaction’s worker thread) is responsible for applying the index changes. This ensures that any subsequent query within the same transaction can already use the new index entries if needed. However, visibility to _other concurrent transactions_ is governed by MVCC (transaction IDs and snapshot rules).
- **INSERT**:  
    When a new tuple is inserted, PostgreSQL first allocates space for the tuple in a heap page (in memory, via shared buffers). This tuple gets a **CTID** (which is essentially `<page_id, offset>`). Because the page and offset are already determined inside the buffer pool, Postgres knows the CTID even before the page is written to disk. It then creates index entries in all indexes for that table, pointing to this CTID. If another concurrent transaction scans the index and finds this entry, visibility checks (using the tuple’s `xmin`, snapshots, and isolation level) decide whether it should be seen.
- **DELETE**:  
    A DELETE does not physically remove the row or its index entries immediately. Instead, the tuple in the heap is marked as deleted (with `xmax`), and the corresponding index entries are updated with this “dead” status. The actual removal of dead rows and index entries happens later via **VACUUM** or HOT (Heap-Only Tuple) pruning. If the deleting transaction aborts, the delete mark is rolled back, so the row remains valid.
- **UPDATE**:  
    PostgreSQL treats UPDATE as a two-step process: mark the old tuple as deleted, and insert a new version of the tuple. Both tuples remain in the heap, linked via HOT chain if possible.
    
    - If the updated columns do **not** participate in any index, Postgres can use HOT update and avoid touching indexes — the new tuple reuses the same index pointers.
    - If indexed columns are modified, Postgres must insert new index entries pointing to the new CTID, while marking the old ones as dead.

-> For all of these operations, PostgreSQL uses the standard B+Tree rules for maintaining index structure. When inserting, the system locates the appropriate leaf node page, and if space is available, the new key is added there. If the page is full, a **page split** occurs, and parent pages may be updated recursively. In the case of deletion, PostgreSQL finds the relevant leaf page and marks the key as dead. If after many deletions a page becomes underutilized (size < M/2 entries), then a **page merge** may eventually occur as part of B+ Tree maintenance, though in PostgreSQL some cleanup is handled lazily via VACUUM and doesn’t always trigger immediate merges.


Q. If i update a some attributes of a column, postgres will create a new tuple. so due to the insertion of this tuple, it will have to change all the indexes present on this table even though i haven't changed indexed columns. this makes updates costly.
ans : To overcome this problem, postgres uses optimization called HOT (heap only tuples). 
- If:
    1. The update changes only **non-indexed columns**
    2. The updated tuple can fit **in the same page** as the old one

👉 Then Postgres creates a **new tuple in the same heap page** and **does NOT update the indexes**.

- Instead, the index entry still points to the old tuple, which has a chain to the new tuple.
- Index lookup follows the chain and lands on the latest visible version.
This saves **a lot of index writes**.

---------------------------------------<<<<<<<<<<<>>>>>>>>---------------------------------


Type of Indexes : 

Primary Index : It is the index build using primary key of table. postgres automatically creates index on primary key. in case of index organized storage databases, this primary index's leaf nodes stores the actual tuple data. but in case of tuple organized storage databases like postgres, the leaf node stores TID only.


Secondary Index : it is index build on some secondary key other than primary key. we can build as much secondary index we want. but building unnecessary indexes cause slow write as we have to update all relevant indexes of that table. MySQL's secondary indexes stores the primary key of tuple as value instead of TID or ctid(in case of postgres). notice that due to this, MySQL first has to find PK then scan PK index to get actual data. so this requires to index lookups. while Postgres secondary index stores ctid as value. so in single index look up you can find physical location of tuple.

Q. What will happen if there are duplicate keys on which index is built?
ans : First thing here to note is that, this problem will only occur in secondary indexes not in primary indexes. because primary indexes are build on top of primary key which is always unique. so now how will it read this index? ex you have index on secondary key "username" which is not unique. so when you do insert, it will first find the appropriate leaf node where this username can be inserted. then it will travel this leaf nodes to the last key with this username. now it will insert this new username in that leaf node. similarly, for read, it will go to the first leaf node with this username and will travel right side till the last leaf node containing this same username. and will collect all the TIDs and will do heap I/O call.


Composite Index : We can build B+ Tree index on multiple composite attributes. if our primary key is of composite type, then primary index will be composite index. if you have made indexing on columns ex C1, C2, C3 and you make query involving all three, C1, C1 and C2 then DBMS will use this index. if you make query using only C2, C2 and C3, C3 then DBMS can't use this index. so index can be used on prefix of column not on suffix of columns. the way postgres creates composite index is first order based on C1, then on C2 then on C3. 


Clustered index : A **clustered index** means that the actual table data (the heap file) is physically stored on disk in the same order as the index. In index-organized databases like MySQL’s InnoDB, the primary index is automatically clustered, because the data itself is stored within the index structure. But in tuple-oriented storage engines like PostgreSQL, the table is stored as a plain heap, and you have to explicitly issue a `CLUSTER` command to reorganize the heap in the order of an index (often the primary key). This operation physically rewrites the whole table. However, PostgreSQL does not maintain this clustering automatically — after new inserts or updates, rows may end up in random positions in the heap file, breaking the clustered order. To restore clustering, you must re-run the `CLUSTER` or use `pg_repack`.

The main advantage of a clustered index is that it reduces **random I/O**, since related rows that are close in index order are also close on disk. This is especially helpful for range queries and sequential scans. However, even without a clustered index, databases can still reduce random I/O through a technique called **index-only batch fetching**. When the index lookup identifies tuples that live on different pages in the heap, instead of fetching each page one by one (causing many random I/O calls), the database first collects all the required page IDs, sorts them, and then reads the heap pages in sequence. This way, the disk reads are performed mostly in order, turning what could have been random I/O into mostly sequential I/O, which is much faster.

In summary, clustered indexes improve performance by physically aligning data with index order, but even without them, databases use smart page-fetching strategies to minimize random I/O.

-> In case of postgres, this index-only batch fetching is called bitmap scan. 
```
### What is a Bitmap Scan?

 A **bitmap scan** is an access method in PostgreSQL used when an index lookup would return many rows spread across multiple heap pages. Instead of fetching each page immediately from the heap (which could cause a lot of random I/O), PostgreSQL first builds a **bitmap of heap page locations** that need to be visited, and then fetches the relevant pages in sorted order.
```

Index Only Scan :

-> In index entry, you can add some addition attributes which you want. ex: you want to build on email of person. now the entry in index leaf node will be like key (email in this case) -> TID. but if you want to add some more attributes value in this entry, like you want to add person name, person id, you can add those as well. so new entry will look like key (email) -> TID + (p_name, p_id). so when you write query like select p_name from person where email = 'something@..', then it will find out leaf index node with given email, and from that node page itself, it will get p_name. so it doesn't need to require make separate heap call. so this is called "Index only scan". but this can reduce the number of entries which can be stored in each index node page. due to which new index node page needs to be added which can make tree taller.