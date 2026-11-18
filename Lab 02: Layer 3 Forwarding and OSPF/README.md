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

## Lab Evidence

- **Topology:** Network topology diagram.
- **Verification:** Screenshots of interface status, routing tables, OSPF neighbors, and traceroute results.

See the `Topology` and `Verification` folders for the practical evidence.

## Course Information

- **Course:** CCNP Enterprise
- **Lab:** 02
- **Topic:** Layer 3 Forwarding and OSPF
