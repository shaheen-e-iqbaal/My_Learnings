

-> Critical section is a block of code that is executed by multiple threads and the output of that block can differ on the basic of order of execution by different threads.  such section of code is called critical section.

-> When the result of multiple threads executing a critical section may differ depending on the sequence in which the threads execute, the critical section is said to contain a race condition.

-> Race condition can be avoided using locks on critical section.