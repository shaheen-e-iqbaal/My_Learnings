
1. Strong Reference:

   When we make object of a class, with syntax like, Person person = new Person();
   This type of referencing is called Strong reference.

   Until the object has Strong Reference, then it can't be collected by GC no matter what happens.



2. Weak Reference:

   Syntax:  
   //We have to write this import statement.
   import java.lang.ref.WeakReference;

   WeakReference<Person> weakref = new WeakReference<>(new Person());
   Person person = weakref.get();


   Using above code, we can make Person Object with Weak Reference.

   So, the object referenced by only weak reference is available on heap until GC has not run. once GC runs, it will delete all the objects with only weak references.
   If any object has both strong and weak reference then that object will be considered as with Strong reference.
   If the object is deleted by GC, then weakref.get() will give null. 

3. Soft Reference:

    Syntax will be same as Weak reference but instead of weak replace soft.

    The GC will delete the object with soft reference only if GC finds out that, it will run out of memory if it doesn't delete soft reference objects, Otherwise it will keep the object in heap.
 