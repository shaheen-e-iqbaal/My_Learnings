
=> ==During application startup, spring will find all the class whose proxy needs to be created. so spring will create those proxies during application startup only and will store them in IoC. this proxies will be there in IoC during whole application lifecycle.==

=> ==All the Proxy in spring are by default Singleton irrespective of the Scope of the corresponding class. they will be created during application startup no matter what the scope of the corresponding class is. these proxies will figure out the bean creation as per the scope of target bean.==


When will spring find out that it needs to create a proxy this class? :

1. When you mark a class with @Scope( value = "scope_type", proxyMode = ScopedProxyMode.proxy_type). spring will use this proxy whenever it needs to inject bean of this class. actual bean creation will depend on the scope of this class.
2. When any of aspect advice gets applied to a class or any of it's method, spring will create a proxy of that class.
3. When you mark either any method or class with @Transactional annotation, spring will create proxy of that class.
4. When any method is marked with @Async, in that case also proxy of that class will be created.
5. and many more....


Flow of Spring Boot application start up :

- **SpringApplication.run()** starts.
    
- Active profiles are set (or default is used).
    
- Application context is created (`AnnotationConfigServletWebServerApplicationContext`).

- Scan for @Aspect annotated class. if it finds any, will parse the pointcuts present in it.

- Component scanning finds `@Component`, `@Service`, `@Controller`, @Configuration etc.
    
- Bean definitions are registered (note that it is registered not created), proxies of relevant classes (those who follow conditions like mentioned above) are created.
    
- Servlet container (e.g., Tomcat) is started.
    
- `DispatcherServlet` is registered and initialized.
    
- Beans are instantiated and dependencies are injected. if proxy of target class is in IoC, it will be injected instead of actual bean.


Types of Proxy :

1. Dynamic JDK proxy : If a class implements one or more interfaces, then spring will by default use this proxy. spring will create a new class, which will also implement the same interfaces implemented by original class and will override methods. 
ex : 
//Interfaces
public interface Action {
    void doAction();
}

public interface Extra {
    void doExtra();
}

//Implementing classes
@Component("implOne")
@Scope(value = "singleton", proxyMode = ScopedProxyMode.INTERFACES)
public class ActionImplOne implements Action, Extra {
    public ActionImplOne() {
        System.out.println("ActionImplOne instantiated");
    }

    @Override
    public void doAction() {
        System.out.println("ActionImplOne: Doing action");
    }

    @Override
    public void doExtra() {
        System.out.println("ActionImplOne: Doing extra");
    }
}

@Component("implTwo")
@Scope(value = "singleton", proxyMode = ScopedProxyMode.INTERFACES)
public class ActionImplTwo implements Extra {
    public ActionImplTwo() {
        System.out.println("ActionImplTwo instantiated");
    }

    @Override
    public void doAction() {
        System.out.println("ActionImplTwo: Doing action differently");
    }
}


=> For the above ActionImplOne class, spring will create proxy class which will also implement Action and Extra interfaces. similarly, for the second ActionImplTwo, proxy class will be created and will implement Action interface.

Now, how to use above two classes as a dependency in some other class? see below code.

@Component
public class ClientRunner {

    private final Action action;
    private final Extra extra;

    public ClientRunner(
        @Qualifier("implTwo") Action action, // ActionImplTwo class proxy injected.
        @Qualifier("implOne") Extra extra  //ActionImplOne class proxy injected. 
    ) {
        this.action = action;
        this.extra = extra;
    }

    public void run() {
        action.doAction();   // Works via proxy. will call implementation of ActionImplTwo
        extra.doExtra();     // Also works via proxy. will call implementation of ActionImplOne
    }
}


2.  **CGLIB (Code Generation Library) Proxy in Spring (Simplified)** 

- Spring uses **CGLIB proxying** **by default** if your **bean class doesn't implement any interface**.
    
- CGLIB creates a **proxy class** that **extends your actual class** (i.e., it becomes a subclass of your class).
    
- This proxy class **overrides** all **non-final, non-static, and non-private methods** from your class.

### **What actually happens**

- When you call any method on the **proxy object**, Spring intercepts that call.
    
- If the method is **overridden in the proxy**, Spring can **add logic** (like AOP) and then forward the call to the original method in the **target object**.
    
- This means:  
    → **Call goes to proxy → proxy overrides method → proxy internally calls actual method in your bean.**

### **Methods that are NOT overridable (so, NOT proxied)**

- `private` → Not accessible to proxy class, so not overridden
    
- `static` → Belongs to class, not instance, so can't be overridden
    
- `final` → OOP rule: final methods can't be overridden

→ So, these methods are **called directly without any proxy logic**.