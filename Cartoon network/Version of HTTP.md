
-> HTTP is a client-server architecture based protocol.

-> HTTP Request/Response format :

```
	Method /path HTTP Version
	Header1 : value
	Header2 : value
	Header3 : value
	
	body will start from here.
	
	
	ex :
	
	HTTP/1.1 200 OK
	Date: Wed, 10 Dec 2025 02:02:00 GMT
	Server: Apache/2.4.41 (Ubuntu)
	Last-Modified: Tue, 09 Dec 2025 18:00:00 GMT
	ETag: "12345-6789abc"
	Content-Type: text/html; charset=UTF-8
	Content-Length: 12345
	Connection: keep-alive
	
	<!DOCTYPE html>
	<html>
	<head>
	    <title>Example Page</title>
	</head>
	<body>
	    <h1>Welcome to Example.com</h1>
	    <p>This is a sample webpage.</p>
	</body>
	</html>
```
-> In above HTTP payload, notice there is line break between Headers and Body. this empty line tells HTTP parser that Headers are completed and body will start now. 
-> How will it know when will body of current request ends and new packets it is receiving from TCP is of some other request? for this, the sender sends one more Header i.e. `Content-length: bytes`. this header indicates the bytes present in HTTP body (notice not entire HTTP payload i.e. (Header + Body) but only Body).

==HTTP 1.0 :==

-> this is one of first version of HTTP. in this version, for each request, client-server has to start new TCP connection. we know that, TCP is three way handshaking. which is time consuming if we have to do it each time we want to send request. ex : if i want some image from server, i first have to establish TCP connection. this connection will be closed once i receive that image. now if i want some html file, i again have to establish new TCP connection.... and it goes on.

-> so, this was the main bottleneck of HTTP 1.0.


HTTP 1.1 :

-> HTTP 1.1 solves the limitation of of 1.0 by keeping the TCP connection open by mentioning the "Connection : Keep-Alive" header. so, now, client can send multiple requests over this same TCP connection.
-> so, in plain HTTP 1.1, we send first request on TCP connection, we wait until we fully receives it's response. then we send other request on same TCP connection..... and it goes on.
-> so, to increase parallelism of sending requests, we have to open multiple TCP connections at same time. chrome browser allows only 6 per host simultaneous TCP connection. i.e. while using plain HTTP 1.1, we can send 6 parallel HTTP request in chrome.

-> HTTP 1.1 also offers one more feature called pipe-lining. in which, we can send new requests on same TCP connection even though the earlier ones response hasn't been received yet. but the main catch is : we will receive the response in the same order of requests. i.e. we first receive response of our first request, then response of second one... and goes one. so, here you can notice that, even though the response of our fourth request is completely ready at browser side, we have to still wait for the response of first, second and third request. suppose if second request's response is large or server took long time to prepare it's response, then we have to wait that much long to get the response of fourth request. this is called "Head Of Line Blocking"  at application layer in HTTP 1.1. notice that this is due to application layer. application itself can't send the forth request until it completes first, second and third. this is how HTTP 1.1 pipe-lining was designed. due to this problem, most of the browser has disabled pipe-lining in HTTP 1.1.

GPT Version : 
```
 HTTP/1.1 solves the limitation of HTTP/1.0 by allowing the TCP connection to stay open using the **Connection: keep-alive** header, enabling the client to send multiple requests over the same connection. In plain HTTP/1.1, the client typically sends one request on a TCP connection, waits until the full response is received, and then sends the next request, meaning parallelism requires opening multiple TCP connections. Browsers like Chrome generally allow only **six simultaneous TCP connections per host**, so with standard HTTP/1.1 (without pipelining), a client can issue at most six parallel requests to the same server. HTTP/1.1 also introduced **pipelining**, where the client can send new requests on the same TCP connection even before previous responses arrive. However, responses must be returned **strictly in request order**, which means a slow or large response for one request blocks all following responses—even if those later responses are already prepared—causing **Head-of-Line (HOL) blocking**. Due to this severe limitation, most browsers disabled HTTP/1.1 pipelining by default.
```

-> HTTP 1.1 doesn't do any Header compression. it will pass all the headers as it is.


==HTTP 1.2 or HTTP 2 :==

***Multiplexing :***

-> This version removes the application level HOL by offering the multiplexing. multiplexing is one of main thing offered by HTTP 2. 
-> so HTTP 2.0 we can send multiple requests/response over single TCP connection. application (Browser/Server) converts each request/response payload into independent streams. this streams can be send over TCP connection independent of each other. we can send something like two packets of stream 1, then 3 packets of stream 2, then 1 packet of stream 1....
so, to start sending response of new request, application (browser/server) doesn't need to make sure that previous responses are completely transferred or not which was required in case of HTTP 1.1.

-> By Multiplexing, HTTP 2 increases request parallelism to greater extent, requires only one TCP socket which reduces the requirement of kernel space memory.

-> but still we have TCP level HOL (Head of Line) blocking. how? suppose OS doesn't receive packet 3 of stream 2 but has received packet 4,5. it has also received all the packets of stream 1, stream 3. but still OS won't pass this other stream's packet to application (browser/server).  it will wait until it receives packet 3 of stream 2 i.e. missing packet. 

why does OS doesn't pass other stream packets even though they are fully received?
-> because TCP is byte oriented not the packet oriented. OS's (TCP/IP stack) doesn't understand the language of stream. it only understand the bites language.  it will just simply check if it has received continues bytes or not? if yes, it will pass it to application otherwise will ask for missing packets. 

GPT Version :
```
	HTTP/2 removes the application-level Head-of-Line (HOL) blocking seen in HTTP/1.1 by introducing **multiplexing**. Multiplexing is the main improvement of HTTP/2. It allows the client and server to send multiple requests and responses simultaneously over a **single TCP connection**. To achieve this, HTTP/2 breaks each request/response body into small independent units called **streams**. These streams are split further into frames, and the application is free to interleave them in any order. For example, the browser may send: 2 frames of Stream 1, then 3 frames of Stream 2, then 1 more frame of Stream 1, and so on. Because of this mechanism, the application does not need to wait for one response to finish before sending another—something that was mandatory in HTTP/1.1 due to response ordering rules.

	However, even though HTTP/2 solves the application-level blocking, it **still suffers from TCP-level HOL blocking**. This happens because HTTP/2 runs **on top of TCP**, and TCP has strict in-order delivery semantics. For example, imagine Stream 2 has frames numbered 1, 2, 3, 4, 5. Suppose the operating system receives frames 4 and 5 of Stream 2, and it also receives all frames of Stream 1 and Stream 3, but **frame 3 of Stream 2 is missing**. In this situation, even though many frames belonging to other streams are ready, the OS will not deliver _any_ data beyond the missing TCP sequence number to the application. It must wait for the retransmission of the missing frame because TCP works with a continuous **byte sequence**, not stream IDs.

	The key reason for this behavior is that **TCP is byte-oriented, not packet-oriented**. TCP does not understand the concept of HTTP/2 “streams” at all. All the OS sees is a continuous flow of bytes. It simply checks: “Do I have all bytes in order?” If yes, it delivers them to the application. If even one byte is missing—no matter which HTTP/2 stream it belongs to—TCP must stop and wait for retransmission. This is why a lost packet in one HTTP/2 stream can block the progress of every other stream, even if those streams' data has fully arrived.
```


***Header Compression:***

-> HTTP 2 enable Header compression which was not the case with HTTP 1.1.  when we send any HTTP request/response, it always contains some headers like `Method: GET/POST...`, `Scheme: HTTP 1.x`, `Content-type: JSON/TEXT...`, `Content-length: 133...`, `Authorization: JWT_TOKEN`, `Cookies: dksdk` ... 
so, most of the headers are same during each HTTP request. for some, their values might get changed but the name of header is guaranteed to be same across all future request/response. so after noticing this, HTTP 2 uses one algorithm called `HPACK`. which works as below. 

HPACK has two types of dictionary. 1. Static 2. Dynamic.

Static : this dictionary contains 61 famous HTTP Headers in format : `Index(int) Header_Name: Header_Value` 

 ex : `1 Method: GET`, `2 Method: POST`,....  `3 Scheme: HTTP 2`, `6 Content-type: ""`, `7 Content-type: JSON`, `8 Content-type: Text`.... now when it sees Headers in HTTP request, it will first check in this static dictionary. if it finds any matching pair, it will replace that Header with it's corresponding index value. ex : If header `Method: GET` is present in request/response, it will replace that string with index 1. so, you can see that, the size required for this header is reduced from some xxx.. bytes to 1 byte. now suppose entire header string (Name: Value) doesn't match, but only name matches, then also it will use index like `index: value(encoded using Haufman)`. 

Dynamic : This dictionary it uses to store new Headers which are not present in static one. suppose, request/response contains header `X-custom-header: value`, then it will store this header in dynamic dictionary and use it's index.

-> For all other headers which are not present in Static or dynamic dictionary, it will use Huffman encoding algorithm and will convert them from string to bytes and will use those bytes.

-> This header compression reduces HTTP payload size significantly. which in turn helps reducing latency as we have less data(bytes) to transfer over network between client/server.

GPT Version:
```
	HTTP/2 introduces **header compression** to reduce the overhead caused by repeatedly sending large and mostly identical HTTP headers with every request and response. In HTTP communication, headers such as `:method`, `:scheme`, `content-type`, `content-length`, `authorization`, and `cookie` are sent with almost every request. While header **values** may change (for example, `content-length` or tokens), the **header names** are usually the same across multiple requests and responses. Sending these headers as plain text every time wastes bandwidth and increases latency, especially on slow networks. To solve this problem, HTTP/2 uses a compression mechanism called **HPACK**.

	HPACK works using **indexed header fields** and maintains two types of dictionaries (also called header tables): **Static Table** and **Dynamic Table**. The **Static Table** is predefined by the HTTP/2 specification and contains commonly used HTTP header fields and common values. Each entry has an **index number**. For example, entries include `:method: GET`, `:method: POST`, `:scheme: https`, `content-type: application/json`, and others (about 61 entries in total). When an HTTP/2 endpoint sees a header that exactly matches an entry in the static table, it replaces the entire `name: value` string with just the **index number**. This dramatically reduces the size—for example, instead of sending `:method: GET` as text, it sends a small integer index.

	If a header name matches an entry in the static table but the value is different, HPACK still saves space by sending the **index of the header name** along with the **encoded value**. The value is encoded using **Huffman encoding**, which further reduces size by representing common characters with fewer bits. For example, if the header is `content-type: application/xml` and only `content-type` exists in the static table, HPACK sends the index for `content-type` and the compressed value `application/xml`.

	The **Dynamic Table** is used to store header fields that are not present in the static table but are seen frequently during the connection lifetime. For example, a header like `x-custom-header: abc123` or even frequently changing headers such as `authorization` may be added to the dynamic table. Once stored, future requests can refer to that header using its **dynamic table index**, instead of sending the full text again. The dynamic table is **connection-specific** and changes over time as new headers are added and old ones are evicted based on size limits.

	If a header is **not present in either the static or dynamic table**, HPACK sends it as a literal header field. Even in this case, the header name and value are usually compressed using **Huffman encoding**, so they are still much smaller than plain text headers. This ensures that even uncommon or one-time headers benefit from compression.

	Overall, HPACK significantly reduces header size, avoids repeatedly sending the same metadata, and improves performance—especially on high-latency or low-bandwidth networks—while also avoiding the security issues that existed in earlier HTTP compression mechanisms.
```

-> Other than the headers, HTTP body (mainly HTML, CSS, JS files which usually has lot of repeatable words) is also compressed using algorithms like gzip, deflet... the client sends header `Accept-encoding: algo1, algo2...` which tells server the list of HTTP body compression algorithms it supports. in return, server picks up any one from that list and compresses the HTTP body. it will tell the client which compression algorithm it has used by header `Content-encoding: algo`. 


***[Stream Prioritization:](https://www.cloudflare.com/learning/performance/http2-vs-http1.1/#:~:text=What%20is%20prioritization%3F)***

-> in HTTP 2, server can mention the priority of streams which tells the client (browser) to which stream should used first to render. like in case of blog/article request, which asks for  text, images, videos, ad images... to load requested blog. now based on the requested content, client (browser) will set priority of each of them. now upon receiving it, server will send those streams in that priority.
-> This thing is totally taken care by Browser/Server. we don't have to write any code for it.


==HTTP 3==

-> It's build on top of QUIC and uses UDP as transport layer protocol instead of TCP.

-> QUIC handles all the stuff which TCP was doing earlier i.e. segmentation, congestion control, packets re-transmission in case of packet loss, asking for lost packet....

-> By using UDP, it removes HTTP 2's TCP level HOL blocking problem.

-> It also reduces Connection + TLS roundtrip to 1 from 2-3 RTT required by HTTP 2 for the same functionality. which makes it faster than HTTP 2.

-> As we have shifted the responsibility of reliable communication from OS level to application level (i.e. using UDP + QUIC instead of TCP for reliability), we are now much independent to make any future changes.

-> why we need QUIC even though it implements same things which TCP already offers for reliability ? ans : [[TCP IP at OS level#^5b9d1d]]

-> HTTP 3 supports all the features  of HTTP 2 like multiplexing, Header compression, stream prioritization... etc

***Connection Migration :***

-> One important feature introduced by **HTTP/3 (via QUIC)** is **connection migration**, which allows a client to continue using the same connection even when the network changes (for example, switching from Wi-Fi to mobile data). In HTTP/2 and older versions, a network change breaks the connection and forces the client to start over by creating a new TCP connection, performing a new TLS handshake, and then resending requests. This makes network switches slow and disruptive.

The reason this problem exists in HTTP/1.1 and HTTP/2 is that they are built on **TCP**, where a connection is uniquely identified by a **4-tuple**:  
`Source IP, Source Port, Destination IP, Destination Port`.  
When a client switches networks, its **source IP and source port change**, so the 4-tuple no longer matches. As a result, the existing TCP connection becomes invalid, and the client must establish a completely new connection, including TCP and TLS handshakes.

HTTP/3 avoids this problem because it runs over **QUIC**, which is built on **UDP** and introduces a new abstraction: the **Connection ID (CID)**. Although UDP packets themselves are still sent using IP addresses and ports, **QUIC does not rely on the IP/port tuple to identify a connection**. Instead, each QUIC connection is identified by a **Connection ID**, which is explicitly included in every QUIC packet.

When a client first establishes an HTTP/3 connection, the server assigns (or negotiates) a Connection ID. This Connection ID remains stable for the lifetime of the connection. If the client later switches from Wi-Fi to mobile data, its **source IP and source port change**, but the client continues sending packets with the **same Connection ID**. When the server receives these packets, it recognizes the Connection ID and understands that the packets belong to the **same logical connection**, even though the network path has changed. The server then seamlessly continues processing requests and responses without requiring a new handshake.

-> So, The key difference is that **QUIC moves connection identification from the transport layer into the Application layer (HTTP protocol itself)** using Connection IDs.