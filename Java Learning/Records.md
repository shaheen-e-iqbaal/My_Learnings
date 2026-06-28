
-> Records in java are special type of class with all fields as final, private.
-> the main use case of records is for DTO. before records, we used to create POJO classes with lots of boiler plate code like constructor, getter/setter, toString()... but by using records, we can eliminate all these boiler plate.

-> While using records as Request/Response DTO for API, always use Wrapper classes for primitive types like Integer for int, Double for double. the reason is, if the values of this fields are not passed by user/request, then these fields will be initialized as null if used wrapper otherwise default values if used primitives i.e. 0 for int, false for boolean... so by using wrapper, we get to know if the value is passed or not.

ex :
`public record User(Integer id, String name, Integer age){}`

-> id, name, age :  all are by default private and final. it automatically provides methods like user.id(), user.name()... to access values of these fields.

-> by declaring above User record, we will get all args constructor, separate method to get value for each variable, default implementation of equals(), hashCode(), toString() methods.below will be the generated class for above record.

```
public final class User extends java.lang.Record {
    
    // 1. Private final instance fields
    private final Integer id;
    private final String name;
    private final Integer age;

    // 2. Canonical constructor
    public User(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    // 3. Public accessor methods (No "get" prefix)
    public Integer id() {
        return this.id;
    }

    public String name() {
        return this.name;
    }

    public Integer age() {
        return this.age;
    }

    // 4. Value-based equality method
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
			if (!(o instanceof User user))
			return false;
        return this.id == user.id && 
               this.age == user.age && 
               java.util.Objects.equals
               (this.name, user.name);
    }

    // 5. Hash code generation method
    @Override
    public int hashCode() {
        return java.util.Objects.hash(id, 
        name, age);
    }

    // 6. String representation method
    @Override
    public String toString() {
        return "User[id=" + id + ", name=" + 
        name + ", age=" + age + "]";
    }
}
```

-> if you want to use record as DTO in spring boot with some extra validation, then you can add like below :

```
public record UserDTO(
	@Positive
	Integer id,
	
	@Email
	String emailId,
	
	@Min(18)
	Integer age;
){}
```


Compact constructor : if we want to do more deep validation, then we can use this.
```
public record RequestPayload(Integer top, Integer skip){
	
	public User{
		if(top == null)throw exception;
		if(skip == null)skip = 0;
	}
}

```

-> so you can use this compact constructor to do more specific validation, modify the incoming request payload like generate ID if not available...
-> but notice that, you don' t have to do like this.id = ... java will do it for you. you simply assign variable it's value. after compact constructor, java will call the constructor with new values.

-> you can provide your own all args constructor. in this case, java won't create the default all args constructor. you can overload the constructors. below is sample record with custom constructor. important rule is that, when you create constructor other that all args, then you must have to call all args constructor at last. all args constructor is called canonical constructor. so from non-canonical constructor, you have to call canonical one.

```
public record UserDTO(
	@Positive
	Integer id,
	
	@Email
	String emailId,
	
	@Min(18)
	Integer age;
 ){
	public UserDTO(Integer id,
	 String emailId,Integer age)
	{
		if(id == null)id = Math.ramdome();
		if(age < 40)throw exception;
		
		this.id = id;
		this.emailId = emailId;
		this.age = age;
	}
	
	public UserDTO(String emailId, int age){
		this(null, emailId, age);
	}
}
```

