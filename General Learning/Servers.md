
Web Server :
-> some famous web server are Nginx, Apache HTTP server, Microsoft ssi.


Functionality of web server :

-> The main functionality is to serve static contents.

   **Can handle concurrent client connections**
   **Reverse proxying**
- **Load balancing**
- **Caching**
- **SSL/TLS termination**
- **Rate limiting**
- **Static file serving**
- **Security features**


Application server :

ex : Tomcat, Jettly, NodeJs server.

-> used to serve dynamic data. it can access database, APIs and other services to serve dynamic data.


Origin server : 

Let’s say you have a blog site:

1. **Browser** sends request for `https://yourblog.com/post/123`
2. **Web Server (Nginx)** gets the request:

    - If it's for `/logo.png`, it serves it directly.
    - If it's for `/post/123`, it forwards the request to...
3. **Application Server (Spring Boot)**:
    
    - Fetches post 123 from the database
    - Generates HTML
    - Sends response back to Web Server
4. **Web Server** sends it to the browser.

Now, if you're using a **CDN** like Cloudflare:

- CDN checks if `/post/123` is cached:
    
    - If yes, it serves directly.
    - If not, it fetches from your **origin server** (which is your Web+App server).


Mail server : 

FTP Server :

DNS Server :