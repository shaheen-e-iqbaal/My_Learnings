
ThreadLocal<"T"> :

-> when you want to associate some "Secret" value specific to the current thread only, you can do so using ThreadLocal class. 
-> That value can only be seen by the current thread.
-> you can use single ThreadLocal instance across multiple threads. it will associate that secret field to the current running thread only.

-> In practice, we don't create thread local instance per thread. we create one global instance of it. and multiple threads use this instance to store value in it. for each purpose, we can have separate thread local instance. like to store user context, we can have one global `static ThreadLocal<UserContext>` object. to store request context, we can have one  global `static ThreadLocal<RequestContext>` object. so, all the threads who want to store user context or request context can use these instance only.

ex : 
```
public class Example {
    private static final ThreadLocal<Integer> threadLocalId = new ThreadLocal();

    public static void main(String[] args) {
        Runnable task = () -> {
            threadLocalId.set((int)(Math.random() * 100));
            System.out.println(Thread.currentThread().getName() + " has ID: " + threadLocalId.get());
            threadLocalId.remove();
        };

        new Thread(task).start();
        new Thread(task).start();
    }
}
```
so in above example, when you do call threadLocal.set(someValue) method, it will set someValue to the thread running that line. it uses Map Data structure to store this key value pair i.e. thread -> value.

-> each thread has instance field of ThreadLocalMap. so when you call threadLocalId.set(someValue), it will add the 
key->value pair in the map `ThreadLocalMap` of the current thread where key = threadLocalId and value = someValue. this way, each thread can have multiple entries in it's map. like one entry can store current txn context, other can store current user context with info about authorization & authentication.

-> while using thread pool, make sure to remove that secret at the end of task execution. so when you do threadLocalId.remove(), it will remove the key = threadLocalId from the thread local map of the current thread.

-> so single ThreadLocal instance can be used by multiple threads. in practical scenario, we have static global thread local object which is used by multiple threads.

-> In ThreadLocalMap, the keys (ThreadLocal objects) are stored as WeakReferences, while the values are stored as strong references. If a ThreadLocal object becomes unreachable, it can be garbage collected, causing its corresponding key in the ThreadLocalMap to become null. However, the associated value remains strongly referenced by the map and cannot be garbage collected until the entry is explicitly removed or the thread terminates. This can lead to memory leaks, especially in thread pools, where worker threads are long-lived and reused across multiple tasks. Therefore, always call ThreadLocal.remove() (typically in a finally block) once the ThreadLocal is no longer needed to prevent stale entries and memory leaks.

Thread Local used by Spring Boot :

-> In transactional annotation, we can have multiple db calls. it is mendatory to perform this DB Operations under single txn. so, how does spring boot use single txn for multiple different db call? spring boot will create proxy and will intercept txn method call. it will then get one txn from transaction manager. will create one thread local with this transaction as it's value. now answer why does present txn context doesn't get propagated to methods marked as @Async or more formally methods which are gonna run in different thread then caller thread?

-> Spring security is another area where spring boot uses Thread local. when request arrives, spring boot stores authentication and authorization context holder class instance in thread local. this way, we don't have to pass this context to all the way down to our application methods.




Virtual Thread : 

-> normal threads which we create like new Thread(Runnable), they are called platform threads. each platform thread is 1:1 associated with OS thread. so whenever we create any platform, new OS thread is created by kernel and associated with platform thread. this creation of OS thread requires system calls to OS which are expensive. also platform thread requires around 1MB of heap space from JVM instance. so it is costly in that way as well. also if we create large number of platform thread, then the context switching process becomes frequent in CPU. so considerable amount of time will go in this context switching process which can be batter utilized to perform actual task.

-> so considering above problems of platform threads, java created a Project Loom under which development of virtual thread concept got initiated. 

-> virtual threads are much more light-weight than platform threads. each virtual thread need memory of some KB order which is much lesser than platform thread. 

-> how virtual threads works? : when you create a virtual thread, it will run on top of any available platform thread. who creates these platform thread then? they will be created by the JVM. JVM will use ForkJoinPool (**it is not ForkJoinPool.commonPool() one. this new pool is created specifically for virtual threads carrier purpose**) to maintain these platform threads. the total platform threads will be <= CPU cores present in system. so when you start virtual thread, that thread will get associate with any available platform thread. that platform thread is already associated with OS thread. so when this virtual thread goes into blocking state, like due to some network call, IO call etc. then this thread will offloaded from that platform thread and some other waiting virtual thread will start running on this platform thread. so the platform thread doesn't get blocked due to the blocking task of virtual thread. this is one of the major benefit of virtual thread. now when that virtual thread comes back from waiting state, it will run top of any available platform thread. here note that, it is not necessary that it run on same platform thread again on which it ran earlier. so that's why virtual threads are much efficient to perform the blocking task. so we can say that, virtual threads increases throughput. 

-> In normal platform thread, it is onto the OS scheduler to when to allocate CPU and for how long. but in case of virtual thread, until the virtual thread doesn't goes into blocked state (Loom aware), it will have underlying platform thread. JVM won't take away from it.

-> virtual threads are totally managed by JVM. their scheduling on top of platform thread and all relevant jobs are managed by JVM only.

-> How does JVM will find out that this virtual thread is now gonna enter in block state? JVM will know it by identifying that a task is loom aware or not? if it's loom aware task, it will know that virtual thread is gonna enter in blocking state. if it's not loom aware task, then it won't know that thread is gonna enter in block state. even thread remains blocked for long amount of time, still JVM will don't de-allocate platform thread from it. some loom aware task are like : Thread.sleep()... 
and most of the JDBC drivers (MySQL, PostgreSQL) are non loom aware. so if you make DB call using any of these driver, your virtual thread will go into block state. but JVM won't know about it. so, it won't deallocate platform thread from it.
-> you can't manually tell JVM that i am about to go into some blocking state. JVM automatically handles it for loom aware tasks.


When to use / not use virtual threads :

-> when your tasks are Blocking IO type and they are Loom aware, then virtual threads huge positive difference in performance and throughput.

-> it does not make any difference when used for CPU intensive tasks.

-> it also does not make any difference when you use for blocking IO task using libraries which are non loom aware.


What will happen when virtual thread tries to acquire monitor, ReentrantLock....? : (we can use this point to show benefits of ReentrantLock over synchronized/monitor lock)

-> when virtual thread tries to acquire monitor lock, if it's available, then it will proceed, but if not, then it will wait and will come into blocking state. but this blocking state doesn't trigger JVM to offload it from the underlying platform thread. so, that platform thread will remain attached to it and remain in blocked until virtual thread doesn't get monitor lock.

-> but when it tries to acquire lock like ReentrantLock, if not available, then JVM will offload it from underlying platform thread. so the platform thread won't be blocked.
when it get's the lock, it will get reassigned to some available platform thread again.


How to make virtual thread in Java? :

```
1.
	Thread t = Thread.ofVirtual().start(Runnable r);

2.
	Thread t = Thread.ofVirtual().unStarted(Runnable r);
	t.start();

3.
	ExecutorService virtualThreadPool =                              Executors.newVirtualThreadPerTaskExecutor();

	virtualThreadPool.submit(Callable/Runnable task);
```


Normal Loom aware tasks :

1. Thread.sleep()
2. wait()
3. countDownLatch.wait()
4. semaphore.acquire()
5. reentrantLock.lock()
6. Socket I/O (network calls where we have to wait until the data is received on socket)

Loom un-aware tasks :
1. synchronized
2. native calls