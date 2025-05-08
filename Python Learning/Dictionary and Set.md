
-> The key of dictionary and values of set must be hashable.
-> it must have hash method in it.
-> custom classes inherits this hash method from object class which returns id of object.
-> so custom python classes are by default hashable.

-> all immutable python data type are also hashable. ex int, str, tuple etc.
-> only list and dictionary are not hashable because they are mutable data type.

-> so, list and dictionary can't be used as a key of dictionary in python because they are mutable, so python does not provide hash method implementation for it.

-> It is highly recommended to use immutable objects as a key of dict.

-> insertion order of keys are maintained from python 3.6


Methods of dictionary:

keys() : returns a list containing list of keys
values() : returns list containing values
items() : returns a list containing tuples where each tuple is key, pair 
pop(Key) : removes the key  = Key from the dictionary.
popitem() : removes the last inserted key
get(key, default_value) : returns the value corresponding to Key = key if it is present in dictionary. if not, it will return default_value.
setdefault(key, value) : returns the value of Key = key if it is present, if not, then it will add Key = key with Value = value.
copy() : returns the shallow copy of dictionary.


### **How to Initialize a Set in Python**


`# Using curly braces {} 
`my_set = {1, 2, 3}  
`# Using the set() constructor 
`empty_set = set()  ``# Correct way to create an empty set 
`another_set = set([1, 2, 3])  # From a list 
`string_set = set("hello")  # {'h', 'e', 'l', 'o'} (unordered, unique characters)`


Methods of set :

add(v) : adds the v into set
remove(v) : removes v from the set if it is present. if it is not, then it will raise KeyError.
discard(v) : same as remove but won't raise any error if v is not present.
pop() : removes and returns any arbitrary value from set. if set is empty, it will raise KeyError.
copy() : returns the shallow copy of set.
