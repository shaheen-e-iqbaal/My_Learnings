
-> When you mark any component as @Lazy, spring will not create it's bean until it is required. so, when will it's bean will be required? suppose you have dependency of this component in some other component, then to resolve dependency, spring will create a bean of it, and will inject it. so in this case, lazy will not work.

-> If you mark any dependency as @Lazy, spring will always inject a proxy no matter bean of that dependency is available in IoC or not.
ex: 
@Component
public class A{
 B b;
 A(@Lazy B b){this.b = b;
 }
}

in above example, spring will inject proxy of B not the actual Bean.