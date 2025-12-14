
properties: 

1. Enum has by default private constructor. if we make custom constructor then it also must be a private.
2. Enum by default extends a java.lang.Enum class. so we can't extend any other class.
3. Enum can implement multiple interfaces.
4. Enum can have variables, methods just like any normal class.
5. All CONSTANTS of Enum are automatically static and final.
6. Enum can have abstract method also. but that method must me implemented by all CONSTANTS.
7. No class can extend Enum.


Ex: 
          public enum Days{
	          MONDAY(2, "any_string"),
	          TUESDAY,
	          FRIDAY;

				 private Days(int total_days, String any_string){
					 this.total_days = total_days;
					 this.any_string = any_string;
				 }
				 private Days(){}
            int total_days;
            String any_string;
            public void print(){
                 System.out.println("Hi from Enum");
            }
             //getter and setter for total_days and any_string.
          }

In above example, MONDAY, TUESDAY, FRIDAY are CONSTANTS. they are equivalent to following:
		 public static final Days MONDAY = new Days();

so, basically they are object of the same enum.

we can access any method or variable of enum in following way.

Days.MONDAY.print(); 

OR

Days day = Days.MONDAY;
day.print();

-> Enums can't be broken by Reflection. bcz when you do 