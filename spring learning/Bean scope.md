
-> Request, Session beans are connected with http request and session.
-> request bean will be only created and managed for each new http request and session bean will also be created and managed for each new http session.
-> after request gets completed, beans with request scope gets destroyed by IoC. similarly for session bean.

-> if there is no http request or active session, these beans won't be created. so if you have dependency of these beans inside singleton scope class, then during application initialization, you will get error. because there is no http request during application startup, so request or session beans won't be created.

-> request, session and prototype are by default lazy initialized. so even there is http request or active session, the bean won't be created until it is not required.

-> if you want to have dependency of request, session bean inside singleton scope class, then there are two ways to achieve it. 1. mark the dependency as Lazy. so spring will inject proxy of CGLIB type instead of real bean. 2. add @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS). in this case, spring will also inject the proxy of CGLIB type. to inject proxy of dynamic JDK type, you can replace TARGET_CLASS with INTERFACES.


-> Beans of Prototype scope are not managed by IoC after their creation. spring will create them, use it and drop. so for any new requirement, new bean will get created.

-> If you have marked any dependency as Lazy, spring will inject proxy instead of real bean during dependency resolution.
ex :
@Component
public class User{
   Account account;
   User(@Lazy Account account){
   this.account = account;
   }
}
for above account dependency, irrespective of either bean of Account is there in IoC or not, spring will always inject proxy.


-> if you have dependency of Account which is not marked as lazy, but the Account class is marked as Lazy. spring will create bean of Account and will inject it into account dependency. it won't inject proxy this time irrespective of the scope of Account. (Note that scope of Account can only be Singleton or prototype. why not request and session?)

ex: 
@Component
public class User{
   Account account;
   User(Account account){
   this.account = account;
   }
}
@Component
@Lazy
public class Account{

}
