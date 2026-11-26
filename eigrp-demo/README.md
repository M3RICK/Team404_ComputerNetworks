# EIGRP Demo Cheatsheet

Side demo, separate from the hospital network. Finished result: `eigrp.pkt`.

```
PC1 ---- R1 ---------- R2 ---- PC2
    g0/0     g0/1  g0/1    g0/0

192.168.1.0/24   10.0.0.0/24   192.168.2.0/24
```

Devices: 2 routers **2911**, 2 **PC**. All 3 cables: **Copper Cross-Over**.

## PC addresses

Desktop > IP Configuration > Static

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC1 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |

## R1

```
enable
conf t
hostname R1
int g0/0
ip address 192.168.1.1 255.255.255.0
no shut
int g0/1
ip address 10.0.0.1 255.255.255.0
no shut
router eigrp 100
network 192.168.1.0
network 10.0.0.0
end
```

## R2

```
enable
conf t
hostname R2
int g0/0
ip address 192.168.2.1 255.255.255.0
no shut
int g0/1
ip address 10.0.0.2 255.255.255.0
no shut
router eigrp 100
network 192.168.2.0
network 10.0.0.0
end
```

Expected right after `network 10.0.0.0` on R2:

```
%DUAL-5-NBRCHANGE: IP-EIGRP 100: Neighbor 10.0.0.1 (GigabitEthernet0/1) is up: new adjacency
```

## Verify

```
show ip eigrp neighbors
show ip route
show ip protocols
```

From PC1 (Desktop > Command Prompt):

```
ping 192.168.2.10
```
