
-> There are some features of hardware like instruction-ordering, memory cache (registers, L1, L2, L3 caches) which can cause a problem for software developers while writing concurrent code. these features are included at hardware level (CPU, RAM) to increase the speed of program execution.

-> To avoid the bugs due to these features (i.e. Instruction reordering, Memory caches), JVM has to provide some specifications which help us to overcome them. JVM provides this specification with JMM i.e. Java Memory Model. so JMM specifies how threads interact with memory, how re-ordering should be done so it doesn't affect final output.

-> We know that each Java apps have centralized Heap memory shared across all threads and stack memory which is explicit to particular thread. both these Heap and Stack memory are backed by main memory.

//First Feature

-> We also know that, each CPU/core has it's own register, L1 and L2 cashes. L3 cache is shared between multiple cores of processor. now, when any thread is running on any core and it wants to read value corresponding to some variable, then it will first check in register of that core, if not found then in L1, then in L2 then in L3 and only than in RAM/Main memory. similarly, When a CPU core modifies a value, the write may remain temporarily in registers, store buffers, or cache levels before becoming visible to other cores. 

-> Now, the problem we face with this CPU working during concurrency is that, when some resource is shared across multiple thread, and these threads are running parallel on multiple cores and also updating this shared resource, then is not guaranteed that other thread will be able to see the latest value of that shared resource. because it is high possibility that each CPU core might be reading/writing from it's own cache. this is called ***==visibility problem of concurrency==***. 

//Second Feature

-> We also know that, JIT or CPU reorders the instructions to optimize the program performance. ex : below are our instructions.

```
firsValue = 5; //instruction 1
secondValue = 3;  //instruction 2

firstValue += 3; //instruction 3

thirdValue = first + second; //instruction 4
```

now, we can clearly see that, instruction 1 and 2 can't be optimized. but instruction 3 can be optimized by merging it with instruction 1. if we merge it then we will have one less instruction to run separately. so JIT or CPU optimizer can combine instruction 1 and 3 before running them. in this case, we won't have any issue with our program result. but for below example :

```
public class {

	boolean canProceed = false;
	int score = 0;
	
	public void decreaseScore(){
		while(!canProceed){
			//wait until score is available.
			continue;
		}
		score--;
	}
	
	public void increaseScore(){
		score++;
		canProceed = true;
	}
}
```

now, assume JIT or CPU optimizer reorders the instructions of method `increaseScore` as below : 

```
	public void increaseScore(){
		canProceed = true;
		score++;
	}
```

-> Now, imagine the scenario of two threads. one job is to increase score and other is to reduce score. with re-ordered instructions, when thread B marks the boolean field to true and thread A reads it and comes out of while loop and then decreases the score even though thread B hasn't yet increased the score. this will led to malfunction of our application. notice that, we have written the code correct, but due to JIT or CPU optimizer, our code got re-ordered and resulted in malfunction. To solve this issue, we need ordering and visibility guarantees so that once another thread observes `canProceed=true`, it is also guaranteed to observe the updated score value.


Every concurrency bug can generally be reduced to one or more of the following:

1. Visibility
   - One thread's update is not visible to another thread.

1. Atomicity
   - Multiple threads interleave operations and corrupt shared state.

3. Ordering
   - Compiler, JIT or CPU reorders operations and another thread observes an unexpected order.

Java solves these through synchronization mechanisms such as:

- volatile Provides visibility and ordering guarantees.

- synchronized Provides visibility, ordering and atomicity guarantees.

The Java Memory Model defines these guarantees through the happens-before relationship.

If action A happens-before action B, then:
1. A is guaranteed to occur before B.
2. The effects of A are guaranteed to be visible to B.

Refer [Jankov's](https://jenkov.com/tutorials/java-concurrency/java-memory-model.html) Java memory model, Java happens before,  Java synchronized blocks, Java volatile keyword section