
in Java, every object has it's corresponding class "Class", which is created by the JVM when it was loaded by class loader.

-> It contains all the metadata information about this class.

-> class Class is a final class. which is present in java.lang.reflect package.

-> There are various ways to get instance of class Class. one of the easy is :

	suppose we have class Student. then the instance of class Class         corrosponding to Student will be :

			Class studentClass = Student.class;

-> Using studentClass, we can get information about all fields of Student class, constructors of Student class, methods, super class of Student class, interfaces implemented by Student class, etc..

-> we can even change the access modifier of any field, method, constructor from private to public.

-> due to reflection, Singleton class can breached as we can change it's constructor access modifier to public from private.


Common Methods of class Class : 

- **`String getName()`**: Returns the fully qualified name of the class.
- **`Package getPackage()`**: Returns the package of the class.
- **`Field[] getFields()`**: Returns public fields (including inherited ones).
- **`Field[] getDeclaredFields()`**: Returns all fields declared in the class including private field.
- **`Method[] getMethods()`**: Returns public methods (including inherited ones).
- **`Method[] getDeclaredMethods()`**: Returns all methods declared in the class including private methods.
- **`Constructor<?>[] getConstructors()`**: Returns public constructors.
- **`Constructor<?>[] getDeclaredConstructors()`**: Returns all constructors declared in the class including private.
- **`setAccessible(true)`** : sets the access modifier of private field, method, constructor to public.