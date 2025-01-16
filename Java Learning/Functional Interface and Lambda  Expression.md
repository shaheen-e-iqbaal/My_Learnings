
Functional Interface : An interface, which has only one abstract method is called FI.

-> Functional Interface can have default, static, private.

-> FI can additionally have Object class methods as abstract methods also.(equals, hashcode, toString). because the implementing class will get the implementation of this methods from Object class. so it will only have to implement one abstract method.

There are mainly four types of FI.
1. Consumer: it takes one argument and doesn't return any value.
2. Supplier: it doesn't take any value but returns something.
3. Function: it takes one argument and returns something also. ex. compareTo method.
4. Predicate: it takes one argument and returns Boolean always. ex. filter method.