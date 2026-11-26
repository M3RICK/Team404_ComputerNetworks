# EIGRP Demo: Speaker Script

Read aloud while the demo runs. Each part says what is happening on screen.

## 1. Introduction

"Today we show EIGRP, the Enhanced Interior Gateway Routing Protocol.
It is a routing protocol created by Cisco. Its job is to let routers share the networks they know,
so nobody has to type routes by hand, and to find another path when a link fails.
We will build three sites, each with a router, a switch and a PC, and connect the routers in a triangle.
Please follow along on your own Packet Tracer."

## 2. Devices and cables

"We place three 2911 routers, three 2960 switches and three PCs.
Each PC is connected to its switch, and each switch to its router, with straight-through cables:
those are different types of devices.
The routers are connected to each other with cross-over cables: same type of device on both ends.
The triangle gives two paths between any two sites. This is what will let us show failover at the end.
The links to the routers are red for now: router ports are off by default."

## 3. Addressing

"Each site has its own LAN: 192.168.1.0, 192.168.2.0 and 192.168.3.0, with a /24 mask.
Each PC gets the address .10 of its LAN, and its default gateway is its router, .1.
When a PC wants to reach another network, it sends everything to its gateway.

Between the routers we use /30 networks: 10.0.12.0 between Router0 and Router1,
10.0.13.0 between Router0 and Router2, 10.0.23.0 between Router1 and Router2.
A /30 has exactly two usable addresses, one for each end of the cable.
The switches need no configuration: they just connect the PC to its router."

## 4. Router interfaces

"Now we configure Router0 in the CLI.
`enable` and `conf t` take us to configuration mode.
On each port we set an IP address, then `no shut` turns the port on.
Look: the links turn green.
At this point Router0 only knows its own three networks. It has no idea where the other LANs are,
so a ping from PC0 to PC1 would fail."

## 5. EIGRP on Router0

"`router eigrp 100` starts EIGRP. 100 is the autonomous system number.
It must be the same on every router, otherwise they ignore each other.
The `network` commands tell EIGRP which networks to advertise and on which ports to look for neighbors.
For the links we add a wildcard mask, 0.0.0.3. It is the inverse of the /30 mask,
so EIGRP runs exactly on that link and nowhere else.
Router0 now sends Hello packets every five seconds, but nobody answers yet."

## 6. Router1 and Router2: the adjacencies

"We do the same on Router1, with its own addresses and the same AS number, 100.
Watch the console right after the last `network` command."

(The `new adjacency` message appears.)

"This message comes from DUAL, the algorithm behind EIGRP.
Router0 and Router1 found each other with Hello packets, checked they have the same AS number and the same network,
and became neighbors. They immediately exchanged their routes."

"Now Router2. This time we get two messages: Router2 becomes neighbor with both Router0 and Router1."

## 7. Verification

"`show ip eigrp neighbors`: each router has two neighbors.
`show ip route`: here are lines starting with D. D means a route learned by EIGRP.
In brackets, 90 is the administrative distance of EIGRP, and the second number is the metric.
Nobody typed these routes: EIGRP learned them.
`show ip protocols`: K1 and K3 are set to 1. The metric is calculated from the bandwidth and the delay of the links."

"And now PC0 can ping PC1 and PC2.
The first packet may time out: the router is resolving the MAC address with ARP. The next ones get a reply."

## 8. Failover

"`show ip eigrp topology` shows, for each network, the best path, called the successor,
and when it exists a backup path that DUAL has already checked, called the feasible successor."

"From PC0 we trace the path to PC2: it goes directly from Router0 to Router2."

"Now we simulate a failure: we shut down the link between Router0 and Router2."

(The `Neighbor 10.0.13.2 ... is down` message appears.)

"EIGRP detects the lost neighbor and immediately switches to the other path.
We trace again: the traffic now goes through Router1. Nobody changed a route by hand.
We turn the link back on, and the direct path comes back."

## 9. Conclusion

"To sum up, EIGRP:
- is a Cisco protocol, called advanced distance vector;
- finds neighbors automatically with Hello packets;
- uses bandwidth and delay for its metric;
- uses the DUAL algorithm, which keeps a backup route ready, so it converges very fast;
- has an administrative distance of 90, so a router prefers it over OSPF (110) and RIP (120).
Thank you. Any questions?"

## Likely questions

| Question | Answer |
|---|---|
| Why straight-through and cross-over cables? | Straight-through between different devices (PC to switch, switch to router), cross-over between the same type (router to router). |
| Why /30 between routers? | Only two hosts on a point-to-point link: a /30 gives exactly two usable addresses and wastes none. |
| What is `0.0.0.3`? | A wildcard mask, the inverse of 255.255.255.252. It limits EIGRP to that exact /30. |
| What if the AS numbers differ? | The routers never become neighbors and no routes are exchanged. |
| What is the feasible successor? | A loop-free backup route DUAL keeps ready. If the best route fails, it switches instantly. |
| Why administrative distance 90? | Cisco trusts EIGRP more than OSPF (110) and RIP (120) when they offer the same network. |
| Why did the first ping fail? | ARP: the router had to learn the PC's MAC address first. |
| EIGRP vs OSPF? | EIGRP: Cisco, advanced distance vector, bandwidth + delay. OSPF: open standard, link state, cost from bandwidth. |
