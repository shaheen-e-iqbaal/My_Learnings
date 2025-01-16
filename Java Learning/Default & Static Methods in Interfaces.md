

1.  Why does the java allowed Static and Default Method inside Interfaces from Java 8 ?

    > Before java 8 there were only abstract method allowed inside interfaces. so if we want to add any new method (obviously abstract method bcz they are allowed only till that time) inside interface, we must also have to change all the classes which have implemented this interface. so to save from this problem, they allowed the static and default methods also. since this method have body, we doesn't need to change all the implementing classes.


2. Static methods of Interface can't be inherited but of class can be inherited.

3. default methods of interfaces can be inherited.

4. suppose there are more  than one interface with same signature of default method. if any class/interface wants to implement/extend these interfaces, then it leads to a diamond problem, bcz jvm won't be able to decide which default method it has to use. to solve this problem, java made a rule that, you must have to override that default method. whether you can provide your own implementation of just use the implementation of any one interface like Animal.super.bark(); here Animal is a name of interface whose default method implementation we want to use and super is keyword and bark is default method name.

5. All Variables inside interface are by default public, static and final.


6. Every Default, static and abstract methods of interface are implicitly have public access modifier.

7. In Java 9, We can make private method inside interface also. these methods must have implementation. They can be static and non static.