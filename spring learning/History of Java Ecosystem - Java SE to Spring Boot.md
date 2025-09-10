

### Java SE (Standard Edition)

- It is the **core** of the Java platform.
- Supports core programming concepts like:
    - **OOP**, collections, **exception handling**
    - **Multithreading**, **networking**, **I/O streams**
- Used to build **console-based** or **desktop applications** (e.g., Swing, JavaFX).
- Foundation for **all other Java platforms** (Java EE, Jakarta EE, Spring, etc.)


Java EE (Enterprise Edition) : now called Jakarta EE

-> it built on top of the Java SE.
-> It included technologies like : JDBC, JPA, Servlet etc.
-> Using it, we can create enterprise applications. but it requires too much boiler plate code, configuration etc.


Spring :

-> It got developed as an alternative to Java EE. 
-> It added concepts like, Spring Core (IoC/DI container), Spring webMVC (also called Spring MVC), Spring AOP, Spring JDBC etc.
-> It has reduced the boiler plate code and also improved the project structure.
-> the problem with spring is that, still we have to write configuration classes like 
WebConfig.java, DataConfig.java, SecurityConfig.java, AppConfig.java.

**WebConfig.java : we have to make @Bean of DispatcherServlet, ViewResolver, MappingJackson2HttpMessageConverter (to map JSON to Java object and vice versa)...**
**DataConfig.java : we have to make @Bean of DataSource (to manage db connection, default one is HikariCP.), JdbcTemplate, PlatformTransactionManager (to use @Transactional)** 


Spring Boot :

-> It is built on top of the Spring.
-> It provides convention over configuration.
-> it provides better dependency management. we don't need to include all the dependencies with their compatible versions manually. spring boot will do it. we just need to mention which dependencies we need. ( NOTE : spring boot will do it for the dependencies which are managed by itself like web, data JPA, security etc. but when you want to add dependencies like jjwt-api, jjwt-imp.... then we have to mention it's version also. dependencies list in spring initializer websites are managed by spring boot.)
-> it provides auto configuration. we don't need to manually provide configuration for Web config, data config...
-> it also has embedded servlet container like tomcat. so we don't required to build WAR and deploy separately like we do in spring environment.

-> when we add spring-boot-starter-web dependency in spring boot project, it automatically includes : 

|Dependency|Purpose|
|---|---|
|**spring-core**|Core container (IoC, beans)|
|**spring-web**|General web capabilities|
|**spring-webmvc**|MVC framework (REST, controllers, views)|
|**jackson-databind**|JSON serialization/deserialization|
|**spring-boot-starter-tomcat**|Embedded Tomcat server|
|**spring-boot-starter-logging**|Logging via Logback|


Flow of Request in Spring Boot :

### 1. **Client sends HTTP request**

- A user hits a URL via a browser or Postman
### 2. **Embedded Servlet Container (like Tomcat) receives the request**

- Spring Boot uses an **embedded Tomcat** server by default.
    
- The request is received on a **port (e.g., 8080)** bound by the server.

#### 🧵 When is a thread created?

- Tomcat uses a **thread pool** (default: 200 threads).
    
- A **new request is handled by a free thread** from the pool.
    
- If all threads are busy, the request waits in the **request queue**.
    

> 📌 So, the thread is picked _immediately_ when the request arrives and assigned to handle the lifecycle of the request-response.

### 3. **Request is handed over to the `DispatcherServlet`**

- `DispatcherServlet` is the **front controller** of the Spring MVC framework.
    
- It is registered automatically at startup via `@SpringBootApplication`.
    

> Think of `DispatcherServlet` as the "traffic controller" for all incoming requests.

### 4. **DispatcherServlet consults HandlerMapping**

- Spring has one or more `HandlerMapping` implementations.
    
- These are used to find **which controller and method** should handle this request.
### 5. **Controller method is invoked**

- The actual method in your `@RestController` or `@Controller` is called.
    
- If it's a REST API, it returns a Java object (like `String`, `ResponseEntity`, or POJO).
### 6. **Response is processed by ViewResolvers or HttpMessageConverters**

- If returning HTML (e.g., Thymeleaf), a `ViewResolver` resolves it.
    
- If returning JSON/XML (REST APIs), `HttpMessageConverter` serializes the Java object to JSON or XML.
### 7. **Thread is released**

- The thread is returned to the **Tomcat thread pool**, ready for the next request.
