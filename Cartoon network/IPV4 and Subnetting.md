
IPV4 address : 

-> it is a total 32 bit address. represented as a.b.c.d

Class A :

-> first bit is always set to zero.
-> first 8 bit (one octet) represent network address.
-> the range of first octet is 0-127.
-> so, total network address in class A = 2^7 = 128
-> 0 and 127 is not assigned to any network. 0.x.y.z is used as identifier for WWW.
-> so total useful network address in class A = 128 - 2 = 126.
-> for each network, total possible host = 2^24
-> host address x.0.0.0 is used as identifying address of that network. so it is not assigned to any host. similarly, address x.255.255.255.255 is used as broadcast address. so it is also not assigned to any host.
-> so total number of useful host addresses in each network of class A = 2^24  - 2.
-> Default mask of class A is = 255.0.0.0.


Class B :

-> first two bit are always set as 10.
-> first 16 bit (two octet) represent network address.
-> the range of first octet is 128-191.
-> so, total network address in class A = 2^14. and all of them are useful. i.e. we can assign them to any network.
-> for each network, total possible host = 2^16 
-> host address x.y.0.0 is used as identifying address of that network. so it is not assigned to any host. similarly, address x.y.255.255 is used as broadcast address. so it is also not assigned to any host.
-> so total number of useful host addresses in each network of class A = 2^16  - 2.
-> Default mask of class A is = 255.255.0.0.

Class C :

-> first three bit are always set as 110.
-> first 24 bit (three octet) represent network address.
-> the range of first octet is 192-223.
-> so, total network address in class A = 2^21. and all of them are useful. i.e. we can assign them to any network.
-> for each network, total possible host = 2^8 
-> host address x.y.z.0 is used as identifying address of that network. so it is not assigned to any host. similarly, address x.y.z.255 is used as broadcast address. so it is also not assigned to any host.
-> so total number of useful host addresses in each network of class A = 2^8  - 2.
-> Default mask of class A is = 255.255.255.0.


Class D :

-> first four bit are always set as 1110.
-> the range of first octet is 224-239.
-> total possible IP addresses of Class D = 2^28 
-> all of these IP addresses are reserved for multicasting, group email etc.
-> so there is no such thing as network and host in Class D.



Class E :

-> first four bit are always set as 1111.
-> the range of first octet is 240-255.
-> total possible IP addresses of Class D = 2^28 
-> all of these IP addresses are reserved for military purposes etc.
-> so there is no such thing as network and host in Class E.



-> for subnetting use [This video](https://youtu.be/rdb2ki4iGuo?si=Q0_qo92U5l1hsTGQ) 
-> for classless addressing, use [This video](https://youtu.be/N-ywmOpWehE?si=mr-x_7iS_kfstZXo) 
-> for unicast, multicast and broadcast, use [this article](https://www.geeksforgeeks.org/difference-between-unicast-broadcast-and-multicast-in-computer-network/)

unicast : one to one communication
multicast : one to many communication
broadcast : one to every one communication. it is of two type. 
1. limited broadcast : when a host want to send message to all other hosts of the same network it is residing. it is called limited broadcast. it will use destination address = 255.255.255.255
2. direct broadcast : when a host want to send message to all host of different network. it is called direct broadcasting. for that it will use broadcast address of that network.