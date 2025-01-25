
-> to sort a list in place, we can use  arr.sort() function. 

-> to sort any data structure, we can use sorted() function. this function won't make any changes in original data structure but will create a new one and return that.

-> we can do custom sorting by giving lambda function also.

ex: result = sorted(persons, key = lambda person : person.age, reverse = False)

above sorted function will sort the persons list using person.age in increasing order. if we make reverse = True then it will sort in decreasing order.