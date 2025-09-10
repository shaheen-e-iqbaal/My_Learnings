
-> Linux command to get ARP table : arp -a

-> After it gets IP packet from Network layer and gets to know about the next hop IP address, it will use ARP (Address Resolution Protocol) to figure out the MAC address of next hop.

-> It will create Header known as Layer 2 header, and will join it with received IP packet. now this whole thing will called Frame. so Frame = Layer 2 header + IP packet.

-> The length of the Layer 2 header is 14.

Fields present in Layer 2 Header :

-> Source MAC address, Destination MAC address

-> EtherType field : it indicates which protocol is used in payload encapsulated after layer header. like IPv4, IPv6, ARP ... it gets helpful for the receiver that how to evaluate this frame further.

There is one field with name FCS (Frame Check Sequence) which you will find also present in Layer 2 header. actually this FCS is not part of Layer 2 header. It is attached by the NIC (network interface cart) at the end of whole frame i.e. layer 2 header + IP packet (IP header + TCP segment). 

-> The purpose of this FCS is to check whether a  frame got corrupted or not? so whenever receiver NIC receives frame, it will use this FCS to find out. if it is corrupted, it will immediately drop that frame. 

-> The sender calculates this FCS value using CRC (cyclic redundancy check).

## When and How Is FCS Used?

### ➤ At the Sender:

- **NIC (Network Interface Card)** calculates the CRC-32 checksum.
- Appends the 4-byte FCS to the frame before putting it on the wire.
### ➤ At the Receiver:

- NIC receives the frame and recalculates CRC on the received data (excluding FCS).
- Compares its own CRC with the received FCS:
    - **If matched** → Accept the frame.
    - **If mismatched** → Drop the frame (corrupted in transit).