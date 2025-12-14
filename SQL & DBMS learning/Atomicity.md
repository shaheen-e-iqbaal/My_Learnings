
-> Atomicity and Durability are two related terms in case of DBMS. if your DBMS guarantees Durability, then it means it automatically guarantees Atomicity.

-> DBMS achieves Durability / atomicity by mainly two techniques :

1. Logging (WAL)
2. Shadow paging


Logging :

-> database makes log of every action performed by each transaction. they keep these logs copy in memory and disk both. when db crashes, they can achieve recovery by using these logs. recovery of db can be bit slower but performance is really good.
-> black box of planes is one of the example of this technique where we record everything going on with plane.



Shadow paging :

-> the DBMS makes copies of pages modified by transactions and transactions make changes to these pages only. only when transaction commits is the page made visible. if db crashes in between, DBMS can discard those pages whose transaction hasn't yet committed. so recovery is fast. this is not good approach as it reduce performance.


--------------------------<<<<<<<<<<<<<>>>>>>>>>>>>>>----------------------


For Durability / Atomicity, DBMS has to ensure below things :
1. If DBMS tells somebody that txn has been committed, then the changes made by that txn must be visible no matter what happens after txn is committed.
2. If a txn gets aborted then partial changes made by it must also be aborted to ensure atomicity.

DBMS uses Undo to implement second guarantee and Redo to implement first Guarantee.

-> Undo means remove all the changes of aborted txn and Redo means apply all the changes of committed txn in case they are not persisted in db(non volatile memory) when that txn was committed.


WAL (Write Ahead Log) :

-> In this strategy, DBMS has one additional file called as Log file which is separate from actual data file. DBMS store this log file in disk.

-> This log file contains bare minimum information about the changes made by each transaction. in log file, whenever any txn updates any db object, at that instance, DBMS write something like old_value(will be used for Undo), new_value(will be used for Redo), txn_id, object_id... in log file. 

-> When a txn say T1 commits, DBMS has to make sure that all logs corresponding to T1 are flushed to log file in disk before the data files are flushed. so a txn is not considered as committed until all of it's log record are flushed to log file in disk.
notice that the flush of dirty data files can occur later but flush of log files must be done before telling outside world that txn T1 has been committed.


How PostgreSQL uses WAL ?

-> postgres is based on MVCC. when a new txn starts, postgres adds entries in log file like <"T1 start">. now whenever any T1 updates any tuple, it creates a new version of that tuple and applies the changes in it. before making new version of that row, DBMS first records the changes made by T1 in WAL log file like <T1, page_no, old_value, new_value>. then creates a new version of that tuple and apply changes in it.

-> now when T1 commits, it will add entry like <"T1 Commit"> in log file and flush that memory log file into disk log file. after that it will update the status of T1 from active to committed in transaction status table which is managed by postgreSQL to track status of each txn. then it will tell world that T1 has committed. this makes commit process fast.

-> notice that till this time, the actual data file present in memory hasn't yet flushed to the disk. now suppose the db crashes. when it recovers, it will bring back all the log file in memory. now it will read that log file and will Redo the changes committed by T1. so this way WAL helps in ensuring durability.

		-> now when will actual data file get flushed to the db? DBMS does this at some fixed interval of time. it will flush all the dirty data pages present in memory to disk. this process is called Checkpointing. note that, some page might have uncommitted changes as well. this changes will also get flushed. postgres uses MVCC. so for each tuple which has been updated, there will be separate tuple with different version. so postgres can decide which tuple version to keep based on the txn status which has created / modified it. 

-> so in case of postgres, due to MVCC, it doesn't need to use anything for Undo. but for Redo, it uses WAL. 

-> So, commit process is fast in postgres, but recovery can be slow.

-> Note that, WAL files can grow too much in size as there can be so much update in db. so how postgres keeps WAL size in control? ans : when checkpointing process occur, it add it's log into WAL file like <"Checkpoint">. so at this point of time, all the changes are flushed into disk. so we can discard the log of those txns which got committed before this checkpointing process.


-> One question can occur is that, every time we flush, it makes I/O call. doesn't it make application slow? without WAL, we have to flush data files directly when a txn commits. this data files can be different pages which requires multiple I/O calls to flush. but this log file is single file, which will be flushed in single I/O call. so this tradeoff is batter than flushing data files every time.


```
PostgreSQL uses Write-Ahead Logging (WAL) to ensure durability and crash recovery. Since PostgreSQL is based on MVCC (Multi-Version Concurrency Control), when a new transaction starts, PostgreSQL first records an entry like <T1 start> in the WAL. Whenever the transaction modifies a tuple, it doesn’t overwrite the existing one. Instead, it creates a new version of the tuple (MVCC). But before making that change in memory, PostgreSQL writes the details of the modification to the WAL in the form <T1, page_no, old_value, new_value>. Only after logging the change does it update the in-memory page with the new tuple version.

When the transaction commits, PostgreSQL writes a <T1 commit> record to the WAL and **flushes the WAL buffer to disk**. It also marks the transaction status as _committed_ in its transaction status table. At this point, PostgreSQL can safely acknowledge the commit to the client — even though the actual modified data pages in memory are not yet written to disk. This makes commits very fast.

-> Now when db crashes, DBMS required to Redo all the changes of committed txns and Undo all the changes of uncommitted txns.

How Redo will be done?
If the database crashes before the modified data pages are flushed, recovery simply - It reads the WAL file from the last known good state. - It replays (redoes) all the changes from committed transactions, rebuilding the lost in-memory changes. This guarantees that no committed work is lost.

How Undo Will be Done?
Afte crashe, Since PostgreSQL uses MVCC, it does not require undo logs — it can just ignore uncommitted tuple versions (which were flushed to disk during some earlier checkpointing process) based on transaction status. 

However, due to need to apply WAL file (Redo logs), recovery may take longer.

The actual data files (dirty pages) in memory are written to disk later, during a process called checkpointing. At each checkpoint, all modified pages are flushed, and a <Checkpoint> record is added to the WAL. After this point, WAL records of committed transactions before the checkpoint can be discarded, which keeps the WAL size under control.

Although WAL itself adds I/O writes, it is still more efficient than flushing data pages directly on every commit. Without WAL, committing would require flushing multiple scattered data pages (many I/O calls). With WAL, only a single sequential log file needs to be written, which is much faster for disk I/O.

In summary: PostgreSQL uses WAL for redo, relies on MVCC instead of undo, makes commits fast, recovery slower, and checkpoints help control WAL size.
```