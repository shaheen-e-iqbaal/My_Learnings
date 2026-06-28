
-> It is used to synchronize the critical section.
-> it can be used in three way. 

```
1. synchronized void m() { ... } //locks on 'this'

2. static synchronized void s() { ... } //locks on ClassName.class object

3. void m() { synchronized (lockObj) { ... } } //locks on an explicit object
```

-> The primary difference between a synchronized instance method and a synchronized static method lies in the lock they acquire: an instance method locks on the object instance (this), while a static method locks on the Class object (ClassName.class)

-> You can call notify() and notifyAll(), wait() methods only while holding monitor of that object. if you call them without having monitor, it will throw `IllegalMonitorStateException`.

-> Never synchronize on a String literal, Boolean , or boxed primitive — these are pooled/cached and shared 
process-wide, so unrelated code can collide on the same monitor. Use a dedicated private final Object lock = new Object(); 

-> When a thread is trying to acquire monitor lock or trying to enter synchronized block/method, it is said that Thread is in Blocked state. this is only limited to monitor lock. if it's trying to acquire some other type of lock like Reentrant, ReadWrite... then it will be called in Waiting state and not blocked state. blocked state is explicitly for monitor.

-> synchronized blocks are re-entrant. i.e. when a thread acquires monitor lock of obj and enters critical section. now it calls some other method which has synchronized block synchronized on same obj. so this time, thread doesn't need to re-acquire monitor lock. this is called.

---

-> ==Limitations of synchronized block or tradoff between other locks and synchronized== :
1. It doesn't guarantee that which thread from the set of blocked threads will get monitor lock next. it possible that, without this guarantee, we can have thread starvation.
2. It doesn't have any functionality like wait this particular time to acquire monitor. if doesn't get by that time, return.
3. we can't do something like : we acquire monitor lock in first method and release it in some other method. 

ChatGPT version : 
## Limitations of `synchronized`

### 1. No guarantee of fairness

When multiple threads are blocked waiting to acquire a monitor, Java does **not guarantee** which thread will get the monitor next.

```java
synchronized(lock) {
    // critical section
}
```

Suppose three threads are waiting:

```text
Thread A
Thread B
Thread C
```

When the current thread releases the monitor, **any one of them may acquire it next**. There is no guarantee that the thread which has been waiting the longest will get the lock first.

Because of this, some threads may repeatedly lose the race and wait indefinitely, leading to **thread starvation**.

---

### 2. Cannot wait only for a limited time to acquire the lock

With `synchronized`, a thread trying to enter a synchronized block waits until the monitor becomes available.

```java
synchronized(lock) {
    // critical section
}
```

There is no way to say:

> "Try to acquire the lock for at most 2 seconds. If it is still unavailable, give up and do something else."

The thread simply remains blocked until it acquires the monitor.

---

### 3. Lock acquisition and release must happen in the same scope

A monitor is automatically acquired when entering a synchronized block and automatically released when leaving it.

```java
void methodA() {
    synchronized(lock) {
        // lock acquired

        // work...
    } // lock released here
}
```

It is **not possible** to do something like:

```java
void methodA() {
    // acquire lock
}

void methodB() {
    // release lock
}
```

In other words, you cannot acquire a monitor in one method and release it in another. The lock's lifetime is tied to the synchronized block or method.

---

### These limitations led to the introduction of explicit locks

Classes like `ReentrantLock` provide additional features such as:

- Fair locking (optional)
- Timed lock acquisition (`tryLock(timeout)`)
- Non-blocking lock acquisition (`tryLock()`)
- Acquiring and releasing locks in different methods
- Multiple condition variables (`Condition`)
- Interruptible lock acquisition (`lockInterruptibly()`)

---


---

##  You should always use loop inside synchronized block instead of `if` statement?

```
synchronized (lock) {
while (!conditionHolds()) {
	// MUST be while, not if
	lock.wait();
}
	// condition is true here; proceed
}
```
in above example, you shouldn't do like `if(!conditionHolds())`.  the reason is : suppose there are multiple threads which can influence the output of `conditionHolds()`. now, when some other thread calls notifyAll(), all the threads will move from waiting set to entry set. and one of them gets monitor lock and alters the output of `conditionHolds()`. now when your thread gets monitor, the result of `conditionHolds()` already becomes false. which your thread won't check due to if statement. which is against your expectation.

ChatGPT version of above :
### Always use a `while` loop with `wait()`, never an `if`

When a thread calls `wait()`, it releases the monitor and enters the object's **wait set**. After another thread calls `notify()` or `notifyAll()`, the waiting thread does **not** immediately continue execution. Instead, it first moves to the **entry set** and must compete again to acquire the monitor.
Therefore, after a thread wakes up and reacquires the monitor, it **cannot assume that the condition it was waiting for is still true**. It must recheck the condition before proceeding.
```
synchronized (lock) {    
while (!conditionHolds()) {        
	lock.wait();    
}    
		// At this point conditionHolds() 
		is guaranteed to be true    
		// Safe to proceed
}
```
**Never do this:**

```
synchronized (lock) {
    if (!conditionHolds()) {   
	    lock.wait();    
    }    
    // Unsafe!
}
```


###  Why `while` instead of `if`?

###  1. Another thread may change the condition before this thread gets the lock

Suppose multiple threads are waiting for the same condition:

```
Wait Set:Thread AThread BThread C
```

Another thread calls:

```
lock.notifyAll();
```

All waiting threads move from the **wait set** to the **entry set**:

```
Entry Set:Thread AThread BThread C
```

Assume Thread A acquires the monitor first and consumes the shared resource, making:

```
conditionHolds() == false
```

again.

Later, Thread B acquires the monitor.

If Thread B had used:

```
if (!conditionHolds())    wait();
```

then after waking up it would continue execution without checking the condition again, even though the condition is now false.

Using a `while` loop forces Thread B to recheck the condition and go back to waiting if necessary.

---

###  2. Spurious Wakeups

The Java specification allows a thread waiting on a monitor to wake up **without any thread calling `notify()` or `notifyAll()`**.

This is called a **spurious wakeup**.

For example:

```
synchronized(lock) {    while (!conditionHolds()) {        lock.wait();    }}
```

Suppose:

```
conditionHolds() == false
```

The thread executes:

```
lock.wait();
```

Even if nobody calls `notify()`, the JVM is allowed to wake the thread up. If an `if` statement were used, the thread would proceed even though the condition is still false.

The `while` loop protects against such unexpected wakeups.

---

###  3. A notification does not guarantee that the condition is true

A call to:

```
lock.notify();
```

only means:

> "Something may have changed."

It does **not** mean:

> "The condition you are waiting for is definitely true."

Therefore, after waking up, the thread must always verify the condition again.

---

---



--- 

## How does wait() and notify() works

Excellent question.

To answer it properly, we need to separate **two completely different groups of threads** that are associated with a monitor:

1. **Threads blocked trying to acquire a monitor**
2. **Threads that already owned the monitor, called `wait()`, released it, and are waiting for a notification**

Most developers incorrectly think these are the same queue. They are not.

---

  Monitor Internals

Consider:

```java
Object lock = new Object();
```

Every Java object can act as a monitor.

Conceptually, JVM associates something like this with the object:

```text
Monitor
|
├── Owner Thread
├── Entry Set (Blocked Threads)
└── Wait Set (Waiting Threads)
```

---

 1. Threads blocked acquiring monitor

Suppose:

```java
synchronized(lock) {
    // critical section
}
```

Thread A enters first.

---

Current state:

```text
Monitor(lock)

Owner = Thread A

Entry Set = empty

Wait Set = empty
```

---

Now Thread B tries:

```java
synchronized(lock) {
}
```

But A already owns the monitor.

So JVM does:

```text
Monitor(lock)

Owner = Thread A

Entry Set:
    Thread B

Wait Set:
    empty
```

Thread B becomes:

```text
BLOCKED
```

state.

---

Now Thread C tries:

```java
synchronized(lock) {
}
```

State becomes:

```text
Monitor(lock)

Owner = Thread A

Entry Set:
    Thread B
    Thread C

Wait Set:
    empty
```

---

These threads are not running.

They're parked by JVM/OS until monitor becomes available.

---

 What happens when A exits synchronized?

```java
synchronized(lock) {
}
```

finishes.

JVM performs:

```text
monitor exit
```

and releases ownership.

---

Monitor becomes:

```text
Owner = none

Entry Set:
    Thread B
    Thread C
```

JVM selects one waiting thread.

Suppose B.

Now:

```text
Owner = Thread B

Entry Set:
    Thread C
```

B becomes runnable.

---

 Important

These threads never called:

```java
wait()
```

They only attempted:

```java
synchronized(lock)
```

and failed to acquire monitor.

Therefore they belong to:

```text
Entry Set
```

---

2. Threads that call wait()

Completely different scenario.

Suppose:

```java
synchronized(lock) {
    lock.wait();
}
```

Thread A owns the monitor.

---

Before wait:

```text
Owner = Thread A

Entry Set = empty

Wait Set = empty
```

---

When JVM executes:

```java
lock.wait();
```

it performs three operations atomically:

Step 1

Release monitor.

```text
Owner = none
```

---

Step 2

Move thread into wait set.

```text
Wait Set:
    Thread A
```

---

Step 3

Suspend thread.

State becomes:

```text
WAITING
```

---

Monitor now:

```text
Owner = none

Entry Set = empty

Wait Set:
    Thread A
```

---

Notice:

A is no longer competing for the lock.

It is waiting for a notification.

---

Another thread enters

Thread B:

```java
synchronized(lock) {
}
```

acquires monitor.

State:

```text
Owner = Thread B

Wait Set:
    Thread A
```

---

notify()

Thread B:

```java
synchronized(lock) {
    lock.notify();
}
```

---

Many developers misunderstand this.

`notify()` DOES NOT wake A immediately.

Instead JVM moves A:

```text
Wait Set
   →
Entry Set
```

---

After notify:

```text
Owner = Thread B

Entry Set:
    Thread A

Wait Set:
    empty
```

---

Thread A still cannot continue.

Because B still owns monitor.

---

B exits synchronized

```java
synchronized(lock) {
    lock.notify();
}
```

finishes.

Monitor released.

Now A competes again.

---

State:

```text
Owner = Thread A

Entry Set:
    empty

Wait Set:
    empty
```

A resumes after wait().

---

notifyAll()

Suppose:

```text
Wait Set

Thread A
Thread B
Thread C
```

---

When:

```java
lock.notifyAll();
```

executes:

All waiting threads move:

```text
Wait Set
   →
Entry Set
```

---

Result:

```text
Entry Set

Thread A
Thread B
Thread C
```

Now they compete for monitor one by one.

---

How JVM Tracks These Threads

The JVM creates an internal monitor structure.

The actual implementation differs between JVMs, but conceptually:

```text
Object
  |
  ---> ObjectMonitor
```

OpenJDK internally has something similar to:

```cpp
ObjectMonitor
{
    Thread* owner;

    Queue entryList;

    Queue waitSet;
}
```

where:

owner

Tracks:

```text
Current monitor owner
```

---

entryList

Tracks:

```text
Threads blocked trying to enter
synchronized block
```

---

waitSet

Tracks:

```text
Threads that executed wait()
```

---

Thread States

Blocked acquiring monitor:

```java
synchronized(lock)
```

fails

↓

```text
BLOCKED
```

state.

---

Calling wait():

```java
lock.wait();
```

↓

```text
WAITING
```

or

```text
TIMED_WAITING
```

if timeout supplied.

---

This distinction is very important in thread dumps.

---

Interview-Level Summary

Every monitor conceptually maintains two separate collections of threads:

```text
1. Entry Set
2. Wait Set
```

Entry Set

Contains:

```text
Threads trying to acquire monitor
but unable to because another thread owns it.
```

Thread state:

```text
BLOCKED
```

---

Wait Set

Contains:

```text
Threads that owned monitor,
called wait(),
released monitor,
and are waiting for notify()/notifyAll().
```

Thread state:

```text
WAITING
or
TIMED_WAITING
```

---

notify()

Moves one thread:

```text
Wait Set → Entry Set
```

---

notifyAll()

Moves all threads:

```text
Wait Set → Entry Set
```

---

Important

A thread awakened by `notify()` does **not** immediately continue execution.

It must first re-acquire the monitor by competing with threads already in the Entry Set. This is why every correct `wait()` usage is written inside a loop:

```java
synchronized(lock) {
    while(!condition) {
        lock.wait();
    }
}
```

because by the time the thread reacquires the monitor, the condition may no longer be true.