# EIGRP Demo: Speaker Script

Read aloud while the demo runs. Each part says what is happening on screen.

## 1. Introduction (before starting)

"Today we show EIGRP, the Enhanced Interior Gateway Routing Protocol.
It is a routing protocol created by Cisco. Its job is to let routers share the networks they know,
so nobody has to type routes by hand.
We will build a small network together: two PCs, two routers, three networks.
Please follow along on your own Packet Tracer."

## 2. Devices and cables

"We place two 2911 routers and two PCs.
We connect them with copper cross-over cables, because we link a PC to a router and a router to a router, with no switch.
PC1 goes on port g0/0 of R1. R1 and R2 are connected on g0/1. PC2 goes on g0/0 of R2.
The links are red for now: router ports are off by default."

## 3. PC addresses

"Each PC gets an IP address, a mask and a default gateway.
PC1 is 192.168.1.10, PC2 is 192.168.2.10.
The gateway is the address of the router on the PC's network: 192.168.1.1 for PC1, 192.168.2.1 for PC2.
When a PC wants to reach another network, it sends everything to its gateway."

## 4. Router interfaces

"Now we configure R1 in the CLI.
`enable` and `conf t` take us to configuration mode.
On each port we set an IP address, then `no shut` turns the port on.
Look: the link to PC1 turns green.
At this point R1 only knows its own two networks. It does not know where PC2's network is,
so a ping from PC1 to PC2 would fail."

## 5. EIGRP on R1

"`router eigrp 100` starts EIGRP. 100 is the autonomous system number.
It must be the same on every router, otherwise they ignore each other.
The `network` commands tell EIGRP which networks to advertise and on which ports to look for neighbors.
R1 now sends Hello packets every five seconds, but nobody answers yet."

## 6. R2: the adjacency

"We do the same on R2, with its own addresses and the same AS number, 100.
Watch the console right after the last `network` command."

(The `new adjacency` message appears.)

"This message comes from DUAL, the algorithm behind EIGRP.
R1 and R2 found each other with Hello packets, checked they have the same AS number and the same network,
and became neighbors. They immediately exchanged their routes."

## 7. Verification

"`show ip eigrp neighbors`: R2 sees R1 as its neighbor.
`show ip route`: here is a line starting with D. D means a route learned by EIGRP.
In brackets, 90 is the administrative distance of EIGRP, and the second number is the metric.
Nobody typed this route: EIGRP learned it.
`show ip protocols`: K1 and K3 are set to 1. It means the metric is calculated from the bandwidth and the delay of the links."

"And now the ping from PC1 to PC2 works.
The first packet may time out: the router is resolving the MAC address with ARP. The next ones get a reply."

## 8. Conclusion

"To sum up, EIGRP:
- is a Cisco protocol, called advanced distance vector;
- finds neighbors automatically with Hello packets;
- uses bandwidth and delay for its metric;
- uses the DUAL algorithm, which keeps a backup route ready when one exists, so it converges very fast;
- has an administrative distance of 90, so a router prefers it over OSPF (110) and RIP (120).
Thank you. Any questions?"

## Likely questions

| Question | Answer |
|---|---|
| Why a cross-over cable? | Same type of device on both ends (PC to router, router to router), no switch. |
| What if the AS numbers differ? | The routers never become neighbors and no routes are exchanged. |
| What is the feasible successor? | A backup route DUAL keeps ready. If the best route fails, it switches instantly. |
| Why administrative distance 90? | Cisco trusts EIGRP more than OSPF (110) and RIP (120) when they offer the same network. |
| Why did the first ping fail? | ARP: the router had to learn the PC's MAC address first. |
| EIGRP vs OSPF? | EIGRP: Cisco, distance vector, bandwidth + delay. OSPF: open standard, link state, cost from bandwidth. |
