

***PHYSICAL LAYER:  PDU = BITS***


Devices : Ethernet Cables, Wi-Fi, Hub, NIC(network interface card which is available in all hosts)

Refer [this](https://shardeum.org/blog/physical-layer-in-osi-model/) article for understanding the Job of physical layer.

Refer [This](https://www.computernetworkingnotes.com/networking-tutorials/network-cable-types-and-specifications.html) and  [This](https://www.geeksforgeeks.org/types-of-ethernet-cable/) article to understand types of Cables i.e. Coaxial, Twisted pair, Fiber optics.

Refer [This](https://www.geeksforgeeks.org/types-of-network-topology/) article to understand types of Topologies in network i.e. star, mesh, ring etc.



<--------------------------------------|||||||||----------------------------------------------->



***DATA LINK LAYER : (Hop to Hop connection) PDU = FRAME***


Devices : Switch, Wireless Access points, Modem, NIC(network interface card which is available in all hosts)

Protocols : ARP (Address Resolution Protocol)

Every Node and Hop in network maintains ARP table of its own. Refer [This 5 minutes Video](https://youtu.be/QPi5Nvxaosw?si=DMWSPzcQ68yUAyFr) to Understand ARP.

Main job of This layer is to add a L2 header. which contains source Hop and next Hop (notice not the Mac address of Final destination Hop) MAC address.

If the communication is within local network, then Destination MAC address will be MAC address of Destination host.

else if the communication is outside of local network, then Destination will be MAC address of Next router. after reaching at local router, it will update Source MAC with it's own MAC and destination MAC with Next router / Hop MAC. This process will keep repeating until it reaches to final destination. in return journey same will also be done.

Refer [This Video](https://youtu.be/rYodcvhh7b8?si=ecAUgHw-rGxgdijf) to understand MAC address resolution in both inter network and Intra network communication.




<--------------------------------------|||||||||----------------------------------------------->




***NETWORK LAYER (END TO END CONNECTION) PDU = PACKET***

Devices : Router

Protocols : IP, NAT (network address Translation), ARP (Address Resolution Protocol)

In local network, all host gets a private IP addresses. so when any host try to communicate outside of the network, then upon receiving data to the router, it replaces  private IP address of Host with public IP address of it's own i.e. router which is assigned to it by ISP. so conversion of Private to public is done using NAT protocol. 

so now data will traverse with source IP address as public IP address of router and destination IP address as IP address of Server.

here comes one interesting question. which is :
when data comes back from server, it will have source IP address as server IP address and destination IP address as router IP address. so how will router decide, where to forward this data further?

Answer:

When a response comes back to the router, the router uses **Network Address Translation (NAT)** to determine which local device should receive the packet. Here’s how it works:

### Steps of NAT and Port Address Translation (PAT):

1. **Outgoing Request (From Device to External Server):**
    
    - When your device (e.g., `192.168.0.110:51740`) sends a request to an external server (e.g., Google at `172.217.164.110:443` for HTTPS), the router:
        - Rewrites the **source IP address** in the packet from `192.168.0.110` (your private IP) to the router's **public IP address** (e.g., `203.0.113.5`).
        - Rewrites the **source port** from `51740` (the port your device used) to a new port number generated by the router, e.g., `32123`.
        - This process is called **Port Address Translation (PAT)**, a form of NAT.
    
    The modified packet looks something like this:
    
     `Source IP: 203.0.113.5 
     `Source Port: 32123 
     `Destination IP: 172.217.164.110 
     `Destination Port: 443`
    
2. **NAT Table on the Router:**
    
    - The router keeps a record of this translation in its **NAT table**, mapping:
        
        `203.0.113.5:32123 → 192.168.0.110:51740`
        
    - This way, the router knows which internal (private) device initiated the connection and which port to use for return traffic.


3. **Incoming Response (From External Server to Router):**
    
    - When the external server (e.g., Google) sends a response, the packet is addressed to:
    
     `Destination IP: 203.0.113.5 (the router's public IP) 
     `Destination Port: 32123 (the port assigned by the router)
    
    - The router checks its **NAT table** for a match on `203.0.113.5:32123` and finds the corresponding internal device:
        
        `192.168.0.110:51740`
        
4. **Packet Rewriting (Router to Internal Device):**
    
    - The router rewrites the **destination IP address** and **destination port** in the packet back to the internal device's private IP and port:
        
        `Destination IP: 192.168.0.110 
        `Destination Port: 51740`

	Using this Information, it will generate L4 and L3 headers.

5.  **Resolving MAC address of Destination Device
    
    - After obtaining the IP address and destination port of the local device from the NAT table, the router uses the ARP table to retrieve the corresponding MAC address for that IP address and Use it to generate L2 header.

6. **Delivering the Packet to the Device:**
    
    - The packet is now sent to your device (`192.168.0.110`) on the local network with the appropriate port, and your device can process the response.


Private IP address Ranges : 

**10.x.x.x**
**172.16.0.0 to 172.31.255.255**
**192.168.x.x**











<--------------------------------------|||||||||----------------------------------------------->




***TRANSPORT LAYER (SERVICE TO SERVICE CONNECTION) PDU = SEGMENT


Protocol : TCP / UDP

HTTP/s, SMTP, FTP uses TCP while DNS, DHCP uses UDP.

For Everything about TCP and UDP : [Video](https://youtu.be/jE_FcgpQ7Co?si=P8U5ZHEjqnYCjWlh )

For TCP Handshaking : [Video by Hussein Nasser](https://youtu.be/bW_BILl7n0Y?feature=shared)

Each time Ack message transmitted in TCP for every packet? : [Stack overflow](https://stackoverflow.com/questions/3604485/does-tcp-send-a-syn-ack-on-every-packet-or-only-on-the-first-connection)

TCP also manages the congestion control in network. it uses three policies for that :

Slow start phase: first it will sent one packet, if it receives ack, then it will double it. i.e. send 2 packet. again if it gets ack for all 2 packet, then it will now send 4 packets. this process will continue until threshold is reached.

Congestion avoidance phase: after reaching the threshold, now it will increase the window size by one. ex. if window size is 15, then it will be increased to 16. if it gets ack for all 16, then it will get increased to 17 and so on.

Congestion detection phase: If congestion occurs, the congestion window size is decreased. The only way a sender can guess that congestion has happened is the need to retransmit a segment.


<--------------------------------------|||||||||----------------------------------------------->




***APPLICATION, PRESENTATION AND SESSION LAYER (PDU = DATA)

Protocols of Application Layer : HTTP, HTTPS, DHCP, DNS, FTP, SMTP, POP


DNS Steps:  Browser cache -> OS cache -> Recursive Name server(present at ISP) -> root level server -> Top level domain (TLD) server -> Authoritative name server.

->WWW is a collection of web pages which are linked together. it was developed by Tim Berner Lee.

-> Major work of Presentation layer includes encryption / decryption, data compression.

-> Major  work of session layer is to start, maintain and terminate a session