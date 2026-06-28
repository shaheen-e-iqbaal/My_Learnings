
-> PostgreSQL is designed around Process model instead of Thread model. i.e. for each new connection, central Postgres process (called Postmaster) will fork a new back-end child process. which is similar of how Google Chrome works. i.e. upon opening new tab, it will create a new process. i.e. each opened tab in chrome is separate process. but in spring boot, each new incoming request is handled by new thread. no new child process is created.

-> There is active talk in PostgreSQL community about the need of converting the postgres from Process model to Thread model. because forking new process for each new connection requires xMB memory and lot of CPU computation. but forking new Thread requires only xKB of memory. so it makes application so much light-weight.

-> Refer [This Image](https://share.google/QzAphKL1pK1ztG2xQ) of postgres architecture to understand it.


-> Now suppose you are running your backend application on 20 EC2 instances. you are also using Connection pool like HikariCP per application instance. in that pool, you have mentioned the size of connection pool as 50. so, now, at peak load time, the total number of active connections database is going to handle is = 50 * 20 = (Per instance total connection * total instance) = 1000. so, Postmaster will create 1000 active connection to handle your load. now imagine, AWS auto increments your EC2 instances to handle the traffic to 100 instances. now total DB connection during Peak time will be = 50 * 100. which will be 5000. which will result into forking of 5000 postgres child processes. considering each process takes 5 MB of space, total space requirement will be = 5 MB * 5000 = 25000 MB. which is huge and impractical to have in production. which can eventually crash your DB server.

-> To solve this problem, Developer introduced a new level of connection pooling between application servers and DB server. all instances of our backend server will now establish connection with this new intermediate pool. this pool will create and store the actual DB server connection. now, our backend servers will ask this pool for actual DB connection and will get from there. this pool will limit the maximum number of active DB connection for any given time. so now, no matter how much EC2 instances you have, it won't affect to DB.

-> On of the example of this intermediate connection pool is pgBouncer. 


ChatGPT Version :
```
 -> **PostgreSQL uses a process-based architecture, not a thread-based one.**  
 For every new client connection, the central PostgreSQL process (called the **postmaster**) **forks a new backend child process**. Each connection is handled by its own OS process. This is conceptually similar to **Google Chrome’s multi-process model**, where opening a new tab creates a separate process.  
 In contrast, applications like **Spring Boot** usually follow a **thread-per-request model**, where a fixed number of processes exist and each incoming request is handled by a new thread within the same process.

 -> **There has been ongoing discussion in the PostgreSQL community about moving from a process model to a thread model**, mainly due to resource usage concerns.  
 Forking a new process for each connection consumes **more memory (MBs per process)** and incurs higher OS overhead compared to creating a thread, which typically requires only **KBs of stack memory**. A threaded model could theoretically reduce memory usage and improve scalability.  
 However, PostgreSQL’s process-based design was a **deliberate architectural choice** to gain better **isolation, stability, and simpler memory management**, and changing this model would require a massive redesign. So far, PostgreSQL has continued with the process model.

 -> **Refer to the PostgreSQL architecture diagram** (linked image) to visualize this clearly.  
 You will see a single **postmaster process** at the top, and multiple **backend processes** underneath it—one for each active client connection.

 -> **Now consider a real-world deployment scenario.**  
 Suppose your backend runs on **20 EC2 instances**, and each instance uses **HikariCP** with a connection pool size of **50**.  
 At peak load:

 - Total database connections = `20 × 50 = 1000`   
 - PostgreSQL will fork **1000 backend processes**  

 Now imagine **AWS Auto Scaling** increases your backend to **100 EC2 instances**:

 - Total database connections = `100 × 50 = 5000`  
 - PostgreSQL now has **5000 backend processes**  

 If each backend process consumes roughly **5 MB of memory**:

 - Total memory usage ≈ `5000 × 5 MB = 25 GB`   

 This level of memory consumption is **huge and often impractical**, and can easily **exhaust RAM**, trigger heavy swapping, or even **crash the database server**.


 -> **To solve this problem, an additional layer of connection pooling is introduced between application servers and the database.**  
 Instead of every application instance directly opening many database connections, all application servers connect to an **intermediate connection pooler**.  
 This pooler:

 - Maintains a **limited number of actual PostgreSQL connections** 
 - Reuses those connections across many client requests
 - Acts as a gatekeeper so the database never sees an unbounded number of connections  

 From the application’s point of view, it still “gets a connection,” but that connection may be **logically multiplexed** over a smaller set of real database connections.


 -> **One popular example of such an intermediate pooler is PgBouncer.**  
 PgBouncer dramatically reduces the number of PostgreSQL backend processes by:

 - Allowing thousands of client connections   
 - While keeping only hundreds (or fewer) real DB connections open. This makes PostgreSQL **far more scalable and stable** in cloud environments with auto-scaling application servers.
```

