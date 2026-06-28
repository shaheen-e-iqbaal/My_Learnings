
**Blocking Queue :**

-> Class like `ArrayBlockQueue`, `LinkedBlockingQueue` implements interface `BlockingQueue`. these queues are thread safe alternatives of normal queue. 

-> `ArrayBlockingQueue` is bounded i.e. we have to specify the size of it while creating it's instance. this is because it uses array internally. 
-> `LinkedBlockingQueue` is un-bounded. we don't have to give any size while creating it. we can give it suggestions on possible capacity. but the queue can grow more than it without any problem. this is because it uses Linked list.

-> These queues synchronizes using `Reentrant` locks. when queue is at it's full capacity and thread wants to insert new value, it will have to wait until some space becomes available.
similarly, when queue is empty and threads want to remove something, they have to wait until queue becomes non-empty. so basically, they implements producer-consumer synchronization using locks.

-> BlockingQueue is a thread-safe queue designed for producer-consumer scenarios. It supports blocking operations, allowing producer and consumer threads to wait automatically instead of busy waiting.

-> Most queue operations are provided in four variants depending on how they behave when the operation cannot be completed immediately:

Throws Exception immediately– add(), remove(), element()
Returns Special Value immediately– offer(), poll(), peek() (false/null on failure)
Blocks Indefinitely – put(), take() (waits until the operation becomes possible)
Blocks with Timeout – offer(timeout), poll(timeout) (waits up to the specified time before giving up)

-> BlockingQueue does not allow null elements. Methods such as add(), put(), and offer() throw NullPointerException if null is inserted. This is because null is reserved as a special return value by methods like poll() to indicate that no element is available.

-> All individual queue operations are thread-safe and atomic, achieved using internal locks or other concurrency mechanisms. However, bulk collection operations such as addAll(), removeAll(), retainAll(), and containsAll() are not guaranteed to be atomic unless explicitly stated by the implementation. As a result, they may succeed only partially if an exception occurs midway.

-> The size() method returns only a snapshot of the queue size at the moment it is called. Since other threads may simultaneously add or remove elements, the returned value can become outdated immediately. Therefore, size() should not be used to decide whether operations like put() or take() will block. Instead, rely on the blocking methods themselves (put(), take(), offer(timeout), poll(timeout)) for correct concurrent behavior.

-> Most BlockingQueue implementations provide weakly consistent iterators. An iterator never throws ConcurrentModificationException if the queue is modified while it is being traversed. It may reflect some, all, or none of the modifications made after the iterator was created, but it always traverses elements safely without failing. Unlike CopyOnWriteArrayList, the iterator does not traverse a fixed snapshot; instead, it may observe concurrent updates as the traversal progresses. This behavior avoids locking the queue during iteration while still allowing concurrent producers and consumers.


Non-Blocking Queue : 

-> ConcurrentLinkedQueue is a thread-safe, non-blocking implementation of Queue, while ConcurrentLinkedDeque is a thread-safe, non-blocking implementation of Deque. Unlike BlockingQueue, these collections never block producer or consumer threads. If an operation cannot be completed immediately (e.g., polling from an empty queue), it simply returns a special value such as null.

-> These collections are lock-free and do not use synchronized blocks or ReentrantLock for normal operations. Instead, they rely on CAS (Compare-And-Swap) and volatile fields to synchronize concurrent access. Multiple threads can attempt to modify the queue simultaneously; if one thread's CAS operation fails because another thread updated the queue first, it simply retries until it succeeds. This is known as optimistic concurrency.

-> Since operations are performed using CAS instead of locks, no thread enters the BLOCKED state waiting for a lock. The collection is lock-free, meaning the system as a whole always makes progress even if some individual threads have to retry due to CAS failures.

-> ConcurrentLinkedQueue is implemented as a singly linked list with head and tail references, whereas ConcurrentLinkedDeque uses a doubly linked list with prev and next references. Nodes are linked or unlinked atomically using CAS operations.

-> Most read operations (peek(), poll(), iterator(), etc.) do not require locking. They simply read volatile references, ensuring visibility of updates made by other threads.

-> Iterators are weakly consistent. They never throw ConcurrentModificationException and do not traverse a fixed snapshot. While iterating, they may reflect some, all, or none of the modifications made after the iterator was created, but they always traverse the collection safely without locking.

-> The size() method is O(n) because the queue does not maintain an atomic size counter. It traverses the linked list to count elements, and since other threads may modify the queue concurrently, the returned size is only an approximate snapshot. Therefore, size() should not be used to determine whether operations like poll() will succeed.

-> Although CAS is powerful, it is most suitable for operations involving one or a few atomic pointer updates (such as linking or unlinking nodes). Data structures like ConcurrentHashMap still use a combination of CAS and fine-grained locking, because operations such as updating an existing bucket, treeification, or resizing require multiple related updates to remain atomic and consistent, making a purely lock-free implementation much more complex.