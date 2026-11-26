# Secure Network for a Multi-Site Hospital

Computer Networks [ISE2232] project, Team 404. Built in Cisco Packet Tracer 9.0.

## Open it

Open `team404.pkt` in Cisco Packet Tracer 9.0 or newer. Wait about a minute for STP and OSPF to converge; if a PC has no address, run `ipconfig /renew` in its Command Prompt.

Separate side demo: [`eigrp-demo/`](eigrp-demo/) (EIGRP between two routers: cheatsheet, speaker script, `.pkt`).

## Topology

```
 EXT-WEB   EXT-PC                     SW-A (Bldg A)        SW-B (Bldg B)
     \      /                        MED1 MED2 GUEST1/2      ADM1 ADM2
      ISP ──── FW ──────────────── CORE (L3 switch) ──── SW-DC: SRV-DNS, SRV-RECORDS, SRV-BILLING
               │                    │          │
            SW-DMZ               CLINIC1    CLINIC2
            SRV-WEB              SW-C1/PC   SW-C2/PC
```

| Link | Ports |
|---|---|
| ISP – FW | Gi0/0 – Gi0/0 |
| FW – SW-DMZ | Gi0/1 – Gi0/1 |
| FW – CORE | Gi0/2 – Gi0/1 |
| CORE – SW-A / SW-B / SW-DC | Fa0/1 / Fa0/2 / Fa0/3 – Gi0/1 |
| CORE – CLINIC1 / CLINIC2 | Fa0/4 / Fa0/5 – Gi0/0 |
| CLINICx – SW-Cx | Gi0/1 – Gi0/1 |

## Addressing (VLSM on 10.10.0.0/16)

| Subnet | VLAN | Network | Gateway | Hosts |
|---|---|---|---|---|
| Guest | 30 | 10.10.0.0/24 | 10.10.0.1 (CORE) | DHCP |
| Medical staff | 10 | 10.10.1.0/25 | 10.10.1.1 (CORE) | DHCP |
| Administration | 20 | 10.10.1.128/26 | 10.10.1.129 (CORE) | DHCP |
| Clinic 1 | – | 10.10.1.192/27 | 10.10.1.193 (CLINIC1) | DHCP |
| Clinic 2 | – | 10.10.1.224/27 | 10.10.1.225 (CLINIC2) | DHCP |
| Servers | 40 | 10.10.2.0/28 | 10.10.2.1 (CORE) | DNS+DHCP .2, Records .3, Billing .4 |
| DMZ | – | 10.10.2.16/29 | 10.10.2.17 (FW) | Web .18 (public 203.0.113.3) |
| FW – CORE | – | 10.10.2.24/30 | | .25 / .26 |
| CORE – CLINIC1 | – | 10.10.2.28/30 | | .29 / .30 |
| CORE – CLINIC2 | – | 10.10.2.32/30 | | .33 / .34 |
| ISP – FW (public) | – | 203.0.113.0/29 | | ISP .1, FW .2, static NAT .3 |
| Internet server / user | – | 198.51.100.0/24, 192.0.2.0/24 | .1 (ISP) | .10 |

Native VLAN 99 on trunks. Credentials: `admin` / `Admin404`, enable secret `Hospital404`, SSH only.

## Services

| Service | Where | Detail |
|---|---|---|
| VLAN | SW-A, SW-B, SW-DC, CORE | 10 Medical, 20 Admin, 30 Guest, 40 Servers, 99 native |
| OSPF | FW, CORE, CLINIC1, CLINIC2 | area 0, FW injects the default route |
| DHCP | SRV-DNS, relayed by CORE and clinics (`ip helper-address`) | one pool per subnet |
| DNS | SRV-DNS | `records`, `billing`, `www`, `ns` `.hospital.local`, `www.internet.com` |
| NAT | FW | PAT for 10.10.0.0/16, static NAT for the public website |
| ACL | CORE `SERVERS-OUT`, `GUEST-IN`; FW `OUTSIDE-IN`, `DMZ-IN` | access policy below |
| HTTP / FTP | SRV-WEB, SRV-RECORDS, SRV-BILLING / SRV-RECORDS | FTP login `cisco` / `cisco` |
| Security extras | all switches / all devices | port security (sticky, max 2), SSH v2 only |

Access policy:

| From \ To | Records | Billing | Public website | Internet |
|---|---|---|---|---|
| Medical staff | yes | no | yes | yes |
| Administration | no | yes | yes | yes |
| Clinics | yes | no | yes | yes |
| Guests | no | no | yes | yes |
| Internet | no | no | yes (203.0.113.3) | – |

## Test plan

| # | Check | Where | Expected |
|---|---|---|---|
| 1 | DHCP lease | PC-MED1 `ipconfig` | 10.10.1.x/25, gw 10.10.1.1, DNS 10.10.2.2 |
| 2 | VLANs and trunks | SW-A `show vlan brief`, `show interfaces trunk` | Fa0/1-10 in 10, Fa0/11-20 in 30, Gi0/1 trunking |
| 3 | OSPF | CORE `show ip ospf neighbor`, `show ip route` | 3 neighbours (FW, CLINIC1, CLINIC2), `O*E2` default route |
| 4 | Inter-department | PC-MED1 `ping` PC-C1, `tracert` PC-C2 | replies, path through CORE |
| 5 | DNS + HTTP | PC-MED1 browser `http://records.hospital.local` | Patient Records page |
| 6 | FTP | PC-C1 `ftp 10.10.2.3` (cisco/cisco) | login OK |
| 7 | ACL: admin to records | PC-ADM1 browser `records.hospital.local` | timeout; CORE `show access-lists SERVERS-OUT` deny counter rises |
| 8 | ACL: admin to billing | PC-ADM1 browser `billing.hospital.local` | Billing page |
| 9 | ACL: guest isolation | GUEST2 `ping 10.10.1.1x` (a medical PC) | fails; `www.hospital.local` and `www.internet.com` work |
| 10 | NAT outbound | PC-MED1 browser `www.internet.com`, then FW `show ip nat translations` | page loads, PAT entry 203.0.113.2 |
| 11 | NAT inbound / DMZ | EXT-PC browser `http://203.0.113.3` | public website; `http://10.10.2.3` unreachable |
| 12 | DMZ isolation | SRV-WEB `ping 10.10.2.3` | fails (DMZ-IN) |
| 13 | SSH | PC-ADM1 `ssh -l admin 10.10.1.129` | login OK; telnet refused |
| 14 | Port security | SW-A `show port-security` | sticky MACs learned |

## Scope notes

- The firewall is a 2911 router (NAT + ACLs + DMZ), the fallback planned in the proposal.
- The clinics use private WAN links to CORE. IPsec was dropped: the clinic traffic ends on a layer 3 switch, which cannot terminate a VPN.
- The guest network is wired; a Wi-Fi access point would add nothing to the protocol list.
