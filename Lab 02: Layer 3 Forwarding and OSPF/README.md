# Lab 02: Layer 3 Forwarding and OSPF

## Overview

This lab demonstrates Layer 3 forwarding and OSPF dynamic routing using five routers and one PC.

## Objectives

- Configure OSPF on all five routers.
- Analyze Layer 3 forwarding decisions.
- Identify the next-hop router selected by R1.
- Verify routing decisions using the routing table.
- Examine packet forwarding paths using traceroute.

## Network Devices

- 5 Routers
- 1 PC
- OSPF
- IPv4

## Destination Addresses

1. 203.0.113.133
2. 8.8.8.8
3. 172.20.20.27
4. 198.0.27.56
5. 192.168.255.238

## Verification and Testing

The following Cisco IOS commands were used:

| Command | Purpose |
|---|---|
| `show ip interface brief` | Verify interface status and IP addressing. |
| `show ip route` | Examine the routing table. |
| `show ip ospf neighbor` | Verify OSPF neighbor relationships. |
| `traceroute <destination>` | Examine packet forwarding paths. |
| `show ip route <destination>` | Identify the selected route and next hop. |



## Next-Hop Decision Analysis

R1's routing table was examined using the `show ip route <destination>` command for five remote loopback destinations hosted on R5.

All five destinations were learned through OSPF process 1 as intra-area routes.

### Routing Results

| Destination | Prefix | Next Hops | Metric |
|---|---|---|---|
| 203.0.113.133 | 203.0.113.133/32 | 192.168.14.4, 192.168.13.3, 192.168.12.2 | 3 |
| 8.8.8.8 | 8.8.8.8/32 | 192.168.14.4, 192.168.13.3, 192.168.12.2 | 3 |
| 172.20.20.27 | 172.20.20.27/32 | 192.168.14.4, 192.168.13.3, 192.168.12.2 | 3 |
| 198.0.27.56 | 198.0.27.56/32 | 192.168.14.4, 192.168.13.3, 192.168.12.2 | 3 |
| 192.168.255.238 | 192.168.255.238/32 | 192.168.14.4, 192.168.13.3, 192.168.12.2 | 3 |

### Next-Hop Mapping

| Next-Hop Router | Next-Hop IP | Exit Interface |
|---|---|---|
| R4 | 192.168.14.4 | FastEthernet2/0 |
| R3 | 192.168.13.3 | FastEthernet1/1 |
| R2 | 192.168.12.2 | FastEthernet1/0 |

### ECMP Analysis

All five destinations have three equal-cost OSPF paths with an administrative distance of 110 and a metric of 3.

R1 installs all three next hops in its routing table, demonstrating Equal-Cost Multipath (ECMP).

This allows R1 to perform load sharing across the three available paths toward R5.

### Technical Conclusion

The routing table confirms successful OSPF route installation and equal-cost path selection for all five remote destinations.

## Lab Evidence

- **Topology:** Network topology diagram.
- **Verification:** Screenshots of interface status, routing tables, OSPF neighbors, and traceroute results.

See the `Topology` and `Verification` folders for the practical evidence.

## Course Information

- **Course:** CCNP Enterprise
- **Lab:** 02
- **Topic:** Layer 3 Forwarding and OSPF
