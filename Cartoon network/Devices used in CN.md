
1. Hub
2. Switch
3. Router
4. Modem
5. Access Point


Hub :

currently rarely used. It is a Physical layer device. we can say it is older version of modern Switch.

- A hub **broadcasts** all incoming data to **every device / Host**  connected to its ports, regardless of the destination.
- Every device connected to the hub receives the data, but only the intended recipient device processes it; the rest ignore it.


Switch :

It is used to connect multiple Host. it works in local network. it is a Data Link layer device. Switch can further connected with either router or different switch.

- A switch uses a **MAC address table** to identify which devices are connected to which ports.
- When data arrives at the switch, it examines the destination MAC address and sends the data **only** to the port where the destination device is connected, rather than broadcasting to all devices.
- This improves network efficiency because only the intended recipient receives the data, reducing collisions and increasing overall performance.

Look at this example [Switch photo](https://study-ccna.com/wp-content/uploads/2016/01/cisco_switch.jpg)


Router: 

It is a Network layer device. Upon receiving any data, it decides the next location/Hope where this data should be sent based on the destination IP address. It uses it's own Routing table to decide the next hope.

Routing table contains information related to IP addresses and the cost to reach that IP addresses. so for any data, router first match destination IP address of the data with IP addresses present in routing table. if it finds any IP address who's prefix match with the destination IP address, then it will send to that IP address otherwise it will use default gateway which is present in all routing table.

- **Network traffic management**:
    - The router directs incoming and outgoing data to the correct devices within your local network (such as computers, smartphones, smart TVs, etc.).
- **Assigns private IP addresses**:
    - The router assigns private **IP addresses** to each device in your local network, usually using **DHCP** (Dynamic Host Configuration Protocol).
- **NAT (Network Address Translation)**:
    - Routers use **NAT** to translate private IP addresses in your local network into the public IP address assigned by your modem, enabling communication with the internet.
- **Firewall/Security**:
    - Many routers have built-in **firewalls** to protect your local network from external attacks.



Modem: 

It works as a bridge between local network and ISP. it is connected with Local network router's WAN port and ISP. 
The modem is assigned a public **IP address** by the ISP. This IP address allows your home network to communicate with the wider internet.
==In current time, Modem is integrated within router instead of separate device. so router is combo of modem + router. so the IP address of router will be IP address of modem assigned by ISP.==


Access Point:

so when the signal of router is not able to reach certain area of home / office, then we can use access point to solve it.
we can connect access point to router using wire, and set it to the desired location. this access point can now give sufficient signals to that area.

ex. in hotel golden, router is placed at reception, so wi-fi signals can't reach the last room of third floor. so what we can do is that, we can connect one access point to router using wire and place this access point in the third floor. so now any device on third floor withing the range of access point can get signals.





==Use [This article](https://www.practicalnetworking.net/series/packet-traveling/key-players/) to Understand better about Host, Network, Router, ARP.==