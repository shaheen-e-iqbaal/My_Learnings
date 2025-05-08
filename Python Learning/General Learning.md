
-> Input given from terminal are by default string. so if we give int, then also it will be treated as string. so we have to convert string to int using int() function.

-> to convert int to string : str(a)

-> to convert string to int : int(s)

->print() function automatically gets to new line. so to print in the same line :

		print("hello", end = ' ')
		print('World')

	output : hello World


----------------------------------<<<<<<<<<<<<<<>>>>>>>>>>>>>----------------------


Functions in Python :

-> Functions in python are objects, meaning they can be assigned to a variable, passed as a parameter to any other function, returned by some function, can be created inside another function.



----------------------------------<<<<<<<<<<<<<<>>>>>>>>>>>>>----------------------



HTTPX vs request :

-> Both of the above libraries of python are used to make HTTP request to any URL.

-> HTTPX supports asynchronous way, while request only support synchronous way.



----------------------------------<<<<<<<<<<<<<<>>>>>>>>>>>>>----------------------



Difference between package and directory in python

-> Python package is used when we want to group multiple modules together. we can do it using directory also, but it is not good way.

-> we can add some default code into ____init____.py file of python package which will run whenever our package is imported. in case of python directory, we can't run this default code, due the unavailability of ____init____.py file.


----------------------------------<<<<<<<<<<<<<<>>>>>>>>>>>>>----------------------


assert key word :

Syntax : assert condition, error_message

ex : 
`a` `=` `4`

`b` `=` `0`

`# using assert to check for 0`

`print("The value of a / b is : ")

`assert b !=` `0, "Zero Division Error"`

`print(a` `/` `b)`

Output : AssertionError: Zero Division Error

if the condition evaluates to true, then the program flow will keep going, if it is false, then it stops the program execution and gives AssertionError.


----------------------------------<<<<<<<<<<<<<<>>>>>>>>>>>>>----------------------