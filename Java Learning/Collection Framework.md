 
Hierarchy:       

Iterable
Collection
Queue, List, Set

all of the above are Interfaces.

QUEUE:

Concreate classes of Queue : PriorityQueue, LinkedList, ArrayDeque

Refer [This image](https://images.app.goo.gl/UiTeMrWPzTqPrntDA) to see hierarchy of Queue Interface.

ArrayDeque uses array and LinkedList uses Doubly linkedlist to implement Deque.

All this concreate classes are not synchronized means not thread safe.

Most Used methods of Queue interface which can be used in LinkedList and Priority Queue : 
1. peek() : to get the head of queue. if queue is empty, it will return null;
2. element() : same as peek() but will throw exception if queue is empty.
3. add() : to add element in tail of queue. will throw exception if queue is reached it's maximum size;
4. offer() : same as add().
5. poll() : to get and remove head of queue. will return null if queue is empty.
6. remove() : same as poll() but will throw exception if queue is empty.
7. size() : returns the size of the linkedlist.

	Using the below three methods, we can use the LinkedList as a Stack also :
	1. addLast() :
	2. pollLast() :
	3. getLast() :


==Constructor of Linkedlist and Priority Queue :== 

LinkedList(Collection c) : it will create a linkedlist collection c.

PriorityQueue(size) :
PriorityQueue(Comparator c) :
PriorityQueue(size, Comparator c) :
PriorityQueue(Collection c) : it will create priority queue from elements of c.


------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------


LIST:

Concreate classes implementing List: ArrayList, LinkedList, Vector, Stack.

Refer [This image](https://images.app.goo.gl/hD68jMwZVEqbsvRq5) to understand hierarchy of List Interface.

->>Notice that LinkedList implements both Queue and List Interfaces.

Some methods of ArrayList:
1. add(value) :
2. set(index, value) :
3. remove(index) :
4. size() : 
5. addLast(value) : same as push_back() of C++.
6. getLast() : same as back() of C++.
7. removeLast() : same as of pop_back() of C++.

==Java uses Array to implement the ArrayList.==


How to make ArrayList with some predefined values stored in it?

   There are three ways to do that :

1. Arrays.asList(array) : makes fixed size and modifiable list backed by array.
2. List.of(values) : makes fixed size list.
3. new ArrayList<>(Collection c) : makes arraylist from elements of c.

  Use [This Article](https://medium.com/@tecnicorabi/list-of-vs-arrays-aslist-the-hidden-differences-that-can-crash-your-code-02d7890a1d02) to know difference between list returned by method 1 and 2.


Constructor of ArrayList : 

1. ArrayList<>(capacity) : makes the arraylist with capacity of array = capacity.
2. ArrayList<>(Collection c) : makes arraylist from elements of c.

Difference between ArrayList and Vector :

1. Both are implemented using array, but size of the arraylist array will grow by 50% of current size, and for vector, it will grow by 100% of current size.
2. Vector is thread safe and arraylist is not.

------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------



SET:


Concreate classes implementing Set : HashSet, TreeSet, LinkedHashSet.

Refer [This image](https://images.app.goo.gl/edxUq6JtqhNX9P497) to understand hierarchy of Set Interface.

HashSet : 

HashSet uses HashMap internally with default load factor = 0.75

Methods of HashSet : 

 1. add() : O(1) in average case and O(n) in worst.
2. remove(target) : O(1) in average and O(N) in worst.
3. contains(target) : O(1) in average and O(n) in worst.
4. size() : O(1) every time.

Constructor :
1. HashSet(Collection c) :

TreeSet : 

TreeSet uses TreeMap internally.

Methods of TreeSet :

1. add() : O(logn) 
2. remove() : O(long)
3. contains() : O(logn)
4. size() : O(1)
5. first() :
6. last() :
7. pollFirst() :
8. pollLast() :
9. lower() :
10. higher() :
11. floor() :
12. ceiling() :

Constructor :
1. TreeSet(Comparator c) :
2. TreeSet(Collection c) :

TreeSet and TreeMap doesn't require to override hashcode and equals method. as they use compareTo or compare method to check for equality of two objects.


LinkedHashSet :

It uses LinkedHashMap internally.

Methods are same as HashSet with same time complexity.

LinkedHashSet maintains the insertion order while HashSet doesn't. this is main difference between them.


------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------


Map :

It does not Extend Collection Interface. 

Concreate classes of Map : HashTable, HashMap, TreeMap, LinkedHashMap

Refer [This Image](https://images.app.goo.gl/L4Ks7vZFNnh7K1pS8) to understand hierarchy of map interface.


HashMap Internals : 

-> one null key and multiple null values are allowed in HashMap.
-> It is not Synchronized.
-> Initial capacity of hashmap is 16 and initial load factor is 0.75.
-> whenever number of entries in any bucket gets more than 8, it gets converted into Balanced binary search tree like red black tree.
-> after every rehashing operation, the capacity / bucket size gets doubled.
-> the capacity of hashmap must always be power of two. refer [this article](https://stackoverflow.com/questions/53526790/why-are-hashmaps-implemented-using-powers-of-two#:~:text=Powers%20of%20two%20match%20the,occupy%202%5En%20full%20bytes.) to know it's reason.

Effect of Load factor on time and space requirements : 

	Load Factor = number of entries stored / capacity of map.
		so suppose load factor = 0.6 with initial capacity = 64 so,              0.6 * 64 = 38.so on adding the 39th entry, rehashing                     operation will be performed. after rehashing, the capacity               will get doubed i.e. 128.

=> Higher load factor means efficient memory usage but more time required for lookups.

=> Lower load factor means more empty buckets or more memory requirements and less lookup time.

so after testing various values, Java founder decided to keep default load factor as 0.75 which gives good efficiency in both time and space.


HashTable Internals : 

-> It doesn't allow null key or null value.
-> It is synchronized.
-> Bucket doesn't gets converted Into balanced BST like it occurs in HashMap.


LinkedHashMap Internals : 

-> It uses HashMap internally.
-> it maintains the insertion order. to do so, it also keeps two pointers called before and after in each entry.
-> rest of the things are similar of HashMap.



TreeMap Internals :

-> It is implemented using balanced BST like red black tree.
-> It doesn't require to override hashcode and equal methods like it required for hashmap. it uses compareTo or compare method to figure out whether two objects are equal or not and which one is smaller or higher.
-> It doesn't allow the null key but allow the null values.


Methods of TreeMap : 

1. put(k, v) : O(logn) 
2. remove(Key k) : O(long)
3. containsKey(Key k) : O(logn)
4. size() : O(1)
5. firstKey() : returns the smallest key of TreeMap in log(n) time.
6. lastKey() : returns the largest key in O(logn) time.
7. firstEntry() : returns smallest entry in O(logn) time. return type will be Map.Entry<K, V>.
8. lastEntry() : returns largest entry in O(logn) time. return type will be Map.Entry<K, V>.
9. pollFirstEntry() : O(logn)
10. pollLastEntry() : O(logn)
11. lowerKey(Key k) : returns the largest key less than k in O(logn) or null.
12. higherKey(Key k) : returns the smallest key greater than k in O(logn) or null.
13. floorKey(Key k) : returns the largest key less than or equal to k in O(logn) or null.
14. ceilingKey(Key k) : returns the smallest key greater than or equal to k in O(logn) or null.


ConcurrentHashMap Internals :

-> it has same implementation as of HashMap for collision resolution and treeify.
-> it doesn't allow null as either key or value.






------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------



Collection Vs Collections :

Collection : It is a part of collection framework. it is an Interface which is extended by various collection like queue, list, and set.

Collections : It is a utility class. it contains static methods like sort, reverse etc. it makes working with collection easy as it provides some frequently used methods.

Both Collection interface and Collections class are part of java.util package.


------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------


Collections.sort() :

It always (for both primitive and custom object) calls list.sort(). which in turn calls Arrays.sort(). which uses Merge sort or Tim sort algorithm.


Arrays.sort():

for primitive, it uses DualPivotQuicksort algorithm. it doesn't take custom comparator for primitive. so sorting will be always in natural order.

And for custom objects, it also use Merge or Tim sort algorithm.


------------------------<<<<<<<<<<<<<<>>>>>>>>>>-----------------------------
