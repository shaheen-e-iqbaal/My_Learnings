
-> In asynchronous, we have single thread, which performs all operations. for all I/O bound task, it uses event loop and task queue. 

-> in multi-threading, we can have multiple threads, which can perform operations on different cores.

-> In general, multi-threading is a good choice for CPU intensive application. and Asynchronous is good for I/O bound application. so using multi-threading we can achieve parallelism on multi-core CPU.

-> but for python, due to the concept of GIL, multi-threading doesn't let to achieve parallelism, to achieve it, we have to use multiprocessing library.

-> to handle concurrent requests, Fast API uses application server named uvicorn which is based on Asynchronous architecture. by default there is only one worker thread being run by uvicorn. but to improve application performance we can increase worker thread count.

-> to handle concurrent requests, the default application server for spring boot application is tomcat. which uses thread pool to serve incoming concurrent requests. each thread from a thread pool will serve one request. by default there is 200 thread in a tomcat thread pool.


==Use [This article](https://medium.com/pythons-gurus/understanding-concurrency-and-parallelism-in-python-a-comparative-guide-with-java-and-c-6167f3732b15) to understand how concurrency and parallelism can be achieve in languages like python, Java==