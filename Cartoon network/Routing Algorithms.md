
1. Adaptive or dynamic routing algorithms -> routing tables gets updated based on the present conditions of the network and it's traffic.
2. Non adaptive or static routing algorithms -> ones the routing tables built, routing of packets happens based on that table. table doesn't updated based on current network conditions.
3. Hybrid routing algorithms -> they are the combination of both adaptive and non adaptive algorithms. they can be classified further as 1. Distance vector 2. Link State.

Distance vector : in this algo., router calculates distances to every possible destination based on its immediate neighbors only, the router’s routing table is shared with routers that are directly connected with it i.e. it's neighbors, during regular intervals. Bellman ford algo. is used to calculate distances. 

-> count to infinity problem occurs in Distance vector method.


Link State : in this algo., the router shares it's routing table to all the router present in network also  learns from all of them. Dijkstra's algorithm is used.