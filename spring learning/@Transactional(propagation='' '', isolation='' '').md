
-> `@Transactional` is specifically designed for **managing transactions in resources that support transaction boundaries**, most commonly:

- **Relational databases** (via JPA/Hibernate, JDBC, etc.)
- Some NoSQL databases also supports it. (like MongoDB)
- **Message queues** (JMS, Kafka with some configuration)
- **Distributed transactions** (XA, JTA — advanced cases)

-> Can we @Transactional annotation to perform file system operations?
ans :
`@Transactional` **only makes sense** for resources that support transaction semantics (commit/rollback). File systems do not, so Spring cannot roll back file changes even if the method is marked `@Transactional`

-> So when we use this annotation, we don't need to worry about ACID properties, Spring itself handles that part of commit, rollback in case of any failure etc. we just have to write our logic related to transaction.

-> @Transactional is implemented using AOP with @Around type advice. so spring will create a proxy of class if that class or any of it's method is marked with this annotation.

-> To use @Transactional, we have to add dependency of Data JPA or JDBC in pom.xml. this annotation doesn't come with spring web dependency.

-> It can be applied to any method or class, interface, enum

-> There are various transactional manager which can be used by spring like Data source transaction manager (use when u are using plain JDBC), JPA transaction manager, Hibernate Transaction manager, JTA (java transaction API) transaction manager (used in distributed transactions). 

-> for learning about propagation and isolation level, see or read part 3 of Shrayansh Jain playlist.



--------------------<<<<<<<<<<<<<<<<>>>>>>>>>>------------------------



Q. What happens if one `@Transactional` method calls another `@Transactional` method in the same class?
->If an `@Transactional` method calls **another `@Transactional` method in the same class**, **the second method's transaction behavior will be ignored**. i.e. new transactional context won't be created for second method. it will stay in the transactional context of first method i.e. caller method. 

### Why?

Spring's `@Transactional` works by creating a **proxy** around the bean. This proxy intercepts method calls to apply transaction logic.

But when one method in the same class calls another, **it's just a plain Java method call — not going through the proxy.**
ex: 
@Service
public class MyService {

    @Transactional
    public void outer() {
        // Transaction starts here
        inner(); // This call is NOT intercepted by Spring proxy
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void inner() {
        // This won't actually start a new transaction! it will stay in the same parent transaction context.
    }
}

In this case:

- `outer()` starts a transaction ✅
    
- `inner()` is just a **regular method call**, not intercepted 🚫
    
- So `inner()` **runs within the same transaction context as `outer()`**, ignoring its own `@Transactional` settings like `REQUIRES_NEW`

-> to get the transactional work for the inner() method as well, define it in different bean.

-> In Spring, when a method is executed within the scope of an active transaction and runs on the same thread, it automatically participates in that transaction. This is true even if the called method is in another class and is not annotated with `@Transactional`. The transaction continues across method calls unless a new one is explicitly requested using a different **propagation setting**, like `Propagation.REQUIRES_NEW`.

ex : 
```
@Service
 public class ServiceA {
    @Autowired
    private ServiceB serviceB;

    @Transactional
    public void outerMethod() {
        // Insert A
        serviceB.innerMethod();  // Runs in same transaction
        // Insert C
    }
}

@Service
 public class ServiceB {
    public void innerMethod() {
        // Insert B
    }
 }
```
#### What happens:

- `outerMethod()` starts a transaction.
- `innerMethod()` is not transactional, but is called **within the same thread** and **context**.
- All inserts A, B, and C run in **one transaction**.
- If an exception is thrown and not caught, everything is rolled back.

==-> Transaction context of one thread doesn't get pass to new thread.==


**Hierarchy of Transaction manager** : 

-> Refer [This](https://www.google.com/search?sca_esv=292513031035339b&q=transaction+manager+hierarchy+in+spring+boot&udm=2&fbs=AIIjpHxU7SXXniUZfeShr2fp4giZ1Y6MJ25_tmWITc7uy4KIeoJTKjrFjVxydQWqI2NcOha3O1YqG67F0QIhAOFN_ob1yXos5K_Qo9Tq-0cVPzex8akBC0YDCZ6Kdb3tXvKc6RFFaJZ5G23Reu3aSyxvn2qD41n-47oj-b-f0NcRPP5lz0IcnVzj2DIj_DMpoDz5XbfZAMcEl5-58jjbkgCC_7e4L5AEDQ&sa=X&ved=2ahUKEwjS04Hw9-mNAxUvcvUHHRSHKYkQtKgLegQIGBAB&biw=1536&bih=730&dpr=1.25#vhid=ZnXwfSEJYJXWlM&vssid=mosaic) image or Shrayansh Jain's transaction part 2 lecture notes to learn about the hierarchy of transaction manager and different types of transaction manager.



----------------------<<<<<<<<<<<<<<>>>>>>>>>---------------------------



→ By default, a Spring-managed transaction will **only roll back** if an exception of type:
   - `RuntimeException` (unchecked exception), or
   - `Error`
  is thrown from within the **proxied method**.

→ If a checked exception (i.e., subclass of `Exception` but not `RuntimeException`) is thrown:
   - Spring will **not roll back** the transaction by default.
   - Instead, it will **commit** the transaction, then rethrow the exception.

→ This is because Spring's default rollback rules mimic the EJB model: rollback only on unchecked exceptions.

→ ==Also note: If you catch the exception within your method and don't rethrow it, the transaction **won’t roll back**, because Spring proxy never sees the exception.==

-> if our method throws checked exception, then to rollback, we can catch that checked exception and can throw runtime exception from it.
ex : 
public void updateData() {  
try {
//some database operations....
// Business logic that might throw checked exceptions  
}
catch(checked exception){
throw new RuntimeException();
}
}

-> but if we do like :

public void updateData() throws checkedException {  
// Business logic that might throw checked exceptions  
}

-> in above case, rollback won't happen, because thrown exception is checked one. so it won't rollback it.

-> To rollback transaction for other exception except runtime and error, we have to do like :

@Transactional(  
rollbackFor = { CustomCheckedException.class },  
noRollbackFor = { CustomIgnoredException.class }  
)  
public void updateData() {  
// Business logic that might throw exceptions  
}

so in above code, when method throws exception of type CustomCheckedException or it's subclass type, rollback will happen. but won't happen for exception of type CustomIgnoredException or any of it's subclass.



----------------------<<<<<<<<<<<<<<>>>>>>>>>---------------------------

When we call transactional method(call should be intercepted by proxy), the proxy will intercept that method call. it will use the transactinal manager (if we have provided our own transaction manager like DataSourceTransactionManager then it will use that or will use the default one) to issue the call to underlying database to start the txn. after that proxy will call our underlying method. now when our method finishes, the transaction manager will issue the commit or rollback call to the db depending on our method threw any exception (default runtime exception) or not.
-> one thing to notice is that, to create the bean of any PlatformTransactionManager, it is required to pass the Bean of DataSource.
-> so the thing we can learn here is, do not mark any method Transactional unnecessary. because if we mark it, it will need PlatformTransactionManager to manage transaction lifecycle which eventually will keep hold the active database connection from connection pool like HikariCP.

ChatGPT version of above note :
```
  ### How `@Transactional` really works in Spring

 1. **Proxy intercepts your method call**  
    When you call a `@Transactional` method,you’re  not calling it directly.  
    The call first goes through a Spring **proxy**  created by AOP.
    
 2. **Proxy asks the Transaction Manager to start a transaction**
    
    - The proxy checks: "This method is transactional. Do we already have a transaction running?"
        
    - If not, it calls a `PlatformTransactionManager` (for JDBC this is usually `DataSourceTransactionManager`).
        
    - The transaction manager grabs a connection from HikariCP and sets it into **transaction mode** (autoCommit=false).
        
    - This binds the connection + transaction to the current thread.
        
 3. **Your method runs**  
    Now the proxy calls your actual method.
    
    - Inside, if you hit the repo layer, queries use the same DB connection bound to the transaction.
        
    - But the DB does **not commit yet**.
        
 4. **Method finishes**
    
    - If no exception (or only checked exceptions): → Transaction manager calls `commit()` on the DB connection.
        
    - If a runtime exception (by default): → Transaction manager calls `rollback()`.
        
 5. **Connection released**  
    After commit/rollback, the connection is returned to HikariCP pool.
    

 ---

 ### 🔹 Important points

 - To create a `PlatformTransactionManager` bean,  Spring needs a **DataSource** bean.  
    That’s why transactions are always tied to a database connection.
    
 - If you mark a method as `@Transactional` unnecessarily:
    
    - Spring will still create and manage a transaction.
        
    - That means:
        
        - One connection is taken from the pool and held until the method finishes.
            
        - Overhead of `BEGIN` + `COMMIT`/`ROLLBACK` calls to the DB.
            

 👉 So best practice: **mark only those methods as `@Transactional` that really need transactional guarantees** (atomicity, rollback safety, multiple statements grouped together).
```
