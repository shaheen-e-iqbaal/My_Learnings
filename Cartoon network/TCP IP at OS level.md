
-Whenever we use the browser or Postman to call a URL, **the browser/postman handles only the application-layer logic**. Everything related to transport (TCP/UDP), IP addressing, routing, packet segmentation, acknowledgments, retransmission, etc., is handled by **the operating system** through the TCP/IP stack inside the kernel.

When we enter a URL, the browser starts by trying to resolve the domain name. It checks its **DNS cache** first. If the domain is not found, the browser issues a **system call** to the OS — essentially something like:

> “Please resolve this domain name".  
> From this point on, the OS takes over. The OS checks its own DNS cache. If needed, it prepares a DNS query and decides whether this DNS query should go over **DNS/TCP** or **DNS/UDP**. The OS creates the appropriate UDP or TCP header, the IP header, and the L2 frame. The NIC sends the DNS packet over the network. When the DNS response arrives, the NIC converts the electrical/wireless signals into bits, detects full frames using length/FCS, verifies the MAC address, and passes the frame upward. The OS removes the Ethernet header, removes the IP and UDP headers, extracts the DNS response, and returns the final **IP address** back to the browser.

Once the browser receives the server's IP, it decides the **application-layer protocol** (HTTP, FTP, SMTP, etc.). Based on the protocol, it tells the OS to create the appropriate **transport-layer socket** by making a syscall such as `socket(AF_INET, SOCK_STREAM)` for TCP or `socket(AF_INET, SOCK_DGRAM)` for UDP. A socket is not a physical object but a **kernel structure** that stores all important connection parameters: local IP, local port, remote IP, remote port, TCP sequence and acknowledgment numbers, buffers, connection state, timers, retransmission info, etc. The OS returns a **file descriptor (FD)** to the browser, which is simply a number representing that socket.

If the browser needs a **TCP connection**, it makes another system call: `connect(fd, serverIP:port)`. Now the OS initiates the **TCP three-way handshake**. The browser itself does _nothing_ in this process. The OS prepares the SYN segment, adds TCP and IP headers, wraps it inside an Ethernet frame, and sends it through the NIC. When SYN/ACK returns, the NIC receives it, the OS processes the TCP headers, updates the socket state, sends ACK back to the server, and completes the connection. Only after the OS finishes this, it informs the browser that the TCP connection is ready.

Now the browser prepares the **HTTP request payload** — headers, body, cookies, etc. — and sends it to the OS via a syscall such as `send(fd, data)`. From this point, OS kernel does everything: segmenting the data into TCP segments, setting sequence numbers, adding headers, retransmitting on loss, managing congestion control, ensuring ordering, creating IP headers, filling Ethernet frames, and sending them out via NIC.

While receiving data from the server, the NIC receives frames, verifies MAC address, reassembles bits, checks CRC, strips the frame header, and passes the packet to OS. The OS checks IP headers, checks the destination port, finds the matching socket, processes TCP headers, updates acknowledgments, handles retransmissions, reordering, and pushes the application data into the socket buffer. The browser never knows about any packet, sequence number, TCP flag, or loss. It only knows that “new data is ready” because the OS marks the socket as readable. The browser then performs a syscall like `recv(fd)` to pull the HTTP response.

For HTTP/1.1, the browser decides to keep or close the TCP connection based on rules like `Connection: keep-alive`, idle timeout (TTL), and number of concurrent connections per domain (Chrome allows ~6). The browser does not close the TCP connection by itself — it simply calls `close(fd)` which instructs the OS to send FIN packets and shut down the connection. We cannot keep infinite connections open because each open TCP socket consumes kernel memory (buffers, state, timers) and OS resources.

For HTTP/2, the browser uses the same TCP connection but sends multiple logical streams inside it. The browser is responsible for multiplexing streams, prioritization, and framing, but the OS still handles TCP reliability. The downside is **TCP head-of-line blocking**: if any packet is lost, all streams stall waiting for retransmission of that packet.

For HTTP/3, the browser does not use TCP anymore. Instead, it uses **QUIC**, which runs over UDP. Initially, the browser may start with TCP/TLS to check ALPN support. Once QUIC is confirmed, it creates a UDP socket via syscall and performs the QUIC handshake. QUIC implements reliability, sequencing, retransmission, congestion control, flow control, and multiplexing — all in **user space inside the browser** rather than inside OS kernel. The OS only provides best-effort UDP delivery (send/receive). Because QUIC handles packet loss per-stream instead of per-connection, HTTP/3 avoids head-of-line blocking. QUIC also supports 0-RTT resume and connection migration using Connection IDs, which TCP cannot do.

Postman works exactly like a browser in terms of networking: DNS resolution through OS, socket creation via syscalls, relying on OS for TCP, receiving data via socket buffers, and only handling the application-layer HTTP content.

Across all of this, the NIC is responsible for converting signals to bits, detecting full Ethernet frames, verifying MAC address, stripping off layer 2 details, and passing the packet to OS. The OS kernel handles layers 3 and 4 (IP, TCP, UDP), including routing, fragmentation, retransmission, acknowledgments, congestion control, and socket management. The browser/Postman only handle application-layer logic, while everything lower-level is done by OS and NIC.


# **Why We Needed QUIC Even Though It Rebuilds TCP Features**

^5b9d1d

Even though QUIC re-implements reliability, retransmission, ordering, congestion control, and flow control (all things TCP already does), it fixes fundamental limitations that TCP can _never_ solve because TCP lives inside the **OS kernel** and has become practically **impossible to evolve**. Every improvement in TCP requires OS updates, kernel patches, driver patches, and must remain compatible with millions of routers, middleboxes, firewalls, NATs, and proxies around the world. These network devices have hard-coded expectations of how TCP should behave, so introducing new TCP features often gets blocked or broken. This creates what is called **protocol ossification** — TCP is frozen in time.

QUIC avoids this completely by running **in user space, over UDP**. Because UDP is minimal and doesn’t enforce any behavior beyond best-effort delivery, QUIC is free to implement any transport-layer behavior without being blocked by old network devices. QUIC can evolve quickly: browsers and servers can update QUIC logic weekly, without waiting for OS vendors or worrying about legacy infrastructure. This freedom enables QUIC to provide modern features that TCP simply cannot add without breaking the internet.

Another major advantage is that QUIC merges **transport layer + cryptographic layer** into one. TCP requires TLS on top, which means 2–3 round trips before actual data can flow (TCP handshake + TLS handshake). QUIC integrates the cryptographic handshake directly into the transport handshake, reducing this to **1 RTT**, and even **0 RTT** in some cases. This dramatically improves page load times, especially on high-latency or mobile networks.

---

# **💡 How 0-RTT Resume Works in QUIC (and Why TCP Can’t Do It)**

QUIC supports **0-RTT** by remembering certain cryptographic information from a previous session. When a client connects to a server for the first time, it completes a QUIC handshake and receives a **session ticket** (similar to TLS session resumption). This ticket contains enough encrypted state so that during the next connection, the client can immediately start sending application data without waiting for the server to confirm the handshake.

In simple words:

- First visit:  
    Client and server exchange keys → server gives client a reusable ticket.
    
- Next visit:  
    Client uses the ticket to derive early keys and sends encrypted HTTP data **immediately**, without waiting for the server. This saves one round trip, hence **0-RTT**.
    

TCP cannot do this because:

- TCP handshake and TLS handshake are **separate**.
    
- TCP handshake _must_ finish before TLS starts.
    
- TLS 1.2 required full round trips; TLS 1.3 supports 0-RTT but **TCP still forces the client to wait for the TCP handshake to complete first**.
    

So TCP+TLS can never achieve true “start sending real data immediately” because TCP itself requires at least 1 RTT before application data is allowed.

QUIC bypasses this by combining transport and encryption into one handshake.

---

# **💡 How QUIC Connection Migration Works (and Why TCP Cannot Do It)**

QUIC connections are identified using a **Connection ID (CID)** — a random ID chosen by the server.

This ID is _independent_ of:

- client IP
    
- client port
    
- network interface (WiFi, LTE, etc.)
    

This allows QUIC to continue the same connection even if the user's IP address changes, such as:

- switching from WiFi → mobile data
    
- moving between cell towers
    
- enabling/disabling VPN
    
- using load balancers that change client source IP
    

When the client's IP changes, QUIC simply updates the path metadata and continues sending packets using the same CID. No handshake restart. No broken streams. No lost downloads.

TCP **cannot** do this because a TCP connection is identified by its **4-tuple**:

`(source IP, source port, destination IP, destination port)`

If the client IP changes, this 4-tuple changes → TCP considers it a **different connection**, not an existing one.

Therefore:

- TCP cannot migrate connections across networks.
    
- TCP cannot survive IP changes.
    
- TCP cannot preserve streams when network paths change.
    

QUIC solves all these because its identity is not tied to IP addresses.

---

# **💡 Summary: Why QUIC Was Necessary**

Even though QUIC duplicates many functions of TCP, it was necessary because it provides capabilities that TCP _cannot support_ due to deep architectural limitations.

### QUIC Advantages:

- **User-space implementation** → fast evolution, no OS updates required
    
- **Works over UDP** → avoids middlebox interference
    
- **0-RTT and 1-RTT handshakes** → faster connections
    
- **Built-in encryption** → no separate TLS layer
    
- **Stream-level multiplexing** without TCP head-of-line blocking
    
- **Connection migration** across changing IPs
    
- **Improved congestion control**, deployable without kernel changes
    
- **Better performance on mobile and lossy networks**
    

### TCP Limitations:

- Stuck in kernel → cannot evolve quickly
    
- Separate TLS → slower handshakes
    
- Connection tied to IP/port → cannot migrate
    
- HOL blocking at transport layer
    
- Cannot experiment with new congestion control widely
    
- Middleboxes block new TCP features
    

---