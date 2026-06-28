
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
-> spring creates AOP Proxy of a class who has one or more async methods. note that due to this, your async method should be of valid signature so that it can be proxied. refer [[Proxy in Spring#^f5e730]] to learn about proxy.

-> Spring boot uses ***ThreadPoolTaskExecutor*** as a thread pool which has fixed (approx  8) corePoolSize and maxPoolSize, task queue size are equal to `Integer.MAX_VALUE`.
this thread pool is wrapper around Java's ThreadPoolExecutor. 

***Which Thread Pool's threads will Spring boot use to run Async methods?***

```
	In Spring Boot, `@Async` methods do **not run on the request thread** (like the one from Apache Tomcat). Instead, they are executed using a separate **Executor (thread pool)** managed by Spring.

	By default, Spring Boot auto-configures a `ThreadPoolTaskExecutor` bean named **`applicationTaskExecutor`**, but only if you have **not defined any custom Executor bean**. This default executor is used for running all `@Async` methods, and its threads usually have names like `task-1`, `task-2`, etc.

	If you create your own Executor bean, Spring Boot will **not create** the default `applicationTaskExecutor`. Now Spring needs to decide which executor to use for `@Async`. It follows a priority-based resolution. First, if you explicitly specify an executor in `@Async("beanName")`, it will use that. If not, it looks for a bean named **`taskExecutor`** and uses it as the default. If there is only one Executor bean in the context, Spring will automatically use that. However, if **multiple Executor beans exist and none is clearly marked as default**, Spring cannot decide which one to use.

	In such ambiguous cases, instead of failing immediately, Spring falls back to using **`SimpleAsyncTaskExecutor`**, which is not a real thread pool. It creates a new thread for every task and does not reuse threads, which can be inefficient and risky under heavy load. You can पहचान this fallback easily because the thread names will look like `simpleAsyncTaskExecutor-1`.

	To avoid confusion and ensure predictable behavior, it is recommended to either define a bean named **`taskExecutor`**, mark one executor as `@Primary`, or explicitly specify the executor in the `@Async` annotation. Also, whenever you create a `ThreadPoolTaskExecutor`, make sure to call `initialize()`; otherwise, it may not work correctly.
```


-> Async methods can have return type as Future or CompletableFuture or void.
 
 -> ==**use @EnableAsync to enable async in spring boot. without this, @Async functionality won't work.**==

-> How to handle exception thrown by method marked as @Async annotation? refer first part of General note under spring learning folder.

-> Refer [Article](https://dev.to/realnamehidden1_61/how-does-async-work-internally-in-spring-boot-35h5) for @Async internal working info.