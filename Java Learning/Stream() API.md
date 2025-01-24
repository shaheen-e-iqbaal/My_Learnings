
-> Stream can be made in different ways. one of famous way is :

    for Collections :  arr.stream() will return the stream
    for Arrays : Arrays.stream() will return the stream.
	for String : string_name.chars() will return IntStream made from                     ascii values of characters of string.

	ex: List<Person> ls = Arrays.asList(p1, p2, p3, p4);
		Stream<Person> st = ls.stream();

	ex: Person[] arr = {p1, p2, p3, p4};
		Stream<Person> st = Arrays.stream(arr);

	ex: String s = "Absdkfele";
	    IntStream is = s.chars();
	    Stream<Integer> st = s.chars().boxed();


-> for primitive data, it will return corresponding primitive stream. like for int, it will return IntStream, for long, LongStream, for Double, DoubleStream.

	ex: int[] arr = {1,2,3,4,5};
		IntStream is = Arrays.stream(arr);

-> to convert the IntStream into normal stream, we can use boxed() method, which will convert primitive to corresponding wrappers.

     int[] arr = {1,2,3,4,5};
     Stream<Integer> st = Arrays.stream(arr).boxed();


-> after creating stream, we can apply zero or more than zero intermediate operations.

-> once a terminal operation is applied on a stream, that stream gets consumed, means, that stream can't be reused.

-> Intermediate operations are lazy, it means they will be applied once the terminal operation gets applied.

-> parallelStream is faster than normal stream as it performs operations concurently.

-> to make parallelStream for collections, use parallelStream() method instead of normal stream() method. and for arrays, use Arrays.stream(arr).parallel().

The primary difference between `stream()` and `parallelStream()` lies in **how they process data**:

- **`stream()`**: Processes data sequentially, one element at a time, in a single thread.
- **`parallelStream()`**: Processes data in parallel, dividing the work among multiple threads to improve performance on large datasets or computationally intensive operations.

### **Key Differences in Working of Stream & ParallelStream**

#### 1. **Threading Model**

- **`stream()`**:
    
    - Operates on a **single thread** (the main thread or the thread that initiates the stream).
    - All operations (e.g., filtering, mapping, collecting) are executed sequentially in the order of elements in the stream.
- **`parallelStream()`**:
    
    - Operates on **multiple threads** using the **ForkJoinPool** (a default thread pool in Java).
    - Splits the stream's data into chunks and processes each chunk on a separate thread.
    - Does not guarantee the order of processing or the output unless explicitly controlled.

---

#### 2. **Performance**

- **`stream()`**:
    
    - Suitable for small datasets or operations where performance is not a bottleneck.
    - Less overhead because no thread management is involved.
- **`parallelStream()`**:
    
    - Suitable for **large datasets** or **CPU-intensive tasks** where dividing the workload can reduce processing time.
    - Has some overhead due to thread management (e.g., splitting the data, managing threads, and combining results).

---

#### 3. **Order of Processing**

- **`stream()`**:
    
    - Maintains the **order of elements** in the input source (e.g., a list or array).
    - Use `.forEachOrdered()` for predictable results.
- **`parallelStream()`**:
    
    - May **not maintain the order** of elements during processing.
    - Use `.forEachOrdered()` if you want to preserve the order at the cost of performance.

---

#### 4. **Execution Model**

- **`stream()`**:
    
    - A **linear execution model**: Each operation processes all elements before moving to the next operation.
    - Example:
        
        `List<Integer> list = Arrays.asList(1, 2, 3, 4, 5); list.stream().filter(x -> x > 2)       // Filters elements > 2     .map(x -> x * 2)         // Doubles the filtered elements     .forEach(System.out::println); // Prints 6, 8, 10`
        
- **`parallelStream()`**:
    
    - A **divide-and-conquer model**:
        - The stream is split into multiple parts.
        - Each part is processed independently in parallel.
        - The results are combined.
    - Example:
        
        `list.parallelStream().filter(x -> x > 2).map(x -> x * 2)     .forEach(System.out::println); 
        // May print in any order, e.g., 8, 10, 6`
        

---






### **Intermediate Operations**

1. `filter(Predicate<T>) : Filters elements based on a condition.`
2. `map(Function<T, R>) : Transforms each element to another type.`
3. `flatMap(Function<T, Stream<R>>) : Flattens a stream of streams into a single stream.`
4. `distinct() : Removes duplicate elements.`
5. `sorted() : Sorts elements in natural order.`
6. `sorted(Comparator<T>) : Sorts elements using a custom comparator.`
7. `peek(Consumer<T>) : Performs an action on each element (useful for debugging).`
8. `limit(long maxSize) : Truncates the stream to the first n elements.`
9. `skip(long n) : Skips the first n elements of the stream.`

---

### **Terminal Operations**

1. `collect(Collector<T, A, R>) : Collects elements into a collection or container.`
2. `forEach(Consumer<T>) : Iterates over each element and performs an action.`
3. `reduce(BinaryOperator<T>) : Combines elements into a single value.`
4. ==`toArray() : Converts the stream into an array. will return the array of type 'Object' if the stream is of any objects. otherwise if the stream is of primitive like IntStream, it will return array of primitive like int[].`==
5. `toArray(size -> new T[size]) : this will return the array of type T not of the type Object.`
6. `findFirst() : Returns the first element in the stream.returns Optional<T>.`
7. `findAny() : Returns any element from the stream (useful in parallel streams).`
8. `count() : Counts the number of elements in the stream. returns long.`
9. ==`sum() : Returns the sum of elements of stream. it can only be applied to IntStream.`==
10. `anyMatch(Predicate<T>) : Checks if any element matches a condition.`
11. `allMatch(Predicate<T>) : Checks if all elements match a condition.`
12. `noneMatch(Predicate<T>) : Checks if no elements match a condition.`
13. `max(Comparator<T>) : Finds the maximum element based on a comparator. it returns Optional<T>`
14. `min(Comparator<T>) : Finds the minimum element based on a comparator. it returns Optional<T>`
15. `reduce(seed, (acc, i) -> {}) : it reduces the stream and stores result in acc. seed represents the starting value of acc and i represent elements of stream.`

---

### **Specialized Stream Methods**

1. ==`mapToInt(ToIntFunction<T>) : Maps elements to an IntStream==.`
2. `mapToDouble(ToDoubleFunction<T>) : Maps elements to a DoubleStream.`
3. `mapToLong(ToLongFunction<T>) : Maps elements to a LongStream.`
4. ==`boxed() : Converts a primitive stream to a stream of objects==.`
5. `parallel() : Converts a stream to a parallel stream.`





Collectors.groupingBy() :

ex : we have a list of employee. we want to group this employee on the basis of their department.
-> Collectors.groupingBy() : returns the Map with key as a department, value as a list of employee present in that department.

ans : Map<String, List<Employee>> ans = employees.stream().collect(Collectors.groupingBy((Employee employee) -> employee.department));

Collectors.groupingBy()  takes maximum three arguments. first is a lambda function which called classifier which denotes the criteria on which we want to group. in our example, we want to group on basis of department. 

The second argument it takes is downstream which is also Collectors. which is used when we want to perform some action on the list we got for each department.

ex; suppose for each department, we want value as a average age of employee of that department.

ans : Map<String, Double> ans = employees.stream().collect(Collectors.groupingBy((Employee employee) -> employee.department, Collectors.averagingDouble(e -> e.age)));

ex : suppose we want to count sum of salary of employee of each department.

ans: Map<String, Long> ans = employees.stream().collect(Collectors.groupingBy((Employee employee) -> employee.department, Collectors.summingToInt((Employee e) -> e.salary)));

ex; suppose we want to count the number of employee in each department.

ans : Map<String, Long> ans = employees.stream().collect(Collectors.groupingBy((Employee employee) -> employee.department, Collectors.summingToInt((Employee e) -> 1)));






Collectiors.partioningBy():

-> it partitions the elements of stream based on the given predicate.

ex; suppose we want to partition employees having salary >= 25000.

ans : Map<Boolean, List<Employee>> ans = employees.stream().collect(Collectors.partitioningBy((Employee e) -> e.salary >= 25000));

->ans.get(true) will return the list of employee having salary >= 25000 and ans.get(false) will return list of employee having salary less than 25000.

-> If we want to perform operations on List<Employee> like we performed in groupingBy(), we can use same methods here also.



Collectors.toMap() :

-> It returns the map.

ex: suppose we want to create map where key = department, value = sum of salary of employee of that department.

ans : Map<String, Double> ans = employees.stream().collect(Collectors.toMap((Employee e) -> e.department, (Employee e) -> e.salary, (Integer oldValue, Integer newValue) -> oldValue + newValue, () -> new TreeMap<>()));

-> The first function is about specifying key.
-> second is about specifying value for that key.
-> third is called when there is a Collison for that key. it will have two arguments, one is present value corresponding to that key, and other is new value. 
-> in our example, oldValue is the salary which is already there for that key, newValue is the salary of the new employee whose department is also same. so in that function, we are summing both salary.

-> forth function is about specifying type of return map we want to return it.














