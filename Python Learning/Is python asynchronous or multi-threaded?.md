
Use [This article](https://medium.com/pythons-gurus/understanding-concurrency-and-parallelism-in-python-a-comparative-guide-with-java-and-c-6167f3732b15) to understand how concurrency and parallelism can be achieved in languages like python, Java


-> in python, we can create multiple threads, but due to GIL, only one thread can access the byte code at a time. so parallelism can't be achieved using multi-threading in python.
so for CPU intensive task, multi-threading is not an option in python. we can use multiprocessing library to achieve parallelism. which bypasses GIL and creates separate process.

-> Python framework FastAPI is asynchronous. while Django and Flask are synchronous.

==->Use [This](https://stackoverflow.com/questions/71516140/fastapi-runs-api-calls-in-serial-instead-of-parallel-fashion) article to learn about synchronicity of FastAPI.==

==-> Use [This](https://stackoverflow.com/questions/77935269/performance-results-differ-between-run-in-threadpool-and-run-in-executor-in/77941425#77941425) and [This](https://sentry.io/answers/fastapi-difference-between-run-in-executor-and-run-in-threadpool/) to learn about difference between run_in_executor() and run_in_threadpool() methods of FastAPI.==