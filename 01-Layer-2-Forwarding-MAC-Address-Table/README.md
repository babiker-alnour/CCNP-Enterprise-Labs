# Lab 01: Layer 2 Forwarding and MAC Address Table

## 1. Lab Overview

This lab explores how Ethernet switches learn MAC addresses and forward frames within a Layer 2 network.

The lab was built using GNS3 and Cisco IOS, with Wireshark used to capture and analyze network traffic.

## 2. Lab Objectives

- Understand how switches learn source MAC addresses.
- Examine the MAC address table.
- Understand Layer 2 frame forwarding.
- Identify unicast, broadcast, and multicast traffic.
- Configure static MAC address entries.
- Understand MAC address aging.

## 3. Network Topology

The topology contains one router, one Layer 2 switch, and three PCs.

| Device | Interface | IP Address |
|---|---|---|
| R1 | FastEthernet0/0 | 192.168.60.100/24 |
| PC1 | Ethernet0 | 192.168.60.1/24 |
| PC2 | Ethernet0 | 192.168.60.2/24 |
| PC3 | Ethernet0 | 192.168.60.3/24 |

All devices are connected through the same Layer 2 switch in VLAN 1.

## 4. Technologies and Tools

- GNS3
- Cisco IOS
- Wireshark
- Ethernet switching
- MAC address learning

## 5. Lab Activities

1. Built the network topology in GNS3.
2. Generated network traffic between devices.
3. Captured Ethernet frames using Wireshark.
4. Examined unicast, broadcast, and multicast traffic.
5. Investigated MAC address learning and forwarding.
6. Practiced static MAC address configuration.
7. Examined MAC address aging behavior.

## 6. Verification Commands

```cisco
show mac address-table
show mac address-table dynamic
show mac address-table static
show mac address-table aging-time
show running-config
```

Disable dynamic MAC address aging:

```cisco
mac address-table aging-time 0
```

## 7. Screenshots

See the [screenshots folder](./screenshots/) for the network topology, Wireshark captures, and MAC address table evidence.

## 8. GNS3 Project

The GNS3 project file and supporting project files are available in the [topology folder](./topology/).

**Note:** Cisco IOS images are not included. You must provide compatible images in your own GNS3 installation.

## 9. Key Learning Outcomes

This lab provides practical experience with Ethernet switching, MAC address learning, frame forwarding, and traffic analysis at Layer 2.
