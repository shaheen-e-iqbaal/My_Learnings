
-> In ThreadPoolExecutor, when you use .submit() method instead of .execute() method, it will return object of future class.

Methods of Future class : 

1. cancel(true) : will cancel the the task being run.
2. isCancelled() : will return the true if the task was cancelled before it get complete.
3. isDone() : will return true if this task got completed. the completion can be anything due to normal completion, explicit cancellation or due to some exception while running.
4. get() : will return the value returned by the task. if the task is still in progress, the caller thread of this get() method will wait until the task is completed by worker thread. throws checked exception.
ex :
```
Future<Integer> futureObj = threadPool.submit(() -> {
Thread.sleep(10000);
return 2;
});

try{
int result = futureObj.get();
}
catch (InterruptedException | ExecutionException e){
//
}
```

-> in above example, when the main thread submits the task, it get's futureObj in return. so in next, when it does futureObj.get(), if the worker thread has completed the task, it will return 2, otherwise main thread will wait until the worker thread completes it's task and returns value.

-> Future's Limitation : 
1. Doesn't have any exception handling functionality.
2. Can't chain the operations.
3. .get() blocks the thread until result is available.


CompletableFuture :

-> It is known as promise of java.
-> it is used to achieve asynchronous programming in java.
-> it by default uses common ForkJoinPool.

-> CompletableFuture addresses the limitations of Future. it allows chaining of multiple operations. you can do like .applyThen() which will run asynchronous after the previous method returns result. 
-> .exceptionally() method allows to handle exception.