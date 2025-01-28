
run() -> This method get's called by start method. it contains all the logic we want to perform using this thread. directly calling this method from another method won't start the thread. to start a thread we must have to call start() method.

start() -> This is method responsible to start a thread. it behind the scene calls the run method.

sleep(t) -> This method makes a thread to sleep for a given time t. ==during this time, thread doesn't release a monitor lock.==

wait() -> this method makes thread to sleep until it is not waked up by notify() or notifyAll() method. ==during this time, it releases monitor lock.==

notify() and notifyAll() -> this method wakes up a threads which are waiting on a same object. notify() will wake up any arbitrary thread waiting on this object and notifyAll() will wake up all thread waiting on this object.

interrupt() ->

join() -> This method makes a thread to wait until the other thread doesn't gets finished. if we write t1.join() in main() function, main thread will wait for the t1 thread to get finished.

yield() -> 

setPriority() -> It is used to set priority of any thread. it is only hint to a Thread schedular. the default priority of a thread is equal to the priority of any thread which has created it. ex. the priority of main thread is 5. so any thread created by main thread will have priority = 5. to change default priority, this function is used.

setDaemon() -> Daemon threads are the ones which runs until any single non daemon thread is running. if all the non daemon threads gets finished, daemon thread will also get terminated regardless of they have completed their job or not.
Garbage collection thread is the example of Daemon thread. if we want to make any thread Daemon, we can do using this method.




### **Thread Methods and Their Explanations**

1. **`run()`**
    - **Usage**: This method is the entry point for the thread's execution logic. It contains the code that we want to execute within the thread.
    - **Behavior**:
        - When called directly like a regular method, it executes in the calling thread, not in a new thread.
        - The `start()` method internally calls this method.
    - **Exceptions**: None.

---

2. **`start()`**
    - **Usage**: Starts a new thread by allocating system resources for the thread and invoking the `run()` method in the newly created thread.
    - **Behavior**: Calling `start()` on a thread object that has already been started will result in an exception.
    - **Exceptions**:
        - Throws `IllegalThreadStateException` if the thread has already been started.

---

3. **`sleep(long millis)`**
    - **Usage**: Puts the current thread to sleep for the specified duration (`millis` milliseconds).
    - **Behavior**:
        - The thread does **not release its monitor lock** during the sleep.
        - It allows other threads to execute while the current thread is paused.
    - **Exceptions**:
        - Throws `InterruptedException` if the thread is interrupted while sleeping.

---

4. **`wait()`**
    - **Usage**: Makes the current thread release the monitor lock and wait until another thread invokes `notify()` or `notifyAll()` on the same object.
    - **Behavior**:
        - It must be called inside a synchronized block or method; otherwise, it throws an exception.
        - The thread will resume execution when it is notified or interrupted.
    - **Exceptions**:
        - Throws `IllegalMonitorStateException` if not called within a synchronized block/method.
        - Throws `InterruptedException` if the thread is interrupted while waiting.

---

5. **`notify()` and `notifyAll()`**
    - **Usage**: Used to wake up one or all threads waiting on the object's monitor.
    - **Behavior**:
        - `notify()`: Wakes up one arbitrary thread waiting on the monitor.
        - `notifyAll()`: Wakes up all threads waiting on the monitor.
        - These methods must be called within a synchronized block/method.
    - **Exceptions**:
        - Throws `IllegalMonitorStateException` if not called within a synchronized block/method.

---

6. **`interrupt()`**
    - **Usage**: Interrupts a thread, signaling it to stop or handle the interruption.
    - **Behavior**:
        - If the thread is sleeping, waiting, or blocked, it will throw `InterruptedException`.
        - For a running thread, it only sets the thread's interrupted status, which can be checked using `Thread.interrupted()` or `isInterrupted()`.
    - **Exceptions**: None (but may cause other methods like `sleep` or `wait` to throw `InterruptedException`).

---

7. **`join()`**
    - **Usage**: Makes the current thread wait for the thread on which `join()` is called to complete its execution.
    - **Behavior**:
        - For example, `t1.join()` in the main thread will make the main thread wait until `t1` finishes execution.
    - **Exceptions**:
        - Throws `InterruptedException` if the current thread is interrupted while waiting.

---

8. **`yield()`**
    - **Usage**: Suggests to the thread scheduler to temporarily pause the execution of the current thread, allowing other threads of the same or higher priority to execute. it will not release a monitor lock.
    - **Behavior**:
        - It is just a hint to the thread scheduler and does not guarantee that the current thread will stop execution.
    - **Exceptions**: None.

---

9. **`setPriority(int priority)`**
    - **Usage**: Sets the priority of a thread, where the value ranges from `Thread.MIN_PRIORITY` (1) to `Thread.MAX_PRIORITY` (10). The default is `Thread.NORM_PRIORITY` (5).
    - **Behavior**:
        - It provides a hint to the thread scheduler to determine the order of thread execution.
    - **Exceptions**:
        - Throws `IllegalArgumentException` if the priority is not in the valid range.

---

10. **`setDaemon(boolean on)`**
    - **Usage**: Marks a thread as a daemon thread or a user thread.
    - **Behavior**:
        - A daemon thread runs in the background and does not prevent the JVM from exiting if all user threads are complete.
        - Must be called before starting the thread; otherwise, it will throw an exception.
    - **Exceptions**:
        - Throws `IllegalThreadStateException` if called after the thread has been started.

---
