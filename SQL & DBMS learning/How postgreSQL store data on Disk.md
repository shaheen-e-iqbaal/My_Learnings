
-> There are multiple ways to store data in disk. 1. Tuple oriented storage. 2. Index oriented storage. 

-> PostgreSQL uses **tuple-oriented (row-oriented) heap storage** for its tables. Indexes are stored separately and contain TIDs pointing into the heap. PostgreSQL does not use index-organized storage by default.

-> In **index-oriented storage**:

- The **table itself _is stored inside the index_**.
- Rows (tuples) are arranged in **sorted order of the primary key (or clustering key)** inside the B-Tree.
- There is no separate heap file — the leaf nodes of the index directly store the full tuple.
So, in case of index oriented storage, as the tuples are stored in increasing order of the primary key, the primary key should be like incremental integer, sequenced UUID... random UUID will cause problem here. so while using MySQL with InnoDB, don't use random UUID as PK, but use sequenced UUID or auto increment integer.

-> So the index oriented storage is good for primary index based read, but bad for insert, delete operations as we have to modify the B+ Tree.

----------------------------------<<<<<<<<<<<<>>>>>>>>>------------------------




How PostgreSQL stores Data :

-> File Structure : ../<"Postgre_version">/main/.... : here multiple files are there. like pg_wal : which stores WAL (write ahead log), pg_version, base : which stores users databases, pg_xact : which stores information like txn status. earlier it is called pg_clog,....

-> Each user database has it's own subdirectory under ../base folder. i.e. postgres creates subdirectory for each client database.

-> inside this subdirectory, there will be multiple files which are called heap files. for each table, postgres creates a new heap file. this heap file is made up of multiple pages. pages are where tuples gets stored. in case of postgres, pages are of 8 KB fixed size. 

-> each page has it's own header, which contains meta data about that page like checksum. ***after header it has one array called slot array where each entry of this array is pointer to the tuple stored in that page***. so each tuple is identified using TID (tuple identifier) which is pair of two value : (page_Id, offset) : page_id is id of a page where this tuple is stored and offset is the location of array index which points to this tuple. 
-> so indexes stores this TID for each tuple.

-> when you delete any tuple, postgres just marks the pointer present in slot array as null. 
-> during the vacuum process, postgres removes this deleted tuples from page and updates the slot array to point to new location in page.
-> when you want to insert some new tuple in some table, postgres first checks the fsm (Free space map) of this table. (besides the heap file of each table, postgres creates two new files for each table as well. 1. fsm (free space map) : which keeps information about free space available in each page present in this heap file. 2. vm : visibility map.) so from this FSM, it will find a page where it should insert that tuple and will insert there. 


-> Q. What do we achieve using this slot array? ans : after the vacuum process runs, it removes some non visible tuples from page and reorganize page to remove in between space which were left by removed tuples. due to that location of some tuples changes. so we can update this new location of tuple into their slot array. all indexes give location of this slot array, not actual tuple location, so we don't need to update indexes or other thing which point to this tuple. we make all things to point this slot array entry of that tuple, whose location is intact even after vacuum. 

-> so insertion of new row is random in any page. this is the main property of this heap data structure which stores these pages. 


-> Now let's say you want to read some tuple : ex : SELECT * FROM User where id = 5;

case 1 : You don't have index on id.

-> postgres query manager will first look into memory buffer pool that this tuple is present or not. if not then it will ask buffer pool to fetch this tuple from disk.
-> now based on the table name i.e. User, it will find out it's heap file. now in this heap file there will be multiple pages. 
-> now in this heap file, it will go scan each page from the start. it will bring each page in memory, will scan tuple for id = 5, if found then will stop there otherwise will continue it.


Case 2 : we have indexes on id.

-> postgres first go to index file of this table, from there it will find out TID which is pair of (page_id, offset).
-> from this page_id, it will calculate the physical location of this page_id in heap file. ex : each page is of fixed 8 KB size. so page with page_id = 2 will have location = 2 * 8192. now it will go to User table heap file. inside that it will directly fetch page present at location calculated above and will bring it into buffer memory.
-> in memory, from the offset value, it will use slot array and will directly go to the location of that tuple. 


Q. Imagine you create a table in PostgreSQL with columns like `id`, `title`, and `content`, where `content` is of type `TEXT` to store articles or documents. Now, suppose you try to insert a row where the `content` column holds a very large string—say several megabytes of text. Since PostgreSQL stores rows (tuples) inside fixed-size **8 KB pages**, the database quickly runs into a problem: this single row cannot physically fit into one page. Unlike fixed-width columns such as `INT` or `BOOLEAN`, large variable-length types like `TEXT`, `BYTEA`, `BLOB` or `JSONB` can easily exceed the page size, making it impossible to store them directly in the heap.

To solve this, PostgreSQL uses a mechanism called **TOAST (The Oversized-Attribute Storage Technique)**. When a value is too large to fit on a page, PostgreSQL first attempts to **compress** the attribute using its built-in pglz compression. If the compressed value still does not fit, PostgreSQL automatically creates a hidden **TOAST table** associated with the main table. The oversized value is then **split into smaller chunks** (about 2 KB each) and stored as multiple rows in this TOAST table. In the original heap tuple, instead of storing the large value itself, PostgreSQL keeps only a **pointer** that references the chunks in the TOAST table. When the row is queried later, PostgreSQL transparently fetches the chunks, decompresses them if necessary, and reconstructs the original value.

In this way, even if your tuple contains attributes far larger than the 8 KB page size—like multi-megabyte text, JSON documents, or images stored as binary—PostgreSQL can still store and retrieve them efficiently using TOAST.


```
Your notes are very close. Here’s a cleaned-up, corrected, and slightly expanded version in plain prose.

 In a PostgreSQL data directory (PGDATA) you’ll find top-level subfolders such as `pg_wal` (the write-ahead log that guarantees durability), `pg_xact` (commit/abort status bits for transactions; this used to be called `pg_clog`), `global` (cluster-wide catalogs), and `base` (all user databases). Under `base/`, each database has its own subdirectory named by its OID; that directory holds the physical files for that database’s tables and indexes. Every table is a “relation” stored as its own file (its relfilenode); very large relations are split into 1-GB segments with suffixes like `.1`, `.2`, etc. Each relation actually has “forks”: the main heap file, an FSM file (free space map), a VM file (visibility map), and for UNLOGGED tables an init fork. Large column values may spill into a separate, automatically managed TOAST table and its index.

 A table’s heap file is an unordered collection of fixed-size pages (blocks). The default page size is 8 KB (set at build time for the cluster). Each page contains a small page header (optionally carrying a checksum if you initialized the cluster with checksums), then an array of item/line pointers (the “slot array”), and the tuple bodies themselves. Pointers live near the front of the page; tuple bodies are packed from the end of the page toward the front; the free space sits in between. Every row (tuple) is identified by a TID, which is `(block_number, offset_number)`. The `block_number` picks the page inside the relation; the `offset_number` is the index of the line pointer on that page. An index stores keys alongside these TIDs. Given a TID, the storage manager seeks to `block_number × BLCKSZ` in the table’s heap file, reads that whole page into the shared buffer pool, and then follows the line pointer at `offset_number` to the tuple; this is why even an index scan always fetches the entire 8 KB page that contains the row.

 When you insert, PostgreSQL does not just append blindly; it first consults the table’s Free Space Map (the separate `_fsm` fork) to find a page that advertises enough free space for the new tuple. The FSM is a compact, approximate tree that lets the server locate candidate pages quickly. If the chosen page turns out to be too full (the FSM can be stale), PostgreSQL tries another candidate and, if needed, extends the relation by adding a brand-new empty page. After placing the tuple it updates the FSM to reflect the new free-space amount. So inserts are not “random” so much as “wherever a page has room,” which often spreads new rows across older pages as space becomes available.

 For reads without an index, a sequential scan opens the table’s heap and walks pages in order, bringing each page into the buffer pool and checking each visible tuple against the predicate. Unless the plan can stop early (e.g., `LIMIT 1`), it keeps scanning to the end to guarantee correctness. With a B-tree (or other) index, the executor traverses the index to find matching keys, collects their TIDs, and for each TID fetches the referenced heap page and tuple, still applying MVCC visibility checks. If the query’s columns are all present in the index and the page is marked all-visible in the Visibility Map (`_vm` fork), PostgreSQL can often skip visiting the heap entirely—this is an index-only scan. The visibility map has one bit per heap page indicating “all tuples visible to all transactions” (and an “all-frozen” bit for VACUUM), letting both VACUUM and index-only scans skip work.

 Deletes and updates don’t remove or overwrite rows in place. A DELETE sets the tuple’s `xmax` in its header to the deleting transaction and leaves the line pointer intact; the row becomes non-visible to future transactions once the delete commits. An UPDATE creates a new version of the row (a new tuple, usually with a new `ctid`) and marks the old one’s `xmax`; when the new version can stay on the same page, PostgreSQL may create a HOT (heap-only tuple) chain to avoid index updates. Normal VACUUM later identifies dead tuples, marks their line pointers as reusable, and compacts the page to reclaim space. It updates line pointers to where tuple bodies were moved during compaction but keeps the pointer numbers themselves stable for live tuples, so a live row’s `ctid` (which is `(block, offset_number)`) does not change because of VACUUM; `ctid` _does_ change on UPDATE because the row version changes. Over time, operations like `VACUUM FULL`, `CLUSTER`, or moving a table to a different tablespace can rewrite the table and assign a new relfilenode (i.e., a new physical file name), but the catalogs track that mapping.

 Finally, two small but useful details. First, you can see a row’s physical location via the system column `ctid`; it shows the same `(block, offset_number)` that indexes store. Second, indexes are separate relations with their own files and page layout; B-trees have a meta page, internal pages, and leaf pages that store keys plus TIDs, enabling direct hops into the heap. Put together, the flow is: name → catalogs → relfilenode → heap file → page (by block) → line pointer (by offset) → tuple → MVCC check, with FSM and VM speeding up insert placement and heap skipping, respectively.
```

-> Use [This article](https://www.interdb.jp/pg/pgsql01/03.html) to see the Page structure (particularly header, slot array, actual tuples data) of postgres and [This article](https://www.interdb.jp/pg/pgsql01/02.html) to understand postgres file system.

-> See the Screenshot named postgres_file_structure. there you can see that each user table has three files. 1. actual table 2. table_fsm (contains free space info) 3. table_vm : visibility map.



----------------------------<<<<<<<<>>>>>>>>>>-----------------------------------