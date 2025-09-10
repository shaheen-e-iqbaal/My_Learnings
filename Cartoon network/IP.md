
->Linux command to see routing table is : ip route show


1. **Transport Layer (TCP)**
    - The transport layer **creates a TCP segment** by combining:
        - A **TCP header** (with fields like source port, destination port, sequence number, etc.)
        - And the **TCP payload** (i.e., actual application data from the upper layer)
    - This full structure is called a **TCP segment**.
2. **Network Layer (IP)**
    - The network layer takes this TCP segment and encapsulates it with an **IP header**.
        - IP header is usually **20 bytes** for IPv4 without any options.
    - Now the entire structure becomes an **IP packet**:
        `IP Packet = IP Header + IP Payload = IP Header + TCP Segment           = IP Header + (TCP Header + Application Data)`
3. **Routing Decision**
    - The network layer **examines the destination IP address** in the IP header.
    - Using the **routing table**, it decides:
        - Whether the destination is on the **same local network**
        - Or if the packet should be sent to a **router (default gateway)** (next hop)
4. **Data-Link Layer (Layer 2)**
    - The network layer hands the IP packet to the data-link layer.
    - The data-link layer needs the **MAC address of the next hop** (either the destination machine or the router).
    - It uses **ARP (Address Resolution Protocol)** to resolve the IP to MAC address if not already cached.
    - Finally, it constructs the **Ethernet frame**:
        `Ethernet Frame = Ethernet Header + IP Packet`

Fields Present in IP Header :

-> Version : how does the IP version is finalized? 
1. **DNS Resolution (if domain is used)**
    
    - When a client wants to connect to a server using a hostname (e.g., `example.com`), it sends a **DNS query**.
        
    - The DNS server may return:
        
        - **IPv4 address (A record)**
            
        - **IPv6 address (AAAA record)**
            
        - Or **both**
            
2. **Client Chooses Which IP to Use**
    
    - Based on the response, the client chooses which IP version to use.
        
    - Many modern systems use a strategy called **Happy Eyeballs**:
        
        - Try **IPv6** first, but fall back to **IPv4** if IPv6 is slower or unavailable.
            
    - Once chosen, the client constructs the IP header accordingly (IPv4 or IPv6).
        
3. **Server Responds Accordingly**
    
    - The server doesn’t choose the IP version—it **responds** based on how it was contacted.
        
    - If it receives a request over **IPv4**, it replies using IPv4.
        
    - If it receives a request over **IPv6**, it replies using IPv6.


-> Source IP address, Destination IP address. 

-> Header Length : Specifies the length of the IP header.

-> Total Length :Specifies the total length of the **entire IP packet**: Total Length = IP Header + IP Payload (i.e., TCP Segment)

-> Time To Live (TTL) : 
### What is TTL?

- It determines **how many hops (routers)** a packet can pass through before being discarded.
- It's used to **prevent infinite routing loops**.
### 🔹 How TTL works:

- The **sender sets a default TTL value**, commonly:
    
    - 64 (Linux/macOS),
    - 128 (Windows),
    - 255 (some network devices).
- **Each router** that forwards the packet **decrements the TTL by 1**.
- If TTL reaches 0, the packet is **dropped**


->Protocol : It specifies which protocol is used by the encapsulated payload.
It uses 6 to represent TCP, 17 for UDP.

-> Header Checksum : It is calculated using IP header fields only. no TCP segment (IP payload) is included while calculating it.

-Whenever any device receives packet, it uses this checksum to verify weather it is corrupt or not? if it is not corrupt, then it will proceed next, i.e. will figure out next hop.
-so one thing to notice is that whenever any router receives packet, it will reduce it's TTL by 1. so value of TTL is changed. so the router will calculate new header checksum and update it with current one.


-> so one thing to learn is, whenever router receives any frame, it will first remove layer 2 header from it, then it will make changes in layer 3 header like update TTL, checksum. now it will figure out next hop. after it, it will regenerate layer 2 header. so router makes changes in layer 2 and layer 3 headers.


==**Where real Routing Happens?**==

#### **On our computer (host):**

- No real routing algorithms are applied.
- The host simply checks:
    1. **Is the destination IP in the same subnet?**
        - If yes → send directly using ARP to resolve MAC.
    2. **If not** → send the packet to the **default gateway** (usually the home router).
> ✅ This is **basic route selection** via a static routing table, not dynamic routing.

#### **→ On the home router (SOHO router):**

- Typically, no dynamic routing happens here either.
- Most home routers just **forward all non-local traffic to the ISP router** (via the WAN interface).
- They may have a few static routes but do **not** run OSPF, BGP, etc., unless explicitly configured.
> ✅ The router acts as a simple gateway.
#### **→ On ISP and backbone routers:**

- This is where **real dynamic routing occurs**.
- Routers maintain **very large routing tables**, with entries for many IP prefixes.
- They use routing protocols like:
    - **OSPF / IS-IS** (within an ISP’s network – IGP)
    - **BGP** (between ISPs – EGP)
- These protocols:
    - Share routing information with neighboring routers.
    - Use algorithms like **Dijkstra** (OSPF), **Bellman-Ford** (RIP), or **Path Vector** (BGP) to compute best paths.
    - Constantly update the **routing table** as the network topology changes.
#### **→ When a packet arrives at such a router:**

1. The router checks the **destination IP**.
2. Finds the **longest prefix match** in its routing table (most specific subnet match).
3. Chooses the **best next-hop** based on cost/metric (e.g., shortest path, least hops, policy).
4. Forwards the packet accordingly.

> ✅ **Forwarding uses the routing table**, which is kept updated by routing algorithms, but the algorithm isn’t run for each packet.