

Advantages of thread pools :
-> We don't need to create a new thread each time, but we can reuse the threads from the pool which are already created. so thread creation time, overhead gets reduced.

-> We don't have to maintain the lifecycle of each thread.


1. ==ThreadPoolExecutor (implements ExecutorService interface) :==

maximum arguments Constructor : 
```
public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue,  
                          ThreadFactory threadFactory,  
                          RejectedExecutionHandler handler)
```

-> corePoolSize -> number of threads will be created by pool before storing the task in queue. this threads are known as core threads.

-> maximumPoolSize -> after the queue gets full, additional (maximumPoolSize - corePoolSize) threads can be created. these threads are known as non core threads.

-> keepAliveTime -> if the non-core thread is idle for this much time i.e. keepAliveTime, it will be terminated automatically no matter what the flag allowCoreThreadTimeOut is. but if the core thread is idle for this much time i.e. keepAliveTime and above flag is set to true, then only it will be terminated otherwise won't get terminated. the default value of flag i.e. allowCoreThreadTimeOut is false.

-> if all the threads are terminated, but the pool haven't shutdown yet, you can still submit the task. the pool will create the new threads in that case.

-> unit -> shows the unit of time like seconds, hours or minutes.

-> workQueue -> queue to store the task which are waiting for thread allocation. BlockingQueue is used because it is thread safe and also good for cases involving producing - consuming of tasks. ArrayBlockingQueue and LinkedBlockingQueue are two implementation of BlockingQueue interface. first one is bounded while second one can be unbounded. so always use first one.


What should be the value of corePoolSize should be?

-> It depends on various factors like # of CPU cores, memory size, task types.
-> if task to be performed by thread pool's threads are CPU intensive, then the value of corePoolSize should be around the number of CPU cores available in system. if it is IO intensives, then the value of corePoolSize should be decided based on memory allocated to JVM instance because in this case, threads won't require CPU time much but each thread will require memory. 


Flow of working of ThreadPoolExecutor :

-> when we do .execute(Runnable task) or .submit(Runnable / Callable task), it will first check total number of threads present in pool. if they are < corePoolSize, it will create one thread and will assign this task to that thread. if threads = corePoolSize, it will check the queue, weather the queue has any space to accommodate this task or not? if yes then will put it. otherwise, will check weather # threads in pool < maxPoolSize ? if yes, then will create a new thread will assign it this task. if # threads in pool == maxPoolSize, then will reject this task.



2. ==ForkJoinPool (implements ExecutorService interface) :==

-> It can be created in two ways :
	1. new ForkJoinPool(int parallelism) : parallelism means maximum number of threads can be created. it's value should be equal to # of cores - 1
	2. ForkJoinPool forkJoinPool = ForkJoinPool.commonPool() : it will create a fork join pool which will be singleton in whole JVM instance. so when you call the .commonPool() method again, it will return the same instance. pool created this way is static and non garbage collectible. pool size is by default # of cores
	   -1.

-> methods of ForkJoinPool :
	1. invoke(ForkJoinTask task) : it takes ForkJoinTask as parameter. if task implements RecursiveTask, then invoke will return something, or if it implements RecursiveAction, then invoke won't return anything.
	2. execute(ForkJoinTask task) : return type is void.
	3. execute(Runnable task) : return type is void.
	4. submit(ForkJoinTask task) : return type is ForkJoinTask
	5. submit(Callable task) : return type is ForkJoinTask
	6. submit(Runnable task) : return type is ForkJoinTask
	7. submit(Runnable task, T result) : return type is ForkJoinTask

-> ForkJoinTask is a concrete class which implements Future interface. so we can do like Future<"T"> result = forkJoinPool.submit(Callable/Runnable/ForkJoinTask)

-> When you create a `ForkJoinPool` with a size of 4, it means the pool has 4 worker threads available to execute tasks in parallel. Each thread has its own **double-ended queue (deque)** where it stores the small subtasks it creates.
When a thread picks up a large task, it checks whether the task can be split into smaller parts. If yes, it **"forks"** the task into two (or more) parts. The thread then pushes one part into the **end of its own deque** and continues working on the other part directly. This forking and splitting process continues recursively, breaking big tasks into smaller chunks.
Now, let’s say one of the worker threads finishes its own tasks and becomes idle. Instead of sitting idle, it first checks the **shared task queue** (if any), and if that’s empty, it tries to **steal a task** from the **front of another thread’s deque**. This behavior is called **work-stealing** and it helps balance the load across threads.
This way, the `ForkJoinPool` makes sure all available threads stay busy by dynamically splitting tasks and stealing unfinished work from others. It’s designed to use CPU resources efficiently, especially for tasks that can be broken into smaller independent units.

-> Use Case of ForkJoinPool : parallel stream execution in java utilizes the shared ForkJoinPool. when you have CPU intensive task and you can divide the parent task in more than one independent subtasks, then you should use ForkJoinPool.

ex : Task type is RecursiveTask 

```
public class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 4;
    private final int[] arr;
    private final int start, end;

    public SumTask(int[] arr, int start, int end) {
        this.arr = arr;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += arr[i];
            return sum;
        } else {
            int mid = (start + end) / 2;
            SumTask left = new SumTask(arr, start, mid);
            SumTask right = new SumTask(arr, mid, end);
            left.fork();
	        right.folk();
            long leftResult = left.join();         // wait for the forked
            long rightResult = right.join();   // wait for forked
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        int[] data = new int[10];
        for (int i = 0; i < data.length; i++) data[i] = i + 1;  // 1 to 10

        ForkJoinPool pool = new ForkJoinPool();
        long result = pool.invoke(new SumTask(data, 0, data.length));

        System.out.println("Sum: " + result);  // Output: Sum: 55
    }
} 
```

ex : Task  type is RecursiveAction :

```
public class PrintTask extends RecursiveAction {
    private static final int THRESHOLD = 3;
    private final int[] arr;
    private final int start, end;

    public PrintTask(int[] arr, int start, int end) {
        this.arr = arr;
        this.start = start;
        this.end = end;
    }

    @Override
    protected void compute() {
        if (end - start <= THRESHOLD) {
            for (int i = start; i < end; i++) {
                System.out.println("Value: " + arr[i]);
            }
        } else {
            int mid = (start + end) / 2;
            PrintTask left = new PrintTask(arr, start, mid);
            PrintTask right = new PrintTask(arr, mid, end);
            left.fork();
            right.fork();
        }
    }

    public static void main(String[] args) {
        int[] data = {10, 20, 30, 40, 50, 60};

        ForkJoinPool pool = new ForkJoinPool();
        pool.invoke(new PrintTask(data, 0, data.length));
    }
}
```


--------------------------<<<<<<<<<<<<>>>>>>>>>>>--------------------------



***Hierarchy of Executor :***


1. Executor (Interface) : 

Methods : execute(Runnable r)


2. ExecutorService (Interface) extends Executor :

Methods : 
1. submit(Runnable r) : return type is Future<"?">. due to Runnable, future.get() will return null when the task is completed.
2. submit(Runnable r, T result) : return type is Future<"T">. so future.get() will return T result.
3. submit(Callable<"T"> c) : return type is Future<"T">. 

-> Note that submit methods can take both Runnable and Callable, but will always return Future object.


3. ThreadPoolExecutor, ForkJoinPool (These two are concrete implementation of ExecutorService interface) :

-> overrides execute(), submit() methods.


--------------------------<<<<<<<<<<<<>>>>>>>>>>>>-----------------------


shutDown(), shutDownNow(), awaitTermination(time, TimeUnit) :

-> All above three methods are present in ExecutorService interface. so these methods are implemented by both ThreadPoolExecutor and ForkJoinPool.

shutDown() : when you call threadPool.shutDown() method, it start shutdowning the pool. no new task can be submitted to the pool. ==**all the currently running tasks and prior submitted task (which might be present in task queue) will get completed and after that pool will get shut.**== 

shutDownNow() : when you call threadPool.shutDownNow(), it will not accept any new task. ==**it will also try to interrupt/stop all the threads present in pool. it will also clear the task Queue. so we can say that thread pool gets shut abruptly.**==

-> these both methods are non blocking i.e. caller thread (thread which calls this method/s) will not get blocked until the pool gets shut.

awaitTermination(time, TimeUnit) :

-> to make the caller thread wait for sometime, you call use awaitTermination(time, TimeUnit) method. see below code example.

```
public static void main(String[] args){
	ThreadPoolExecutor pool = Executors.newFixedThreadPool(5);
	Future<T> res1 = pool.submit(some task);
	Future<T> res2 = pool.submit(some other task);
	
	pool.shutDown();
	
	sout("Main thread completed");
}
```
in above code example, main thread won't wait until the pool gets shut. it will be non blocking operation.

```
public static void main(String[] args){
	ThreadPoolExecutor pool = Executors.newFixedThreadPool(5);
	Future<T> res1 = pool.submit(some task);
	Future<T> res2 = pool.submit(some other task);
	
	pool.shutDown();
	try{
	  boolean isTerminated = pool.awaitTermination(5, TimeUnit.SECONDS);
	}
	catch(Exception e){
	//
	}
	sout("Main thread completed");
}
```
in this example, main thread will wait 5 seconds to get pool shut. if during that time, pool gets shut, it will receive true, otherwise receive false and move ahead. 




------------------------------<<<<<<<<<<<<>>>>>>>>>>>------------------------


Executors class :

-> It is a utility class with below methods :

1. public static ThreadFactory defaultThreadFactory() : this method returns a thread factory. while creating ThreadPoolExecutor, one of the optional argument of constructor is ThreadFactory. so we use use this method to get ThreadFactory object and pass it.
2. public static ExecutorService newFixedThreadPool(int nThreads) : this method creates a ThreadPoolExecutor with fixed number of worker threads. 
3. public static ExecutorService newCachedThreadPool() : this method creates a ThreadPoolExecutor with initial corePoolSize = 0, maxCorePoolSize = Integer.MAX_VALUE. it will reuse threads from the pool if available otherwise will create new thread. thread will remain alive for 60 seconds after being idle.
4. public static ExecutorService newSingleThreadExecutor() : it will also create ThreadPoolExecutor with corePoolSize = 1, maxPoolSize = 1. means there will be only one thread present in pool at any time.
5. public static ExecutorService newWorkStealingPool() : this method will create a ForkJoinPool with corePoolSize (parallelism) = # of CPU core present in system.
6. public static ExecutorService newWorkStealingPool(int parallelism) : this method is overloaded version of above one. here corePoolSize = parallelism given by user.
7. public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) : this method creates the ScheduledThreadPoolExecutor pool with pool size = corePoolSize. 

-> Note that all of the above methods return ExecutorService which is interface implemented by ThreadPoolExecutor, ForkJoinPool and ScheduledExecutorService interface which is implemented by ScheduledThreadPoolExecutor.

| Method                                | `corePoolSize` | `maxPoolSize`       | Task Queue length     | When to Use                                                            |
| ------------------------------------- | -------------- | ------------------- | --------------------- | ---------------------------------------------------------------------- |
| `Executors.newFixedThreadPool(n)`     | `n`            | `n`                 | Unbounded             | When you want a fixed number of threads and limit concurrency          |
| `Executors.newCachedThreadPool()`     | `0`            | `Integer.MAX_VALUE` | Bounded with size = 0 | For many short-lived async tasks; tasks run immediately in new threads |
| `Executors.newSingleThreadExecutor()` | `1`            | `1`                 | Unbounded             | When tasks must run sequentially in a single thread                    |


------------------------------<<<<<<<<<<<<<>>>>>>>>----------------------------


ScheduledThreadPoolExecutor :

-> It is one another thread pool which extends ThreadPoolExecutor and implements ScheduledExecutorService interface.
-> it is used when we want to schedule a task at fixed delay, or at fixed intervals...
-> it has methods like : 
1. public ScheduledFuture<"?"> schedule(Runnable task, long delay, TimeUnit unit) : Submits a one-shot task that becomes enabled after the given delay. note that return type is ScheduledFuture which is child of Future. so we can use Future<"?"> = pool.schedule(task, detal, timeunit); in case of Runnable task, we have to use wild card ? for Future.
2. public <"V"> ScheduledFuture<"V"> schedule(Callable<"V"> callable, long delay, TimeUnit unit) : Submits a value-returning one-shot task that becomes enabled after the given delay. task will run only once. we can use Future<"V"> here as well.
3.  public ScheduledFuture<"?"> scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit) : after the initial delay given by user i.e. = initialDelay in above example, it will start the task execution and will keep repeating it at fixed period of time given by user i.e. = period in above example. you can cancel the task using cancel method of Future class. 
	ex:
	- If your task takes **5 seconds** to complete, but you scheduled it with a **2-second period**, then:
	    - At **t = 0s**, task starts
	    - At **t = 2s**, scheduler _tries_ to start a new run (but the previous one is still running)
	    - Once the first task completes at **t = 5s**, the next run begins **immediately**
	    - This can cause tasks to run **back-to-back** if the task duration > period
	
	✅ Another case:
	- If task takes **2 seconds** and period is **5 seconds**:
	    - At **t = 0s**, task runs
	    - At **t = 5s**, next run starts (even though task finished at t = 2s)
	    - At **t = 10s**, next run starts, and so on…
4. public ScheduledFuture<"?"> scheduleWithFixedDelay(Runnable command, long initialDelay,long delay, TimeUnit unit) : same as above method, but
	ex:
	- If the task takes **5 seconds** and you set the delay to **2 seconds**:
	    - At **t = 0s**, task starts
	    - It completes at **t = 5s**
	    - Next task starts at **t = 7s** (after a delay of 2 seconds)
	    - Next run at **t = 14s**, and so on…


-> **==scheduledFuture, ForkJoinTask are child of Future class. so you can use Future to store result of methods like submit, schedule...==**

