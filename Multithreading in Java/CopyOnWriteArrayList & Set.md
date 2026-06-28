
It is a thread-safe version of ArrayList. CopyOnWriteArraySet is internally backed by a CopyOnWriteArrayList, so both have almost identical behavior and concurrency characteristics.

**How does it work?**

-> Read operations do not require any locking. Multiple readers can access the collection concurrently, and readers never block writers or other readers. This is possible because readers always access an immutable snapshot of the underlying array.

-> All write operations (add(), remove(), set(), clear(), etc.) acquire a single `ReentrantLock`. Therefore, at any given time, only one thread can perform a write operation.

-> Every write operation creates a new copy of the underlying array (shallow copy, not deep copy). The modification is performed on this new array, and once the operation completes, the reference to the backing array is atomically replaced with the new one. The previously published array is never modified, allowing existing readers to continue reading it safely without synchronization.

`private transient volatile Object[] array;

The backing array reference is declared as volatile, ensuring that once a writer replaces it with a new array, the updated reference becomes immediately visible to all threads.

-> Since only the array of references is copied (and not the actual objects), the copy operation is a shallow copy. The objects stored inside the collection remain shared between the old and new arrays.

-> Because an entirely new array is created on every write, both the time complexity and additional space complexity of write operations are O(n). For example, if the list contains 1 million elements, every write operation allocates a new array capable of holding 1 million references, making writes expensive in terms of both CPU and memory.

-> Therefore, CopyOnWriteArrayList and CopyOnWriteArraySet should only be used when write operations are very rare, while read operations are extremely frequent.

**Traversal**

-> When an iterator is created, it captures the current backing array reference (snapshot). Even if another thread modifies the collection afterwards, a new backing array is created and published, while the iterator continues traversing the old snapshot. Therefore, the iterator never sees modifications made after its creation.

-> Since the iterator traverses a snapshot that never changes, it never throws `ConcurrentModificationException`.

-> The iterator is read-only. Calling methods such as remove(), add(), or set() on the iterator throws `UnsupportedOperationException`, because the iterator cannot modify either its snapshot or the current collection.

I especially like one sentence because it captures the entire design philosophy:

Readers never synchronize with writers. Writers create a new backing array and atomically publish it, while existing readers continue traversing the old immutable snapshot.