
-> There are two ways to make java collections thread safe. 1) using Collections.synchronizedXxx() method. 2) using collections from `java.util.concurrent` package. 
ex :  `CopyOnWriteArrayList` for list, `CopyOnWriteArraySet` for set, `ConcurrentHashMap` for hash map...

-> In first way, we have methods like synchronizedList, synchronizedSet, synchronizedSortedSet for TreeSet, synchronizedMap for hashmap, synchronizedSortedMap for TreeMap and synchronizedCollection for any collection. all these methods are present in Collections utility class so they are all static methods. the return type of these methods are normal List, Map and Set.

-> when you do like :
```
List<Integer> synchronizedList = Collections.synchronizedList(someList);
```
it just wraps all the methods like get, set, toString... etc. inside Synchronize block. this block will use the monitor lock of same list object i.e. synchronizedList. so, it makes all operations like read, update, insert... atomic.
-> this is one of the most basic solution to achieve thread safety. but in this way, as the entire list gets blocked for any read, write operations, it has very low concurrency.

-> When you want to **traverse a synchronized collection** (like a list made using `Collections.synchronizedList()`), you need to **manually use a synchronized block** with the same list as the lock. This is because **traversing a list isn’t a single operation**—it involves multiple method calls like `get()` in a loop. If you don't use a synchronized block, each `get()` call will be individually thread-safe, but **between those calls**, another thread could modify the list. This can cause problem of fail-fast and throw  `ConcurrentModificationException`, because the list may change while you're still iterating over it.
ex :
```
synchronized (synchronizedList) {
    for (int i : synchronizedList) {
        System.out.println(s);
    }
}
```
so in above example, you are using monitor lock of synchronizedList, so no other thread can get the same lock until you release it. which will be released after iteration completes.


ConcurrentHashMap :

-> It doesn't put lock on whole map for read/write operations like HashTable or SynchronizedMap.
-> Locks are implemented at bucket levels.
-> Read operations doesn't require any lock. the reason for this is, the value in key->value pair is kept volatile. so each read will always get the latest value.below is the structure of Node class :
```
class Node{
	final K key;
	final int hashCode;
	volatile T value;
	volatile Node next;
}
```
Notice value and next both are volatile. so when ever there is some change in them, it will be immediately visible to all threads. so in concurrent environment, due to volatile, we will always get their latest value.

-> Write operations require a lock. writing thread will acquire the lock of first node of the bucket. so no other thread can go inside that bucket for write operation.
-> so multiple write operations can occur at the same time in a map but at different bucket.
-> writing thread requires to get monitor lock on the first node of the bucket if it's LinkedList. if the bucket is converted into tree (Red-black tree), then it is possible that root of the tree keep changing. so you can't use root's monitor lock. for this case, it will insert one separate node called `Treebin` and will use this node's monitor. this `Treebin` won't have any key->value pair and won't be part of the created tree.
-> If writing is happening first time on any bucket, i.e. first node is being inserted in bucket, it will be done using CAS.
-> multiple read and single write operation can occur on same bucket as read operation doesn't require any lock.
-> operations like traversing a map doesn't required to put any external lock to avoid fail-fast behavior. if during traversal, some other thread modifies map, you won't get `ConcurrentModificationException` exception. the modification might be seen or not during your traversal.

-> We should use methods like `computeIfAbsent`, `computeIfPresent`  instead of doing like first check if key is there or not. if not, then put. this doesn't work in atomic way. these are two separate functions. so it is possible that, after your check, some other thread inserts that key which you got that doesn't exist. above mentioned methods does this checking, insertion operation as atomic.

**Why Null is not allowed as Key or value?**
-> it doesn't allow null as either key or value. why doesn't it allows null values? assume it allows null values. then when you do .get(key) and it returns null, then how will you surely know that is the value of key is null or key itself is not present. you can say, we will use contains(key) method, but what if after you checked contains(key), it returned true. and then some other thread came and deleted key. now when you do get(key), you will get null. you will think that value of key is null. but actually the key itself is no longer available. so this led to false read. due to this, null values are not allowed.
why null keys are not allowed? concurrent hash map uses null as the indicator that the key is not present. when we use methods like `computeIfAbsent`, it will check if the get for the key returns null or not. if null, then proceeds with computation and insertion. so if we allow map entry with null key, then these methods will have ambiguity.

**How does Size(), Empty() methods behave differently?**
-> result of methods like size(), isEmpty() aren't concrete one because they don't apply any global lock to compute result. they give estimates only when multiple threads are concurrently updating map. so, it is highly possible that, the actual size of map and size returned by size() function are different. so, we should use these methods for monitoring purpose only. shouldn't use them for control purpose like running loop until map is not empty....

-> Difference between SynchronizedMap and HashTable: all methods of HashTable are like public synchronized get()..., but in case of SynchronizedMap, it uses the same method of HashMap, but it wraps that method inside synchronized block like Synchronize(mutex){
hashMap.get();
}
-> HashTable was legacy class, while SynchronizedMap is recent one introduced as an alternative to HashTable. SynchronizedMap is bit faster than HashTable.

-> Use this [article](https://javaconceptoftheday.com/how-concurrenthashmap-works-internally-after-java-8/) to learn about internal working of concurrent hash map.

--- 

Summary about each Concurrent Collection : 

| Collection              | Iterator                                                                                  | `size()`                                                      | `isEmpty()`                                                               | `null` Allowed?            | `size()` TC | `isEmpty()` TC |
| ----------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------- | -------------------------- | ----------: | -------------: |
| `CopyOnWriteArrayList`  | **Snapshot** (never reflects later changes, never throws CME)                             | Exact                                                         | Exact                                                                     | ✅ Yes                      |        O(1) |           O(1) |
| `CopyOnWriteArraySet`   | **Snapshot**                                                                              | Exact                                                         | Exact                                                                     | ✅ Yes                      |        O(1) |           O(1) |
| `ConcurrentHashMap`     | **Weakly Consistent** (may reflect some/all/none of concurrent changes, never throws CME) | Approximate under concurrent updates                          | Approximate under concurrent updates                                      | ❌ Keys: No<br>❌ Values: No |        O(1) |           O(1) |
| `ConcurrentSkipListMap` | **Weakly Consistent**                                                                     | Traverses/counts (may be inaccurate under concurrent updates) | Approximate under concurrent updates                                      | ❌ Keys: No<br>❌ Values: No |        O(n) |           O(1) |
| `ConcurrentSkipListSet` | **Weakly Consistent**                                                                     | Traverses/counts (may be inaccurate under concurrent updates) | Approximate under concurrent updates                                      | ❌ No                       |        O(n) |           O(1) |
| `ConcurrentLinkedQueue` | **Weakly Consistent**                                                                     | Traverses/counts (may be inaccurate under concurrent updates) | Approximate under concurrent updates                                      | ❌ No                       |        O(n) |           O(1) |
| `ConcurrentLinkedDeque` | **Weakly Consistent**                                                                     | Traverses/counts (may be inaccurate under concurrent updates) | Approximate under concurrent updates                                      | ❌ No                       |        O(n) |           O(1) |
| `LinkedBlockingQueue`   | **Weakly Consistent**                                                                     | Exact (maintains atomic count)                                | Exact                                                                     | ❌ No                       |        O(1) |           O(1) |
| `ArrayBlockingQueue`    | **Weakly Consistent**                                                                     | Exact (maintains count)                                       | Exact                                                                     | ❌ No                       |        O(1) |           O(1) |
| `PriorityBlockingQueue` | **Weakly Consistent**                                                                     | Exact                                                         | Exact                                                                     | ❌ No                       |        O(1) |           O(1) |
| `DelayQueue`            | **Weakly Consistent**                                                                     | Exact                                                         | Exact                                                                     | ❌ No                       |        O(1) |           O(1) |
| `LinkedBlockingDeque`   | **Weakly Consistent**                                                                     | Exact (maintains atomic count)                                | Exact                                                                     | ❌ No                       |        O(1) |           O(1) |
| `SynchronousQueue`      | **Empty iterator** (always empty)                                                         | Always `0`                                                    | Depends on waiting handoff; practically always `true` for stored elements | ❌ No                       |        O(1) |           O(1) |
| `LinkedTransferQueue`   | **Weakly Consistent**                                                                     | Traverses/counts (may be inaccurate under concurrent updates) | Approximate under concurrent updates                                      | ❌ No                       |        O(n) |           O(1) |


| Iterator Type         | Used By                                                                                                             | Concurrent Modification                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Snapshot**          | `CopyOnWriteArrayList`, `CopyOnWriteArraySet`                                                                       | Traverses a fixed snapshot. Never sees later modifications. Never throws `ConcurrentModificationException`.     |
| **Weakly Consistent** | `ConcurrentHashMap`, `ConcurrentSkipListMap/Set`, all concurrent queues/deques, all `BlockingQueue` implementations | May reflect **some, all, or none** of concurrent modifications. Never throws `ConcurrentModificationException`. |
| **Empty Iterator**    | `SynchronousQueue`                                                                                                  | Always empty because the queue never stores elements.                                                           |
