
-> is checks that the both object refers to the same memory location or not.

-> == first checks whether both items are of the same datatype or not.
then it will call ______eq_____ method of the first object.

-> if the ______eq_____ method is not overridden, then it will use "is" to compare which will ultimately check for memory location equality.