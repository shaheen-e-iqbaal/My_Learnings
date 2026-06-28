
**Traversing Collections**

There are multiple ways to traverse Java collections.

1. Using a normal for-loop (index-based traversal)

This is possible only for collections that support index-based access, such as ArrayList and LinkedList.

`for (int i = 0; i < list.size(); i++) {`
    `System.out.println(list.get(i));`
`}`

This approach does not use an Iterator internally. It simply accesses elements by their index on every iteration.

2. Using an enhanced for-loop

`for (String s : list) {`
    `System.out.println(s);`
`}`

3. Using an Iterator / `ListIterator`

`Iterator<String> iterator = list.iterator();`

`while (iterator.hasNext()) {`
    `System.out.println(iterator.next());`
`}`

For lists, we can also use `ListIterator`, which provides additional operations such as previous(), add(), and set().

The enhanced for-loop (method 2) is only syntactic sugar. Behind the scenes, the compiler converts it into an Iterator-based traversal (method 3). Therefore, both behave almost identically with respect to concurrent modification and fail-fast behavior.

---

**Structural Modification**

While traversing a collection, the underlying collection may be modified either by the same thread or by another thread.

There are two kinds of modifications:

1. Structural modification

A structural modification changes the size or internal structure of the collection.

These modifications can invalidate an ongoing traversal.

2. Non-structural modification

A non-structural modification updates existing data without changing the collection's size or structure.

Example:

`list.set(2, "Hello");`

or

`map.put(existingKey, newValue);`

These operations generally do not invalidate an iterator because the collection's structure remains unchanged.

--- 

**How does Java detect structural modification?**

Most mutable collections in the java.util package maintain an internal integer called `modCount`.

Every structural modification increments this value.

When an Iterator (or enhanced for-loop) is created, it stores the current value of `modCount` in another variable called `expectedModCount`.

On every call to next() (and some other iterator operations), it compares `expectedModCount` with actual `modCount`. If both values are equal, traversal continues normally.

If they differ, the iterator assumes that the collection has been structurally modified outside of itself and throws a
`ConcurrentModificationException`. This mechanism is called fail-fast.

Important: Fail-fast is a best-effort debugging mechanism, not a thread-safety guarantee. In a true multi-threaded environment without proper synchronization, a `ConcurrentModificationException` is not guaranteed to be thrown.

What if the Iterator itself modifies the collection?

An Iterator is allowed to remove elements using its own remove() method.

Similarly, a ListIterator can safely perform add(), remove(), and set() operations.

This works because after performing the modification, the iterator updates its own expectedModCount to match the collection's new modCount.

Therefore, both values remain synchronized, and traversal continues safely.

However, if the collection is modified directly (for example, by calling list.remove() instead of iterator.remove()), the iterator's expectedModCount is not updated, resulting in a ConcurrentModificationException.

---

**Collections in the java.util Package**

Collections such as

ArrayList
LinkedList
HashMap
HashSet
TreeMap
TreeSet

provide fail-fast iterators.

If these collections are structurally modified outside the iterator during traversal, the iterator will usually detect the modification and throw a ConcurrentModificationException.

--- 

Collections Returned by Collections.synchronizedXXX()

Methods such as

Collections.synchronizedList(...)
Collections.synchronizedMap(...)
Collections.synchronizedSet(...)

return synchronized wrappers around the original collection.

Every individual method (add(), remove(), get(), etc.) is synchronized, making each method call thread-safe.

However, an entire traversal is not a single method call.

Similarly, an index-based loop repeatedly calls methods like size() and get().

Since the lock is acquired and released for each individual method call, another thread can modify the collection between two iterator operations or between two get() calls.

Therefore, when traversing a synchronized collection, the entire traversal must be enclosed within a synchronized block:

`synchronized (syncList) {`
    `Iterator<String> iterator = syncList.iterator();`

    while (iterator.hasNext()) {`
        `System.out.println(iterator.next());`
    `}
`}`

Holding the lock for the entire traversal prevents other threads from modifying the collection until iteration completes.

Without this external synchronization, the iterator remains fail-fast, just like the underlying collection, and a structural modification by another thread may result in a ConcurrentModificationException.

---

