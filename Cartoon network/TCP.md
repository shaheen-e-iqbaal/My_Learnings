
-> **TCP is byte stream–oriented**, not packet-oriented.  
This means TCP tracks data in terms of **bytes**, not discrete packets.


-> Suppose the application layer passes **5000 bytes** of data to the TCP layer.
- TCP will **split** this into multiple segments before sending.
- The **maximum size** of each segment's data portion is determined by the **MSS (Maximum Segment Size)**, which is **negotiated during the TCP 3-way handshake** between client and server.


-> - The **SEQ number** in a TCP segment **indicates the byte position** of the **first byte of application data** being sent in that segment.
- Though the **initial SEQ number** is a **random 32-bit number** chosen by the sender, **Wireshark often shows it as 1 for readability** (relative sequence numbers).
#### Example:

- Sender starts with SEQ = 1 (for simplicity).
- First segment carries **664 bytes** of application data.
- Then the **next segment’s SEQ = 665**, because it starts with the 665th byte.

-> suppose in first segment it sends 664 bytes of application data, then the SEQ of second segment will be 665. so SEQ represents the number of first byte of application data being sent by TCP.

-> The receiver the first byte stream with SEQ = 1. from the TCP header of sender, receiver will find out the total application data bytes (TCP payload) sent by sender. in our case, sender has sent 664 bytes, so receiver will send the response with ACK = 665. so ACK represents the next expected SEQ from sender.

-> ACK tells that i have received payload up to bytes = ACK - 1. now i am expecting the payload from byte number = ACK.

-> **TCP payload** refers to the actual **application data** inside the TCP segment.
- It **does not include** the TCP, IP, or Ethernet headers—only the raw data from the application layer (e.g., HTTP, TLS, etc.).

--------------------------------<<<<<<<<<<<>>>>>>>>>>-------------------------

**What will happen if some in *between* segment is not received by receiver? :**

- Suppose Total data to send: **1000 bytes**
- So Sender segments data as Shown below.

|Packet|SEQ|Payload Size|Byte Range|
|---|---|---|---|
|1|1|654|1 – 654|
|2|655|154|655 – 808|
|3|809|77|809 – 885|
|4|886|115|886 – 1000|

Now, sender sends This segments one by one. but The packet with `SEQ = 809` (77 bytes) is **Not received by receiver**.

### **Receiver Behavior**

1. **Receives Packet 1 (SEQ = 1, 654 bytes)**  
    → All in order  
    → Sends `ACK = 655` (next expected byte)
    
2. **Receives Packet 2 (SEQ = 655, 154 bytes)**  
    → All in order  
    → Last byte received: 808  
    → Sends `ACK = 809`
    
3. **Receives Packet 4 (SEQ = 886, 115 bytes)**  
    → This is **ahead of expected SEQ = 809**  
    → TCP discards out-of-order data (unless it supports selective ACKs)  
    → Still sends **duplicate `ACK = 809`** to indicate:  
    _“I’m still waiting for byte 809.”_
    
4. **Receiver keeps sending duplicate ACK = 809**  
    → On receiving 3 duplicate ACKs, sender performs **Fast Retransmit** of the lost segment.

### **Sender Behavior**

- After getting 3 duplicate ACKs (all with ACK = 809), the sender assumes packet with SEQ = 809 is lost.
    
- It **retransmits the missing segment** with SEQ = 809 and payload = 77 bytes.

### Once the missing packet is received

- Receiver now has complete, in-order data up to byte 1000.
    
- Sends **ACK = 1001** (meaning all 1000 bytes received successfully).

***Final Summary***  :

|Stage|SEQ Received|ACK Sent|
|---|---|---|
|Packet 1 OK|1 – 654|655|
|Packet 2 OK|655 – 808|809|
|Packet 3 **lost**|–|–|
|Packet 4 arrives early|886 – 1000|**809 (duplicate)**|
|Fast Retransmit triggered|Retransmit SEQ 809|–|
|Packet 3 received now|809 – 885|1001|

-> suppose there is 1000 bytes of total payload sender has to send. so it sends these segments with SEQ = 1 (payload = 654 bytes), SEQ = 655 (payload = 154 bytes), SEQ = 809 (Payload = 77 bytes), SEQ = 886 (payload = 115).

Receiver receives the packets with SEQ = 1, so it sends the ACK = 655, now it receives packet with SEQ = 655, so it sends ACK = 809. now it receives packet with SEQ = 886 which is not equal to the SEQ expected by receiver which is = 809. now it sends multiple ACKs with ACK = 809, which tells sender that the packet with SEQ = 809 is dropped and not received by receiver. so it will send that packet with SEQ = 809 again. now receiver will ACK for packet with ACK = 1001.

-> There is one more case when retransmission of packet is done by sender. sender waits for some time to get ACK message from receiver for packets already sent. if it didn't get by that time, it will retransmit those packets.


--------------------------------<<<<<<<<<<<>>>>>>>>>>-------------------------


-> The TCP header is of 20 bytes mostly.

-> The **checksum** present in the **TCP header** is used to **detect errors** in the **entire TCP segment**, which includes:

- the **TCP header**
- the **TCP payload (data)**
- and a **pseudo header** (constructed from IP header fields: source IP, destination IP, protocol, and TCP length)

-> If the **receiver detects an error** using the checksum, it **silently discards the corrupted segment** (does **not** send any ACK). The **sender will eventually retransmit** the segment when it **does not receive an ACK** within the expected timeout.

#### List of TCP Flags and Their Meaning :

| Flag    | Name           | Purpose                                     | When It's Used                                                   |
| ------- | -------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| **SYN** | Synchronize    | Start a connection                          | During the **3-way handshake** (1st and 2nd packets)             |
| **ACK** | Acknowledgment | Acknowledge received data                   | Used in **almost every packet** after the initial SYN            |
| **FIN** | Finish         | Gracefully close a connection               | Used during **connection termination**                           |
| **RST** | Reset          | Immediately abort the connection            | Used when there's an error or unwanted connection                |
| **PSH** | Push           | Deliver data to the application immediately | Used when data should be processed **immediately**, not buffered |

-> During the TCP handshake, client and server both agree on the ISNs (initial sequence numbers), MSS (maximum segment size).


-> [TCP header image.](https://share.google/images/vNIU9mJAIdIkZn7UM)
-> [UDP header image](https://share.google/images/SEd9Sjmp9QuoSkZ6h)

