
**Deadlock :**

->It is an un-expected condition which arises in concurrent environment. when multiple threads, processes wait for each other to release something which they are holding currently to proceed with their execution. each thread is holding some locks, wants to acquire some new locks which are currently acquired by some other locks. that other thread wants to acquire the locks which this current thread is holding. so each one is waiting on each other to release something so they can acquire it.

-> In normal OS, below are the four necessary conditions for Deadlocks known as Coffman conditions : 

1. Mutual exclusion : if the locks are of the type that, only one thread/process can hold it at a time.
2. Hold and Wait: A process is currently holding at least one resource while simultaneously waiting to acquire additional resources held by other processes.
3. No Preemption: Resources cannot be forcibly taken from a process; they can only be released voluntarily.
4. Circular Wait: A closed chain of processes exists, where each process holds a resource that the next process in the chain needs.

Prevention Techniques

To prevent deadlocks from happening in the first place, systems are designed to eliminate at least one of the four conditions above: 

- **Eliminating Mutual Exclusion:** While some resources like mutex locks cannot be shared, you can make others shareable. For example, using **spooling** for a printer allows multiple processes to submit print jobs without needing exclusive access to the hardware itself.
- **Eliminating Hold and Wait:** The system requires a process to request and be allocated all of its required resources at once before it begins execution. Alternatively, it dictates that a process can only request resources when it holds none. This ensures a process never waits for a resource while holding another.
- **Eliminating No Preemption:** If a process holding some resources requests another resource that cannot be immediately allocated to it, the system will preempt (forcibly release) all the resources currently held by that process. The process will only restart once it regains all of its old resources along with the new one.
- **Eliminating Circular Wait:** This is the most practical and widely used approach. Resources are assigned a unique, strict numerical ordering. Processes are only allowed to request resources in an increasing (or monotonically ascending) order, meaning a process can never request a lower-numbered resource if it is currently holding a higher-numbered one.


-> In JVM environment, there isn't any mechanism where JVM tries to bring application out of deadlock once it occurs. we have to design our applications in such a way that deadlocks doesn't occurs. in Java, we can use below strategies to avoid deadlocks :

1. Lock ordering : make sure that threads acquire locks in same order. like if multiple threads wants lock A, B, C. they all acquire in same order i.e. A then B then C. so with this, if some thread has already acquired A, no other thread will be able to acquire B or C because before acquiring B or C, they have to first acquire A. 
2. Timed waiting : use methods like `tryLock(time)`. this way, if your thread doesn't get lock after sometime, it will fall back instead of slipping into indefinite waiting.This does not prevent deadlocks by itself, but it prevents threads from waiting indefinitely. After a timeout, the thread can release any acquired locks, retry the operation, or execute an alternative recovery strategy.
3. Keep your critical section smallest possible. i.e. acquire lock only when it is required and release them as early as possible.


-> Databases have mechanism to detect deadlock and strategies to come out of it. they maintain waits-for type graph to keep track the txns waiting on each others. if there is any cycle in graph, then deadlock is there. refer isolation (concurrency control) section of SQL & DBMS learning.

---

**Starvation**

-> When multiple threads are blocked on some lock. but any particular thread is not getting this lock since the long time. the thread which started just now, got the lock but the thread which is waiting since last ex 1 hour doesn't get it. this is called thread starvation. 
-> We can remove it by uses lock fairness. monitor locks doesn't allow any fairness. but other locks like re-entrant, read-write takes boolean parameter in constructor, whose value tell that : have to keep fairness in lock allocation or not?