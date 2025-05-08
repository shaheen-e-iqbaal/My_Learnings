
-> For each new tab, chrome creates separate process.

==**Difference in way of Using TCP between HTTP/1.1 and HTTP/2 :**==

### **HTTP/1.1**

in HTTP/1.1, each request requires a separate TCP connection unless **Keep-Alive** is used to reuse the connection.

	Establishing a TCP connection involves a **three-way handshake** (SYN, SYN-ACK, ACK), which adds latency. Closing the connection also requires additional packets (FIN, ACK).
    
    Without connection reuse, the overhead of opening and closing connections for every request would significantly degrade performance.
        
- **Keep-Alive and Connection Reuse:**
    
    - HTTP/1.1 introduced the `Connection: keep-alive` header to allow multiple requests to reuse the same TCP connection, reducing overhead.
    
    - However, even with Keep-Alive, browsers needed a way to balance **efficiency** and **fairness** when making multiple requests to the same server.

### **HTTP/2 and Multiplexing**

- **How HTTP/2 Works:**
    
    - HTTP/2 introduces **multiplexing**, which allows multiple requests and responses to be sent concurrently over a single TCP connection using **streams**.
        
    - Each request is assigned a unique stream ID, and the server can interleave responses for multiple requests on the same connection.
        
    - This eliminates the need for multiple TCP connections and avoids the head-of-line blocking problem in HTTP/1.1.



-> in HTTP/1.1, for each new HTTP request, new TCP connection is created between client and server.

-> in HTTP/2, there is only one TCP connection between client and server. all requests and responses are transmitted on this single TCP connection. this is called multiplexing.

-> If we are use HTTP/1.1, then for each HTTP request new TCP connection will be created. so if user wants, then he can intentionally crash server by making too much HTTP requests. to solve this problem, Browser such as Chrome limits the maximum of 6 TCP connection between single client and server. these 6 TCP connection will be used to make request and response.

