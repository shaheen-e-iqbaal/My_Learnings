
Need: 
	when we require only one object of any class throughout the application, then we design that class as a singleton class.
	 ex. The class responsible to make database connection need to be singleton.


Steps: 
	1. Mark the constructor of singleton class as private so no one can make object using new keyword.
	2. declare one static method which returns the object of that class.
	


Different Approaches to make Singleton class:

	1. Eager Initialization:
	  class Singleton_class{
		  private static final Singleton_class obj = new                           Singleton_class();
		  //Make constructor as a private.
		  private Singleton_class(){}
           public static Singleton_class getObject(){
             return obj;
           }
	  }

	Disadvantage : if we doesn't require the object, in that case also      it is making the object. so it is aquiring space unnacessary in that     case.


     2. Lazy Initialization:
     class Singleton_class{
		  private static Singleton_class obj;
		  //Make constructor as a private.
		  private Singleton_class(){}
           public static Singleton_class getObject(){
             if(obj == null){
                 obj = new Singleton_class();
             }
             return obj;
           }
	  }

	Disadvantage : when multiples threads execute the method                 simultaneously than in that case two objects are being made.             which is opposite of Singleton class principle.

	3. Lazy Initialization with locking:

	class Singleton_class{
		  private static Singleton_class obj;
		  //Make constructor as a private.
		  private Singleton_class(){}
           synchronized public static Singleton_class getObject(){
             if(obj == null){
                 obj = new Singleton_class();
             }
             return obj;
           }
	  }

	Disadvantage: it removes the drawback of previos approach, but due      to locking, it makes application bit slow.


	4. Bill pugh:
	  class Singleton_class{
		  
		  //Make constructor as a private.
		  private Singleton_class(){}
		  private static class helper{
			  private static final Singleton_class obj = new                          Singleton_class();
		  }
          public static Singleton_class getObject(){
             return helper.obj;
           }
	  }

    This approach utilises the two main feature of java to solve            the drawbacks of above mentioned approach.
         1.Inner class doesn't get loaded by JVM until they are                    referenced.
            this feature helps to get lazzy initialization i.e. we will             create object when we require only.
         2.A class can be loaded only once by JVM.
            this feature rescu from the drawback of making multiple                  instances by different threads simultaneously.

     5.Enum : 

		public enum Singleton{
			INSTANCE;
			 //Declare the other methods, variables etc.
		}