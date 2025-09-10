
->When you call a method annotated with `@Async` (or call any method which will run in different thread than the caller thread) from within an active transactional context, it executes in a **separate thread** with a **different call stack**. As a result, the **transaction context from the caller is not propagated** to the `@Async` method. Unless the `@Async` method is itself explicitly annotated with `@Transactional`, it **runs outside of any transaction**.


ex :
```
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userService;

    @PostMapping(path = "/updateuser")
    public String updateUserMethod(){
        userService.updateUser();  // Calls async method
        return null;
    }
}

@Component
public class UserService {

    @Autowired
    UserUtility userUtility;

    @Transactional
    public void updateUser(){
        userUtility.updateUser(); // Calls transactional method from new thread
    }
}

@Component
public class UserUtility {

    @Async
    public void updateUser(){
        // 1. update user status
        // 2. update user first name
        // 3. update user
    }
}
```
### Step-by-Step Breakdown:

1. **`userService.updateUser()` is called**
    
    - A transaction starts here because it's marked with `@Transactional`.
    - This is running on the **main thread** (HTTP request thread).
2. **Inside `updateUser()`, `userUtility.updateUser()` is called**
    
    - This method is marked `@Async`, so Spring executes it in a **completely separate thread**.
    - It **does not share the transaction context** of the caller (`userService.updateUser()`).
3. **Inside the `@Async` method**:
    
    - Any DB operations (e.g., updating user data) are **not inside a transaction** unless this method itself is annotated with `@Transactional`.
    - So those DB operations may be committed immediately and not rolled back with any outer logic.
4. **If the async method (`userUtility.updateUser`) throws a `RuntimeException`**:
    
    - That exception will occur in the **async thread**.
    - It **does not propagate** back to the main thread or affect the ongoing transaction in `userService.updateUser()`.
5. **If the transactional method (`userService.updateUser`) throws an exception**:
    
    - It will cause the main transaction to roll back.
    - However, since `userUtility.updateUser()` is running **independently** in a separate thread, its operations **will not be rolled back** — they may already be committed.


->To preserve the transaction context, it's better to **call a `@Transactional` method from within an `@Async` method**, rather than the other way around. This ensures that both methods execute in the **same new thread** (created by `@Async`) and the **transaction starts and remains within that thread**.

In contrast, calling an `@Async` method from a `@Transactional` method causes the `@Async` logic to run in a **different thread**, outside the scope of the original transaction.

ex : 
```
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userService;

    @PostMapping(path = "/updateuser")
    public String updateUserMethod(){
        userService.updateUser();  // Calls async method
        return null;
    }
}

@Component
public class UserService {

    @Autowired
    UserUtility userUtility;

    @Async
    public void updateUser(){
        userUtility.updateUser(); // Calls transactional method from new thread
    }
}

@Component
public class UserUtility {

    @Transactional
    public void updateUser(){
        // 1. update user status
        // 2. update user first name
        // 3. update user
    }
}
```



->==When a new thread is spawned (e.g., via `@Async`), it does not inherit the transaction context of the original thread. Each thread maintains its own separate transaction scope.==



How does @Async works :

-> spring uses a threads from a thread pool to execute a methods marked with this annotation. so these methods runs in a different thread then the caller main thread.
-> spring creates AOP Proxy of a class who has one or more async methods.

-> Spring boot uses ***ThreadPoolTaskExecutor*** as a thread pool which has fixed (aprox  8) corePoolSize and maxPoolSize, task queue size are equal to `Integer.MAX_VALUE`.
this thread pool is wrapper around Java's ThreadPoolExecutor. 

-> If spring doesn't find a Bean of ThreadPoolTaskExecutor, it uses ***SimpleAsyncTaskExecutor*** as a thread pool, which will create a new thread for each async task.

-> Async methods can have return type as Future or CompletableFuture or void.
-> use @EnableAsync to enable async in spring boot. 

-> How to handle exception thrown by method marked as @Async annotation? refer first part of General note under spring learning folder.