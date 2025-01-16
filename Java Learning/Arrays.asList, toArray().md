

Arrays.asList(arr) : 

1. Returns fixed size, modifiable list backed by arr.
2. arr can't be of primitive type. 
3. we can use primitive as List<Integer> ans = Arrays.asList(1,2,3,4); but can't use     like int[] arr = {1,2,3}; List<Integer> ans = Arrays.asList(arr);
   4. set, sort, contains operations are allowed but add, remove are not allowed. i.e. we can modify the elements but can't change the size of the list.
   5. returned list is backed by arr i.e. changes made in either of arr or list will be visible in both.



toArray() :

1. converts the given collection (set, queue, list) into array.
2. it returns Object[].


toArray(arr) :

			List<Pair> pairs = new ArrayList<>();
	Signature : Pair[] ans = pairs.toArray(ans);

1. it will copy the elements of collection into given array. datatype of pairs and ans are same.
2. if the arr size is smaller then collection, then it will be resized to size of collection and if the size is larger, then it will remain so.
3. the returned array will be of the same type of collection.
4. return array will be free from collection i.e. changes made in collection or array will not be visible in other.
