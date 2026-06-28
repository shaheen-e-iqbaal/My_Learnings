
`CountDownLatch` :
-> It is a class provided by java.util.concurrent package. it takes one non negative integer. ex :
`CountDownLatch latch = new CountDownLatch(5);`

-> It is used in case like if you want to achieve something like that, if some prerequisite tasks gets completed, then only my current task can start. suppose, before processing order, you want to check inventory, verify user, verify location,... when all these work completes then only you will start processing order.

-> It is one time. once the count reaches to zero, it can't be reset. also multiple threads can wait on single latch to be completed. single thread can multiple times call `countDown()`

Key methods : 

| Method                                       | Description                                                                                                                                                                        |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CountDownLatch(int count)`                  | Creates a latch initialized with the given count. The count represents the number of events/tasks that must complete before waiting threads are released.                          |
| `void await()`                               | Causes the current thread to wait until the latch count reaches **0**. Throws `InterruptedException` if the waiting thread is interrupted.                                         |
| `boolean await(long timeout, TimeUnit unit)` | Waits until the count reaches **0** or the specified timeout expires. Returns `true` if the latch reached zero, otherwise `false`. Throws `InterruptedException` if interrupted.   |
| `void countDown()`                           | Decrements the latch count by **1**. When the count reaches **0**, all waiting threads are released. Calling `countDown()` after the count has already reached zero has no effect. |
| `long getCount()`                            | Returns the current count of the latch. Mainly useful for debugging or monitoring; the value may change immediately due to concurrent updates.                                     |

Cyclic barrier :

-> CyclicBarrier is used when a fixed number of threads need to wait for each other at a common synchronization point (barrier). Each thread performs some work independently and then calls await(). Once all participating threads reach the barrier, they are released together and continue executing the next phase.

-> Unlike CountDownLatch, a CyclicBarrier is reusable. After all threads cross the barrier, it is automatically reset and can be used again for the next synchronization point. This makes it suitable for multi-phase algorithms where the same group of threads repeatedly waits for each other.

-> An optional barrier action (Runnable) can be provided while creating the barrier. This action is executed once after the last thread reaches the barrier and before all waiting threads are released.

| Method                                               | Description                                                                                                                                                                                                                   |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CyclicBarrier(int parties)`                         | Creates a barrier that waits for the specified number of threads (`parties`).                                                                                                                                                 |
| `CyclicBarrier(int parties, Runnable barrierAction)` | Creates a barrier with an action that is executed once after the last thread arrives and before releasing all waiting threads.                                                                                                |
| `int await()`                                        | Waits until all participating threads reach the barrier. Returns the arrival index of the current thread (last arriving thread gets `0`). Throws `InterruptedException` or `BrokenBarrierException` if the barrier is broken. |
| `int await(long timeout, TimeUnit unit)`             | Waits until all threads arrive or the timeout expires. Throws `TimeoutException`, `InterruptedException`, or `BrokenBarrierException` if the barrier is not successfully crossed.                                             |
| `int getParties()`                                   | Returns the number of threads required to trip the barrier.                                                                                                                                                                   |
| `int getNumberWaiting()`                             | Returns the number of threads currently waiting at the barrier.                                                                                                                                                               |
| `boolean isBroken()`                                 | Returns `true` if the barrier is in a broken state (due to interruption, timeout, or reset).                                                                                                                                  |
| `void reset()`                                       | Resets the barrier to its initial state. Any threads currently waiting receive a `BrokenBarrierException`.                                                                                                                    |


