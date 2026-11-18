# Lab 02: Layer 3 Forwarding and OSPF

## Overview
This lab demonstrates Layer 3 forwarding and OSPF dynamic routing using five routers and one PC.

## Objectives
- Configure OSPF on all five routers.
- Analyze Layer 3 forwarding decisions.
- Identify the next-hop router selected by R1.
- Verify routing decisions using the routing table.
- Verify packet paths using traceroute.

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

## Verification Commands
```cisco
show ip route <destination>
traceroute <destination>
```

## Lab Evidence
Screenshots and verification results will be added here.

**Course:** CCNP Enterprise  
**Lab:** 02  
**Topic:** Layer 3 Forwarding and OSPF


## Verification and Testing

The following Cisco IOS commands were used to verify the network:

- `show ip interface brief` – Verify interface status and IP addressing.
- `show ip route` – Examine the routing table.
- `show ip ospf neighbor` – Verify OSPF neighbor relationships.
- `traceroute <destination>` – Examine packet forwarding paths.
- `show ip route <destination>` – Identify the selected route and next hop.

## Evidence

Screenshots of the verification commands and their outputs are included in this lab.
