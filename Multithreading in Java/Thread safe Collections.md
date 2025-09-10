
-> There are two ways to make java collections thread safe. 1) using Collections.synchronizedXxx() method. 2) using implementations like CopyOnWriteArrayList for list, CopyOnWriteArraySet for set, ConcurrentHashMap for hashmap...

-> In first way, we have methods like synchronizedList, synchronizedSet, synchronizedSortedSet for TreeSet, synchronizedMap for hashmap, synchronizedSortedMap for TreeMap and synchronizedCollection for any collection. all these methods are present in Collections utility class so they are all static methods. the return type of these methods are normal List, Map and Set.

-> when you do like :
```
List<Integer> synchronizedList = Collections.synchronizedList(someList);
```
it just wraps all the methods like get, set, toString... etc. inside Synchronize block. this block will use the monitor lock of same list object i.e. synchronizedList. so, it makes all operations like read, update, insert... atomic.
-> this is one of the most basic solution to achieve thread safety. but in this way, as the entire list gets blocked for any read, write operations, it has very low concurrency.

-> When you want to **traverse a synchronized collection** (like a list made using `Collections.synchronizedList()`), you need to **manually use a synchronized block** with the same list as the lock. This is because **traversing a list isn’t a single operation**—it involves multiple method calls like `get()` in a loop. If you don't use a synchronized block, each `get()` call will be individually thread-safe, but **between those calls**, another thread could modify the list. This can cause problems like a `ConcurrentModificationException`, because the list may change while you're still iterating over it.
ex :
```
synchronized (synchronizedList) {
    for (int i : synchronizedList) {
        System.out.println(s);
    }
}
```
so in above example, you are using monitor lock of synchronizedList, so no other thread can get the same lock until you release it. which will be released after iteration completes.


CopyOnWriteArrayList :
CopyOnWriteArraySet :
LinkedBlockingQueue :
ArrayBlockingQueue :
PriorityBlockingQueue :
ConcurrentHashMap :
HashTable :

ConcurrentHashMap :

-> It doesn't put lock on whole map for read/write operations like HashTable or SynchronizedMap.
-> Locks are implemented at bucket levels.
-> Read operations doesn't require any lock.
-> Write operations require a lock. writing thread will acquire the lock of first node of the bucket. so no other thread can go inside that bucket for write operation.
-> so multiple write operations can occur at the same time in a map but at different bucket.
-> writing thread requires to get monitor lock on the first node of the bucket.
-> If writing is happening first time on any
-> multiple read and single write operation can occur on same bucket as read operation doesn't require any lock.
-> operations like traversing a map doesn't required to put any external lock. if during traversal, some other thread modifies map, you won't get `ConcurrentModificationException` exception. the modification might be seen or not during your traversal.
-> it doesn't allow null as either key or value.

-> Difference between SynchronizedMap and HashTable: all methods of HashTable are like public synchronized get()..., but in case of SynchronizedMap, it uses the same method of HashMap, but it wraps that method inside synchronized block like Synchronize(mutex){
hashMap.get();
}
-> HashTable was legacy class, while SynchronizedMap is recent one introduced as an alternative to HashTable. SynchronizedMap is bit faster than HashTable.

