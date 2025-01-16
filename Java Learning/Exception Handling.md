
1.  A try block must be followed by catch or finally of both. if you haven't added the catch block after the try block, then it means that you haven't handled the exception thrown by try block. in this case, you must have to mention the "Throws someException" in the method signature. (for better understanding refer the Head First Exception handling Chapter page 444.)

2.  A method can throw multiple exceptions. so you can write multiple catch block after the try block to handle the different kind of exceptions. (for better understanding refer head First page 435 to 439).

3.  You can throw exception from catch and finally block also. for better under standing of finally block exception, refer this link. https://stackoverflow.com/questions/2911215/what-happens-if-a-finally-block-throws-an-exception

4. we can have return statements in try, catch and finally block also. but to understand how does that work, use this link. https://stackoverflow.com/questions/65035/does-a-finally-block-always-get-executed-in-java

5. Throwable is the superclass of Exception and Error class in java. We can Catch Errors also just like exceptions. but it is always advised not to do because error indicates that there is a serious problem in your code which should be resolved.



                             Throwable


                        Exception             Errors
                                            Ex: OutofMemoryError
                                                StackOverflowError
			Unchecked       Checked
Ex: NullPointerException                Ex : IOException
IndexOutOfBoundException                 SQLException