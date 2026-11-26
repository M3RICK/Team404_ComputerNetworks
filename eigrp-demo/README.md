# EIGRP Demo Cheatsheet

Side demo, separate from the hospital network. Finished result: `eigrp.pkt`.

```
 192.168.1.0/24                                        192.168.3.0/24
 PC0 - Switch0 - Router0 ----- 10.0.13.0/30 ----- Router2 - Switch2 - PC2
                      \                            /
             10.0.12.0/30                 10.0.23.0/30
                        \                        /
                         -------- Router1 -------
                                     |
                              Switch1 - PC1
                             192.168.2.0/24
```

Ports are in the cable table below.

## Devices and cables

3 routers **2911**, 3 switches **2960-24TT**, 3 **PC**. Switches need no configuration.

| From | To | Cable |
|---|---|---|
| Router0 g0/0 | Switch0 Fa0/1 | Copper Straight-Through |
| Router1 g0/0 | Switch1 Fa0/1 | Copper Straight-Through |
| Router2 g0/0 | Switch2 Fa0/1 | Copper Straight-Through |
| Switch0 Fa0/2 | PC0 Fa0 | Copper Straight-Through |
| Switch1 Fa0/2 | PC1 Fa0 | Copper Straight-Through |
| Switch2 Fa0/2 | PC2 Fa0 | Copper Straight-Through |
| Router0 g0/1 | Router1 g0/1 | Copper Cross-Over |
| Router1 g0/2 | Router2 g0/1 | Copper Cross-Over |
| Router0 g0/2 | Router2 g0/2 | Copper Cross-Over |

## Addresses

| Device | Port | IP | Mask |
|---|---|---|---|
| Router0 | g0/0 | 192.168.1.1 | 255.255.255.0 |
| Router0 | g0/1 | 10.0.12.1 | 255.255.255.252 |
| Router0 | g0/2 | 10.0.13.1 | 255.255.255.252 |
| Router1 | g0/0 | 192.168.2.1 | 255.255.255.0 |
| Router1 | g0/1 | 10.0.12.2 | 255.255.255.252 |
| Router1 | g0/2 | 10.0.23.1 | 255.255.255.252 |
| Router2 | g0/0 | 192.168.3.1 | 255.255.255.0 |
| Router2 | g0/1 | 10.0.23.2 | 255.255.255.252 |
| Router2 | g0/2 | 10.0.13.2 | 255.255.255.252 |

PCs: Desktop > IP Configuration > Static

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC2 | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |

## Router0

CLI tab, answer `no` to the setup dialog, then:

```
enable
conf t
hostname Router0
int g0/0
ip address 192.168.1.1 255.255.255.0
no shut
int g0/1
ip address 10.0.12.1 255.255.255.252
no shut
int g0/2
ip address 10.0.13.1 255.255.255.252
no shut
router eigrp 100
network 192.168.1.0
network 10.0.12.0 0.0.0.3
network 10.0.13.0 0.0.0.3
end
```

## Router1

```
enable
conf t
hostname Router1
int g0/0
ip address 192.168.2.1 255.255.255.0
no shut
int g0/1
ip address 10.0.12.2 255.255.255.252
no shut
int g0/2
ip address 10.0.23.1 255.255.255.252
no shut
router eigrp 100
network 192.168.2.0
network 10.0.12.0 0.0.0.3
network 10.0.23.0 0.0.0.3
end
```

Expected: `%DUAL-5-NBRCHANGE: IP-EIGRP 100: Neighbor 10.0.12.1 (GigabitEthernet0/1) is up: new adjacency`

## Router2

```
enable
conf t
hostname Router2
int g0/0
ip address 192.168.3.1 255.255.255.0
no shut
int g0/1
ip address 10.0.23.2 255.255.255.252
no shut
int g0/2
ip address 10.0.13.2 255.255.255.252
no shut
router eigrp 100
network 192.168.3.0
network 10.0.23.0 0.0.0.3
network 10.0.13.0 0.0.0.3
end
```

Expected: two `new adjacency` messages, one for Router1 and one for Router0.

`0.0.0.3` is a wildcard mask (inverse of `255.255.255.252`): EIGRP runs only on that exact /30 link.

## Verify

On a router:

```
show ip eigrp neighbors
show ip route
show ip eigrp topology
show ip protocols
```

- `neighbors`: 2 neighbors on every router.
- `route`: 2 `D` routes to the other LANs, `[90/...]`.

From PC0 (Desktop > Command Prompt):

```
ping 192.168.2.10
ping 192.168.3.10
```

## Failover

1. PC0: `tracert 192.168.3.10`. Path: Router0 then Router2 (10.0.13.2).
2. Router0: cut the direct link to Router2.
   ```
   conf t
   int g0/2
   shutdown
   end
   ```
3. PC0: `tracert 192.168.3.10` again. Path: Router0, Router1 (10.0.12.2), then Router2.
4. Router0: restore it.
   ```
   conf t
   int g0/2
   no shut
   end
   ```
