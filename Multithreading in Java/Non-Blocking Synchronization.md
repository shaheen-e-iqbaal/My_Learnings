
-> We can achieve synchronization by using lock or without locks, by blocking way or  non-blocking way, by optimistic way or pessimistic way. so when someone asks, 1. how would you implement non-blocking thread synchronization 2. how would you implement lock-free synchronization 3. how would you implement optimistic synchronization. they mean to ask same thing.

-> we can use locks like monitor, re-entrant, read-write, semaphores (or pessimistic locking)... to achieve lock based / pessimistic / blocking synchronization. but the issue with this type of synchronization is that, we can have situations like deadlock, thread starvation if no fairness is there. 


-> to solve this issues of lock based synchronization, CPU provides us some mechanisms like CAS (Compare and Swap). which guarantees that, the operation of Read -> Update -> Write will be performed atomically as a single operation. this is low-level mechanism which is implemented by CPU. 

-> ***Optimistic locking can also be said lock free mechanism to achieve synchronization. but it is implemented/guaranteed by application logic rather than any hardware level guarantee. while CAS is guaranteed by hardware level.***

-> Java has provided Atomic classes that uses this CAS. such classes are `AtomicInteger`, `AtomicBoolean`, `AtomicLong`, `AtomicReference`...

`atomicVariable.compareAndSet(expectedValue, newValue)` : this is the main method which performs CAS. other methods like `increamentAndGet`, `getAndAdd` uses this method behind the scene to perform atomic operation. it will check is the current value of `atomicVariable` is equal to the `expectedValue` or not? if yes, then it will update the value of `atomicVariable` with `expectedValue`. this entire operations (i.e. two operation 1. comparing 2. updating) will be done as single operation atomically. if no, then it will return false.


 ***Advantage of Lock Free over Lock Based mechanism :***
1.  in Lock based, when a thread doesn't get lock, it will be blocked. which will prompt CPU to do context switch which is costly. in lock-free, thread doesn't have to acquire any lock, it will always be running.
2. in lock based, we can have situations like deadlock. while in lock-free, we don't get any such situation.

***Disadvantages of Lock Free over Lock based mechanism :***
1. CPU consumption increases. in highly concurrent time, threads might keep failing and re-trying to succeed. this will consume lot of CPU cycle.
2. ABA problem. `AtomicStempadeReference` is the class provided by Java to solve this ABA problem.

-> ***==all Atomic classes (`AtomicInteger`, `AtomicBoolean`, `AtomicLong`, `AtomicReference`...), stampede lock, `ConcurrentLinkedQueue`, `ConcurrentLinkedDeque` allows us to implement non-blocking/lock-free/optimistic synchronization. while locks (monitor, re-entrant, read-write, semaphore), blocking queues, collections like `ConcurrentHashMap`, `CopyOnWriteArrayList ... gives us blocking/lock-based/pessimistic synchronization.

