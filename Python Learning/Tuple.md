
-> Tuple are immutable data structure in python. so it can be used as a key of dictionary.

-> ex:
	tp = 1, "apple", True
	print(type(tp))
	output : tuple

ex: 
	tp = ('apple')
	print(type(tp))
	output : string
-> so to make tuple only containing one element, use below way:
	tp = ('apple',)
	print(type(tp))
	output : tuple


-> Use tuple instead of list if you can because tuple are faster and requires small memory than list to store the same data.


Methods of Tuple:

1. index(val) -> It returns the index at which value = val is present. if no such val is present, it will raise error.
2. count(val) -> returns the number of times a value = val is present in tuple.