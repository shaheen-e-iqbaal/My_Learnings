
-> Interceptor are the code segment which will get run before or after or around the target code.

-> like when you want to run some code before, after or around of certain type of methods, class..., then you can create interceptor for it.

-> there are two types of interceptors : 1. global level 2. class, annotation or method level.

-> global level interceptors are executed before even reaching to controller class.

-> second type of interceptors are executed when any specific class, or method is executed.

-> spring uses AOP to intercept this call.

ex : how to create global level interceptor.
```
//UserController.java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    User user;

    @GetMapping(path = "/getUser")
    public String getUser() {
        user.getUser();
        return "success";
    }
}

//MyCustomInterceptor.java
@Component
public class MyCustomInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("inside pre handle method");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
                           @Nullable ModelAndView modelAndView) throws Exception {
        System.out.println("inside post handle method");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                                @Nullable Exception ex) throws Exception {
        System.out.println("inside after completion method");
    }
}

//AppConfig.java
@Configuration
public class AppConfig implements WebMvcConfigurer {

    @Autowired
    MyCustomInterceptor myCustomInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myCustomInterceptor)
                .addPathPatterns("/api/*") // Apply to these URL patterns
                .excludePathPatterns("/api/updateUser", "/api/deleteUser"); // Exclude for these URL patterns
    }
}
```
-> postHandle will run after the request is processed. if there is any exception during request, it will not run. afterCompletion will run always no matter there is exception or not.
-> So MyCustomInterceptor will run before even request goes to controller class.
-> this type of interceptor can be used for logging, caching, authentication purpose.


ex: Class, method, annotation level interceptor.

-> when you want the call to certain type of class, methods to be intercepted, you can create this type of interceptors.
-> firstly create any custom annotation. mark the desired class, method, field with this annotation. then use AOP aspect and implement before or after or around advice for it.
```
//UserController.java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    User user;

    @GetMapping(path = "/getUser")
    public String getUser(){
        user.getUser();
        return "success";
    }
}

//MyCustomAnnotation.java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomAnnotation {
    String name() default "";
}

//User.java
@Component
public class User {

    @MyCustomAnnotation(name = "user")
    public void getUser() {
        System.out.println("get the user details");
    }
}


//MyCustomInterceptor.java
@Component
@Aspect
public class MyCustomInterceptor {    @Around("@annotation(com.conceptandcoding.learningspringboot.CustomInterceptor.MyCustomAnnotation)")
    public void invoke(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("do something before actual method");

        Method method = ((MethodSignature) joinPoint.getSignature()).getMethod();
        if (method.isAnnotationPresent(MyCustomAnnotation.class)) {
            MyCustomAnnotation annotation = method.getAnnotation(MyCustomAnnotation.class);
            System.out.println("name from annotation: " + annotation.name());
        }

        joinPoint.proceed();
        System.out.println("do something after actual method");
    }
}

output:
do something before actual method
name from annotation: user
get the user details
do something after actual method
```


Filter Vs Interceptor :

-> Filters runs before even request goes to any servlet like dispatcher servlet. all the filters run after one another. after filters complete, request will go to any servlet.
-> so filters will run for every request irrespective of it's path.
-> So filters are not specific to Spring webMVC. they are java concept.

-> Interceptors are spring stuff. global interceptors runs after request comes out from servlet and before going to any controller class.

Use cases :

-> Filters can be used for Authentication, global application level logging, caching, encoding/decoding, compression...
-> Interceptors can be used for API specific authorization, local level logging...

-> Read [This](https://javalaunchpad.com/understanding-the-difference-between-handler-interceptors-and-filters-in-spring-mvc/) article to understand filter Vs interceptor batter.


request -> servlet container (tomcat) -> multiple filters(filter chain) -> dispatcher servlet -> global interceptors -> local controller level interceptor (created using AOP) -> controller -> local API level interceptor (created using AOP) -> controller method -> then reverse....