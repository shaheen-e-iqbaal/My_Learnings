
It is combination of multiple annotations. main three among them are :
1. @SpringBootConfiguration
2. @ComponentScan
3. @EnableAutoConfiguration


1.@SpringBootConfiguration :
-> It marks this class as spring boot configuration class. marks the class as a source of bean definitions.
-> It uses @Configuration.

2.@ComponentScan :
-> Tells Spring to scan for components (`@Component`, `@Service`, `@Repository`, `@Controller`, etc.)  starting **from the package of the main class downward**.

@SpringBootApplication
public class MyApp {
  public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
  }
}
If `MyApp` is in package `com.example`, then Spring will scan:

- `com.example`
    
- `com.example.service`
    
- `com.example.controller`, etc

-> You can override base packages using:
	@ComponentScan(basePackages = {"com.other.package"})



3.@EnableAutoConfiguration :

-> This is the main annotation which does make our life easy by enabling multiple configuration by itself.
-> It enables one of the benefit of using spring boot over spring.
-> without this annotation, You'd write:

- `@Configuration` class for:
    
    - `DataSource`
        
    - `EntityManagerFactory`
        
    - `TransactionManager`
        
- Add `@EnableWebMvc` and configure `ViewResolvers`
    
- Register Spring in your web server using `web.xml` or `WebApplicationInitializer`
    
- Manually wire everything — including 3rd party configs like Thymeleaf, Jackson, Swagger, etc.