
It can be achieved using two ways :

1. Lock Free mechanism (Optimistic locking)
2. Lock based mechanism (Pessimistic locking)


Lock Free (Optimistic) :

-> In this case, we don't use lock explicitly. we just implement a ways which gives functionality which looks like we have put lock. 
-> Postgres uses this type of locking. when we want to read some rows which are being read or being changed by other transaction, we can read them. but when we want to write something, we have to get lock. so at most one transaction at a time can write to any row. 
-> AtomicInteger, AtomitLong etc. in java use CAS (compare and swap) concept which guarantees consistency of data in multithreaded environment without using any locks like monitor... behind the scene it uses low level CPU instructions to read, compare and then write a value to specific memory location. all these three operations combined are atomic.


Lock based mechanism :

-> in this way, we use locks like monitor, reentrant etc. to achieve proper concurrency. we place a lock on resource so no other thread can access it.
-> databases use this mechanism to avoid concurrent write operations on a same resources.

