
-> There is inheritance relationship between application.properties and applicaton-{profile_name}.properties. so all the fields present in first one will be also there in second. you can override them as well.

-> Bean of any class will be created only if active profile matches with the profile of that class.

-> If a class is **not annotated with `@Profile`**, it belongs to the **default profile**.  
Beans from such classes are **created in all environments**, regardless of which profile is active.

->there are two ways to make any environment active. 1. add spring.profiles.active = profile_name. 2. using command mvn spring-boot:run -Dspring-boot.run.profiles=profile_name. 

-> you can't have dependency of a class of different profile in your class.
ex:
@Component
@Profile("dev")
public class User{
}

@Component
public class Employee{
  User user;
  Employee(User user){
  this.user = user.
  }
}

above code will fail if the current active profile is not dev. if active profile is dev, then IoC will create bean of both Employee and User. it won't break in that case.


- ✅ **Inheritance between `application.properties` and `application-{profile}.properties`:**
    
    - `application.properties` is the **base configuration**.
        
    - `application-{profile}.properties` **inherits** from it and can **override** any values.
        
    - Spring **first loads** `application.properties`, then overrides with `application-{activeProfile}.properties`.
        
- ✅ **ways to activate a profile:**
    
    - **Inside config file (application.properties file):**
        
        `spring.profiles.active=dev`
        
    - **Via command line (e.g., Maven):**
        
        `mvn spring-boot:run -Dspring-boot.run.profiles=dev`
        
    - ✅ You can also use environment variables or VM options:

        `java -jar app.jar --spring.profiles.active=dev`
        
- ✅ You **can** inject dependencies from different profiles, **but only if those beans are active** in the current profile.
    
    - If a dependency bean is annotated with `@Profile("prod")` and you're running with `dev`, it **won't be available**, and Spring will throw a `NoSuchBeanDefinitionException`.