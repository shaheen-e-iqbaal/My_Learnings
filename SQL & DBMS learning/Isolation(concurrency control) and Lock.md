
***ISOLATION***:

When multiple transactions run in parallel, the database must ensure a certain **level of isolation** between them — essentially defining what data from one transaction is visible to another concurrent transaction. For example, if one transaction is updating some data while another transaction is reading the same data, should the second transaction see the updated but **uncommitted** value, or only the **committed** old value? This behavior is determined by the **isolation level**. Depending on the needs of the application, you can choose the most appropriate isolation level to balance consistency, concurrency, and performance.

In 1992, the **ANSI SQL-92** standard officially defined four isolation levels: **Read Uncommitted**, **Read Committed**, **Repeatable Read**, and **Serializable**. For each level, ANSI defined the _minimum_ guarantees it must provide in terms of avoiding certain concurrency anomalies. At that time, three major anomalies were recognized: **Dirty Reads** (reading uncommitted changes from another transaction), **Non-repeatable Reads** (getting different values when reading the same row twice within a transaction because another transaction updated it), and **Phantom Reads** (new rows appearing in a result set between two queries in the same transaction because another transaction inserted them).

The ANSI SQL-92 guidelines can be summarized in a table showing which anomalies each isolation level must prevent at minimum. However, actual database implementations may choose to be stricter than the standard (for example, PostgreSQL’s `Repeatable Read` also prevents phantom reads by implementing Snapshot Isolation).

| Isolation Level  | Dirty Reads  | Non-repeatable Reads | Phantom Reads |
| ---------------- | ------------ | -------------------- | ------------- |
| Read Uncommitted | Possible     | Possible             | Possible      |
| Read Committed   | Not Possible | Possible             | Possible      |
| Repeatable Reads | Not Possible | Not Possible         | Possible      |
| Serializable     | Not Possible | Not Possible         | Not Possible  |

-> only three years after the SQL92 standard came out, a very impressive team of researchers published [A Critique of ANSI SQL Isolation Levels (1995)](https://arxiv.org/pdf/cs/0701157.pdf). The paper introduced a whole collection of anomalies that weren't specified in the standard, and therefore were technically allowed at the Serializable level (i.e. most strictest isolation level).

-> There are two strategies to implement transaction isolation levels(or concurrency control), 1. Optimistic 2. Pessimistic:
1. **Multi-Version Concurrency Control (MVCC)** – Maintains multiple versions of data rows to allow readers and writers to work in parallel without blocking each other. This is widely used in systems that want high concurrency with minimal locking.  It is the example of optimistic concurrency control strategy.
2. **Strict Two-Phase Locking (S2PL)** – Uses locks that are held until a transaction commits (both read and write locks), ensuring strict serializability but often with higher contention and blocking. it is example of pessimistic concurrency control strategy.

-> In pessimistic, we don't allow the conflict to occur since start. in optimistic, we assume that conflict will be rare, but when it occur, we will deal with them.
-> optimistic allows higher concurrency but there will be conflict which need retry of transaction. while pessimistic removes need to retry transaction as there will be no conflict but it can lead to deadlock problems.

Different databases adopt different approaches:

- **MySQL (InnoDB)** primarily uses **MVCC** for lower isolation levels, but for `SERIALIZABLE` it falls back to **S2PL** (locking reads) to prevent anomalies.
- **PostgreSQL** uses a variation of **MVCC** for all isolation levels. Starting from version **9.1**, it implements:
    - **Repeatable Read** using **Snapshot Isolation** (prevents dirty and non-repeatable reads, and phantom reads, but can still allow write skew — a serialization anomaly).
    - **Serializable** using **Serializable Snapshot Isolation (SSI)**, which adds conflict detection to snapshot isolation to guarantee full serializability without relying on blocking locks like S2PL.

-> **S2PL** ensures serializability without retries but reduces concurrency due to blocking. **MVCC** increases concurrency by avoiding most blocking but may require transaction retries when conflicts or serialization anomalies are detected.



--------------------------<<<<<<<<<<<<<>>>>>>>>>>>>>-----------------------

Basic Lock Types : 

1. Shared Lock : It is acquired by transaction when it want to do read operation. more than one transactions can have shared lock on database object like Table, Tuple, Attribute.
2. Exclusive Lock : It is acquired by txn when it want to do write operation. any single txn can have this lock on specific database object.

|                   | **Shared (S)**   | **Exclusive (X)** |
| ----------------- | ---------------- | ----------------- |
| **Shared (S)**    | ✅ Compatible     | ❌ Not Compatible  |
| **Exclusive (X)** | ❌ Not Compatible | ❌ Not Compatible  |
-> in above table, note that exclusive lock can be acquired only when no other txn has any lock i.e. shared or exclusive.

-> DBMS has Lock manager which maintains Lock-table where it stores information like : what transaction holds which lock, what transaction are waiting to acquire any lock.

--------------------------<<<<<<<<<<<<<>>>>>>>>>>>>>-----------------------


2 Phase Locking :

-> This locking strategy is used to implement serializable isolation level. MySQL uses it to implement serializable isolation level. it was discovered in early 1970 by software engineer at IBM.

-> in this, there are two phases. 1. Growing phase 2. Shrinking phase.
-> During growing phase, transaction acquires all the locks it will need. after acquiring all the needed lock, it can start releasing the locks. this phase where it starts releasing lock is called shrinking phase.
-> once transaction transition into shrinking phase, it can't acquire any lock during entire transaction life. i.e. it can't do like first acquire locks, then release some, then again acquire some.

There are two problems with 2 Phase Locking. 1. Dirty Read 2. Dead Lock

1.Dirty Read (Cascading abort) :
-> in ***normal two phase locking***, transaction can still get dirty read type anomalies, which is known as cascading abort. ex : T1 starts. it acquires exclusive lock on object A and B. it does some read and write operations on object A now it no longer want to do any read/write on A. so it releases exclusive lock on A. now T2 starts. it acquires exclusive lock on A. it does read then write operations. now T1 does some read and then write on B and then release exclusive lock on B. now T1 aborts. so the value T2 read of A is dirty one. so it leads to dirty read problem. 
-> this problem occurred because T1 released the exclusive lock on A even though it hasn't yet committed or aborted.

Solution :
-> so to solve this problem, most DBMSs use ***Strict/Rigorous 2 Phase locking*** strategy. so in this strict 2 phase, transaction can't release any lock until it either commits or aborts.
->so until the transaction doesn't get to it's final state (i.e. commit/abort), it can't release any acquired locks. so no other transaction can read/write to that object.


2.Dead Locks
-> It occurs ex : T1 is having lock on A but want to acquire on B. T2 has lock on B and want to acquire on A. both T1 and T2 keeps waiting for each other to release a lock. so this situation is called dead lock. there are two ways to solve this problem.
1.DeadLock Detection 2.DeadLock Prevention

1. DeadLock Detection : In this way, we allow the DBMS to have deadlock. but once it occurs, we detect it and abort either of the transaction. the aborted transaction is called victim of deadlock. now there are two questions. 1. How DBMS detect a deadlock 2. how to chose a victim.
	1.  How DBMS detects a deadlock? DBMS maintains a waits-for graph. when one transaction waits for other transaction to release a lock, ex: T1 waits for T2 to release a lock, then DBMS adds edge from T1 to T2 in waits-for graph. this way it prepares and maintains a waits-for graph. if there is cycle in a graph, then it means there is some deadlock between transactions. DBMS periodically runs a process to find out weather there is cycle or not in waits-for graph.
	2. to remove deadlock, DBMS aborts either of the transaction to break the cycle. this transaction is called victim. how does victim is selected? there are multiple strategies for that. 1. by age (lowest timestamp) 2. By progress (least/most queries executed) 3. By the # of objects blocked 4. By the # of txn we have to rollback/abort due to abort of this txn. we should also consider the # of times a txn has been aborted in past due to deadlock to avoid starvation of specific txn.


2. DeadLock Prevention : In this way, we don't allow deadlock to occur at firsthand. how? When any txn want to acquire a lock which is held by another txn, we kill/abort either of the transaction. so this approach doesn't require waits-for graph or any deadlock detection process. now again how will victim get selected? DBMS gives priority number to each txn based on the timestamps. older timestamps mean higher priority. using this priority, it figures out victim select based on two way. 
	1. Old waits for Young : if requesting txn has higher priority than holding txn, then requesting txn waits for holding txn. otherwise requesting txn aborts.
	2. Young wait for Old : if requesting txn has higher priority then holding txn, then holding txn aborts and releases lock. otherwise requesting txn waits.
	-> When a aborted txn by above strategy restarts, what will be it's (new) priority? will it be based on new timestamp or the old one? think of it.
		Spoiler :
```
		ans : based on old timestamp. why? to prevent it's starvation.
```


-----------------------<<<<<<<<<<<<<<<>>>>>>>>>>>>-------------------------


Optimistic Concurrency Control Protocol (OCCP):

-> when you assume / know that your conflict between your transactions are very rare (i.e. txns will rarely touch/access/modify the same db objects concurrently) and that most txn are short-lived, then forcing txns to acquire a lock adds unnecessary overhead. so to increase concurrency in such cases, there should be some protocol (ex : OCC) which allows access/modification of db object without any need to acquire locks.

-> So one of the protocol that implements OCCP is Optimistic concurrency control (OCC).
-> How OCC is implemented? How it figures out txns serializability? OCC uses Time Stamps to determine serializability of txns. each tuple of table will have one additional field to store time stamp of that row. time stamp of row tells, when was the last time this row was accessed/modified by any txn. 
-> each txn is assigned unique fixed time stamp that is monotonically increasing. postgreSQL uses 32 bit integer as a time stamp.

How OCC is implemented ?
-> For each txn, DBMS gives it a separate work space which is private to that only txn and no other txn can modify it. so when a txn wants to read/modify any tuple(s), DBMS creates private work space in memory for that txn, brings those requested tuple into that work space along with time stamp value of each tuple. note that due to this, Dirty Read anomaly will never show up.
-> when txn wants to commit, DBMS figures out if there is any conflict or not? if not, then it puts all those tuples into "Global" database. after which those updated tuples will be visible to all other txn. so this way, it guarantees Read committed level.

Phases of OCC :
1. Read Phase : In this phase, txn copies all the tuples from "global" db to it's private workspace. it will make all read/update on those tuple in it's work space. so this guarantees a Repeatable Read level.
2. Validation Phase : DBMS assigns a unique time stamp to this txn and then check whether it conflicts with other txns or not?
3. Write Phase : If Validation succeed, we push the work space changes to "Global" db.

How does OCC validation works? See CMU lecture note to understand it.

Limitations of OCC : 
-> It requires great amount of memory due to private workspace for each txn.
-> Validation/write phase requires additional computation.


Observation : in both S2PL and OCC, we only dealt with txn that read and update existing objects in the database. but now when txn performs insertion, deletion of objects concurrently, then we will have new problems.

-> based on above learnings, it is easy to understand how S2PL, OCC solves Dirty Read, non Repeatable Read anomalies. but how does they solve phantom read anomaly? ex: in case of S2PL, T1 locks some tuple which match it's predicate. meantime T2 comes and inserts some new columns and commits. some of these new columns match T1's predicate, so when T1 runs the same predicate again, those new rows will also appear.

How to solve it?

Approach 1 : Lock the entire Table or Database. if T1 does this, then T2 can't insert new rows. this approach is rare.

Approach 2 : Re-Execute scan : upon commit, DBMS re-runs all the where queries and sees whether it produces the same result as it was run first time in txn. it's also rare.

Approach 3 : Predicate locking : in this approach, DBMS logically determines overlap to predicate before queries start running. it's very rare.

Approach 4 : Index Locking  : mostly used.


--------------------------------<<<<<<<<<<<<>>>>>>>>>>>>---------------------

Isolation Level Implementation :

Serializable : No anomalies can occur. can be implemented with S2PL + Phantom protection (ex : Index Locking) (MySQL does this). (PostgreSQL uses SSI : Serializable snapshot isolation protocol to implement it.)

Repeatable Read : Phantom can occur. can be implemented with S2PL without phantom protection. (PostgreSQL use Snapshot isolation protocol to implement it which is based on MVCC. MySQL also use some modification of MVCC. but in both postgres and MySQL, phantom read doesn't occur at this isolation level)

Read committed : non-repeatable read + phantom read can occur. use S2PL without phantom protection. also S (shared) lock can be released immediately.



------------------------------<<<<<<<<<<<<>>>>>>>>>>>>-----------------------

MVCC (Multi Version Concurrency Control) :

-> Unlike OCC, Each txn doesn't get it's own private workspace. but each tuple in table stores two additional fields with it. named as xmin, xmax. 
-> xmin denotes the txn id of txn that created it. xmax denotes the txn id of a txn that modified or deleted it. 
-> in this protocol, Readers doesn't block writers and writers doesn't block readers. reading operation on specific db object can occur at anytime but one writing operation can occur at a time on specific db object. i.e. any one txn can write at any point of time on specific db object. to write, it needs to acquire exclusive lock which it will held until that txn doesn't get completed. reading txn doesn't need to acquire any lock like shared lock.

-> when a txn starts, it gets a consistent snapshot of db at that moment. it status of all the current running or just completed txn. so when it want to read any tuple, it will use this snapshot of txn state and will decide which version of tuple it has to read.


-> postgreSQL uses Snapshot isolation to implement Repeatable read isolation and Serializable snapshot isolation to implement Serializable isolation.

-> in case of postgreSQL, Repeatable read doesn't allow phantom read anomalies as it uses snapshot isolation level to find out which tuple it should read based on it's xmin and xmax value.

-> in case of read uncommitted level, postgreSQL doesn't allow even dirty read anomaly. so we can say that PostgreSQL's read uncommitted and read committed are same or postgreSQL doesn't support read uncommitted.

-> You can only get conflict of serialization error in case of Repeatable read and Serialization isolation level only.