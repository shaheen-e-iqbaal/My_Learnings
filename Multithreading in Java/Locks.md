
Lock can be categorized multiple ways : 
1. Shared Lock Vs Exclusive Lock.
2. Optimistic lock Vs Pessimistic lock.

`Monitor Lock :`

-> Synchronize keyword in Java uses monitor lock by default. It can be both object level as well as class level.

-> when you make instance method synchronized, it uses object level lock. also when you do like Synchronized(this), it will also use object level lock.

-> when you make static method synchronized, it uses class level lock. also when you do like Synchronized(Employee.class), it will also use class level lock. means, only one thread at a time can access such block or method, no matter those thread are working on same Employee object or different one.

-> ==wait(), notify() and notifyAll() methods can only be used with monitor lock.==


Reentrant Lock :

-> It provides little more control on lock than synchronized. below are some of it's benefits.

1. In Java, when multiple threads are waiting to acquire a monitor lock using `synchronized`, there is **no guarantee** on the order in which threads will get the lock. This means that even if one thread has been waiting for a long time (say 1 hour), another thread that just started waiting may still acquire the lock first. So, **thread starvation** is possible with `synchronized`. However, `ReentrantLock` provides a way to handle this using a fairness policy. If you create a `ReentrantLock` with the fairness parameter set to `true` like this: `ReentrantLock lock = new ReentrantLock(true);`, then the lock will be given to the thread that has been waiting the longest. This ensures a **first-come, first-served** order and helps prevent starvation. By default, the fairness parameter is `false`, which means the lock can be granted in any order, similar to `synchronized`, but it can offer better performance in some cases.

2. while using reentrant lock, it is possible that we can acquire a lock in one method and can release it in some other method. but in monitor lock, it is not possible.

3. Reentrant class have methods like :
	1.  tryLock() : 
```
if (lock.tryLock()) {
    try {
        // do work
    } finally {
        lock.unlock();
    }
} else {
    // lock was not available
}
```
- Tries to acquire the lock **immediately**
- If not available, returns `false` instead of blocking.

	2. tryLock(long time, TimeUnit unit) :
```
if (lock.tryLock(5, TimeUnit.SECONDS)) {
    try {
        // got the lock within 5 seconds
    } finally {
        lock.unlock();
    }
} else {
    // timeout occurred
}
```
- It will **wait up to 5 seconds** for the lock to become available.
- If it gets the lock within that time, it returns `true`.
- If not, after 5 seconds it gives up and returns `false`.

## Why Is It Called **"Reentrant" Lock**?

A lock is called **reentrant** if the **same thread** can acquire it **multiple times** without getting blocked or causing a deadlock.

This is important when:

- A thread enters a method that has acquired a lock.
- That method calls another method (or itself recursively) that tries to acquire the **same lock**.
- With a **reentrant** lock, this is allowed — the thread is recognized as the "owner" and the lock **just increases a count**.
- Without reentrancy, the thread would **block itself**, resulting in a deadlock.

==Both `synchronized` and `ReentrantLock` in Java are **reentrant**.==

```
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void outerMethod() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " acquired lock in outerMethod");
            innerMethod(); // calls another method that tries to lock again
        } finally {
            lock.unlock();
            System.out.println(Thread.currentThread().getName() + " released lock in outerMethod");
        }
    }

    public void innerMethod() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " acquired lock in innerMethod");
        } finally {
            lock.unlock();
            System.out.println(Thread.currentThread().getName() + " released lock in innerMethod");
        }
    }

    public static void main(String[] args) {
        ReentrantExample example = new ReentrantExample();

        Thread t1 = new Thread(example::outerMethod, "Thread-1");
        t1.start();
    }
}
```


ReadWriteLock :

-> ReadWriteLock is an interface which is implemented by ReentrantReadWriteLock. 
```
ReadWriteLock rw = new ReentrantReadWriteLock(true);
rw.readLock().loc
k(); //Will get read lock
rw.readLock().unlock(); //will release read lock

simillarly you can get and unlock write lock.
```

-> More than one thread can get read lock which is shared lock. but only one thread at a time can get write lock which is exclusive lock. during write lock, no other thread can have read lock.

-> This lock can be used when you have high number of read operations compared to write operations. like 1/1000 type ratio.


-------------------<<<<<<<<<<<<<<<<>>>>>>>>>>>--------------------------


***==When to Use :==***

Optimistic locks  :

-> When you want higher concurrency for a resource and know that there will be less conflict.
-> ex : stampede lock in java


Pessimistic locks :

-> when you know that conflict will be higher. it reduces the concurrent access of resource by putting actual locks like monitor, reentrant, read write... so concurrency will be low.


read write lock :

-> when you have high read and low write operations. like 10/1000.

semaphore lock :

-> it can be used in connection pool. like maximum 10 threads at a time can access a pool.