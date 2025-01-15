

1. Types of Memory allocated by JVM.
	 1. Stack
	     -> It stores local variables like primitive and reference.
	     -> Each thread has it's own Stack.
	 2. Heap
	     -> It stores all the instance variables (primitive + reference).
	     -> Divided in two parts. young, old generation
	     -> Young generation is also divided in three parts. Eden, S0, S1.
	     -> young generation is small compared to old generation.
	     -> GC operation in young generation(knows as minor GC) happens more frequently than in old generation.
	 1. Metaspace
	     -> It stores information related to class like static variables etc.
	     -> it is also garbage collectible.
	     -> it is dynamic in size i.e. will increase it's size if required.
	     -> it is not part of heap.
	 4. PC (Program counter)
	     ->Each thread has it's own PC.
	     -> It stores the address of the instructions currently being executed.


2. Difference between permanent generation (PremGen) and metaspace.
   Permanent Generation : 
   ->It is available before java 8.
   ->It is static in nature i.e. it is assigned fix size memory by jvm.
   -> Once a class is loaded in premgen, it remains inside it until jvm is running.

   Metaspace:
   -> it is included in java 8.
   -> it is allocated memory dynamically i.e. JVM will increase if needed.
   -> jvm loads / unloads class from metaspace as per requirements.

3. Which types of memories are Garbage Collectable.
 ->  Heap and Metaspace memory are garbage collectable.