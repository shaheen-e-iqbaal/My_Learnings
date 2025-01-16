

->GC operations are very heavy. Whenever the GC thread starts work, all other application threads stops working. that's why it is called world is stop.

1. Algorithms of GC.
     1.  Mark and Sweep
     2. Mark, Sweep and compact.

2. Types of GC and current and previous default GC of JVM.
     1. Serial ->only one thread is there for GC. when that thread starts, application threads stops.

     2. Parallel ->there are more than one thread and they do GC operations on their own. so there is less times when application stops. so it is better then serial GC.
     -> For Java <= 8, Parallel GC is being used as default.
		
     3. Concurrent Mark and Sweep -> jvm tries to minimize the stop of application threads when GC threads are running. but it doesn't 100% guarantee that application threads will keep running while GC operations. Memory compaction also doesn't happen after GC operation.

     4. G1 -> it is improved version of CMS and it also does memory compaction after GC operation.
     -> for java > 9, G1 is being used as default.

3. How to call GC. and what's the difference between them.
   -> We can call static gc() method of System class as System.gc(). but it doesn't guarantee that jvm will for sure run GC. it totally depends on JVM to when to run it.


4. Finalize method and it's signature.
   -> It is a "protected" method defined in Object class. it is called by the jvm when the object is about to deleted from heap.
   -> it is not a good idea to override it.
   -> Syntax: protected void finalize () throws Throwable{}

