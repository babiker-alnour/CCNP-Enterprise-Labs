# Lab 04 – GRE Tunnel, MTU, Path MTU Discovery & IP Fragmentation

## 1. Overview

This lab explores Generic Routing Encapsulation (GRE) tunnels and the impact of MTU (Maximum Transmission Unit) on packet transmission.

The lab demonstrates how GRE encapsulation adds overhead, how tunnel MTU affects packet delivery, and how the Don't Fragment (DF) bit influences fragmentation.

It also examines Path MTU Discovery (PMTUD) and tests end-to-end connectivity through a GRE tunnel.

## 2. Objectives

- Configure a GRE tunnel between two Cisco routers.
- Verify GRE tunnel connectivity.
- Understand the difference between physical interface MTU and tunnel MTU.
- Calculate GRE encapsulation overhead.
- Test packet delivery with different MTU values.
- Observe the effect of the DF bit.
- Understand fragmentation behavior and Path MTU Discovery.

## 3. Lab Topology

The topology consists of two GRE tunnel endpoints connected through an ISP router. R2 connects to R3 through an internal network.

![GRE Lab Topology](topology/topology.png)

### Devices

- R1 – GRE tunnel endpoint
- ISP – Underlay network router
- R2 – GRE tunnel endpoint
- R3 – Internal destination router
- SW1 – Layer 2 switch

## 4. IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | Fa0/0 | 203.0.111.1 | 255.255.255.0 |
| ISP | Fa0/0 | 203.0.111.100 | 255.255.255.0 |
| ISP | Fa1/0 | 203.0.112.100 | 255.255.255.0 |
| R2 | Fa0/0 | 203.0.112.2 | 255.255.255.0 |
| R2 | Fa1/0 | 192.168.23.2 | 255.255.255.0 |
| R3 | Fa0/0 | 192.168.23.3 | 255.255.255.0 |
| R1 | Tunnel0 | 192.168.12.1 | 255.255.255.252 |
| R2 | Tunnel0 | 192.168.12.2 | 255.255.255.252 |

## 5. GRE Tunnel Configuration

### R1 Configuration

```cisco
interface Tunnel0
 ip address 192.168.12.1 255.255.255.252
 ip mtu 1476
 tunnel source FastEthernet0/0
 tunnel destination 203.0.112.2
 tunnel path-mtu-discovery
```

### R2 Configuration

```cisco
interface Tunnel0
 ip address 192.168.12.2 255.255.255.252
 ip mtu 1476
 tunnel source 203.0.112.2
 tunnel destination 203.0.111.1
```

## 6. Baseline Connectivity Verification

Before performing the MTU tests, connectivity was verified.

From R1:

| Destination | Result |
|---|---|
| 203.0.112.2 | 5/5 successful |
| 192.168.12.2 | 5/5 successful |
| 192.168.23.3 | 5/5 successful |

The GRE tunnel was operational, and R1 could reach the internal destination behind R2.

## 7. Exercise 1 – Underlay MTU Testing

The first test examined the maximum successful packet size across the physical underlay between R1 and R2.

An extended ping was performed from R1 to 203.0.112.2 with the DF bit enabled.

### Results

| Datagram Size | Result |
|---|---|
| 1496 | Success |
| 1497 | Success |
| 1498 | Success |
| 1499 | Success |
| 1500 | Success |
| 1501–1505 | Timeout |

**Observation:**

The largest successful datagram size in this test was 1500 bytes. Larger tested sizes did not receive replies.

## 8. Exercise 2 – GRE Tunnel MTU Verification

GRE encapsulation adds additional headers to the original IP packet.

For standard IPv4 GRE encapsulation:

- Outer IPv4 header: 20 bytes
- GRE header: 4 bytes
- Total overhead: 24 bytes

The effective tunnel MTU is calculated as:

```text
Underlay MTU - GRE overhead

1500 - 24 = 1476 bytes
```

### Verification Commands

```cisco
show interfaces Tunnel0
show ip interface Tunnel0
```

### Observed Results

Both R1 and R2 reported:

| Parameter | Value |
|---|---|
| Logical interface MTU (`show interfaces`) | 17916 bytes |
| Tunnel transport MTU | 1476 bytes |
| IP MTU (`show ip interface`) | 1476 bytes |

**Observation:**

The effective IP MTU of the GRE tunnel was 1476 bytes, which matches the calculated value.

The 17916-byte value shown by `show interfaces Tunnel0` is the logical interface MTU; it should not be confused with the effective IP MTU.

![R1 GRE Tunnel Up](screenshots/01-R1-gre-tunnel-up.png)

![R2 GRE Tunnel Up](screenshots/01-R2-gre-tunnel-up.png)

## 9. Exercise 3 – Configuring GRE Tunnel MTU to 1500

The IP MTU was temporarily changed to 1500 bytes on both R1 and R2.

### Configuration

```cisco
configure terminal
interface Tunnel0
 ip mtu 1500
end
```

### IOS Warning

Both routers displayed a warning indicating that the configured IP MTU exceeded the tunnel transport MTU of 1476 bytes.

```text
%Warning: IP MTU value set 1500 is greater than the current transport value 1476, fragmentation may occur

%TUN-4-MTUCONFIGEXCEEDSTRMTU_IPV4: Tunnel0 IPv4 MTU configured 1500 exceeds tunnel transport MTU 1476
```

### Ping Tests

From R1 to the remote tunnel endpoint 192.168.12.2:

| Datagram Size | DF Bit | Result |
|---|---|---|
| 1500 | Disabled | Success |
| 1500 | Enabled | Timeout |

**Observation:**

With the DF bit disabled, the ping succeeded. With the DF bit enabled, it timed out.

This behavior is consistent with fragmentation being permitted when DF is disabled and prohibited when DF is enabled. However, packet capture would be required to confirm the actual fragmentation process.

### Restore the Original MTU

After the test, the original MTU was restored on both routers:

```cisco
configure terminal
interface Tunnel0
 ip mtu 1476
end
```

## 10. Exercise 4 – End-to-End MTU Testing

The final test examined packet delivery from R1 to R3 through the GRE tunnel.

Destination: `192.168.23.3`

Source: `192.168.12.1`

### Results

| Datagram Size | DF Bit | Result |
|---|---|---|
| 1496 | Enabled | Timeout |
| 1476 | Enabled | Success |
| 1477 | Enabled | Timeout |
| 1477 | Disabled | Success |

**Observation:**

The largest successful tested datagram size with DF enabled was 1476 bytes.

The 1477-byte test failed with DF enabled but succeeded with DF disabled. This is consistent with fragmentation being permitted when DF is disabled.

A packet capture would be needed to verify the actual fragments on the network.

## 11. Path MTU Discovery (PMTUD)

Path MTU Discovery helps determine the maximum packet size that can travel across a network path without fragmentation.

In this lab, the following command was configured on R1:

```cisco
interface Tunnel0
 tunnel path-mtu-discovery
```

The command enables tunnel Path MTU Discovery behavior on R1.

The tunnel configuration was verified using:

```cisco
show interfaces Tunnel0
show ip interface Tunnel0
```

The R1 output displayed Path MTU Discovery information.

**Important:** The configuration was present on R1. It was not configured on R2 during this lab.

## 12. Final Verification

After completing the tests, the tunnel MTU was restored to 1476 bytes on both routers.

Final verification commands:

```cisco
show ip interface Tunnel0
show interfaces Tunnel0
```

### Final Results

| Parameter | R1 | R2 |
|---|---|---|
| Tunnel status | Up/Up | Up/Up |
| Tunnel IP | 192.168.12.1/30 | 192.168.12.2/30 |
| IP MTU | 1476 bytes | 1476 bytes |
| Tunnel transport MTU | 1476 bytes | 1476 bytes |
| Input errors | 0 | 0 |
| Output errors | 0 | 0 |
| Output drops | 0 | 0 |
| Tunnel PMTUD command | Configured | Not configured |

## 13. Key Technical Concepts

### GRE Encapsulation

GRE encapsulates original packets inside additional headers so that they can be transported across an IP network.

### MTU

MTU defines the maximum packet size that can be transmitted over a network interface without fragmentation at that layer.

### GRE Overhead

Standard IPv4 GRE encapsulation adds 24 bytes of overhead:

```text
20 bytes IPv4 header + 4 bytes GRE header = 24 bytes
```

### DF Bit

- DF = 0: Fragmentation is permitted when required.
- DF = 1: Fragmentation is prohibited.

### Path MTU Discovery

PMTUD helps discover the smallest MTU along a network path so that packets can be sent without requiring fragmentation.

## 14. Lab Environment

- Cisco IOS routers
- GNS3
- GRE tunnel
- IPv4 addressing
- Extended ping testing
- MTU and DF-bit verification

## 15. Conclusion

This lab demonstrated how GRE encapsulation affects packet size and tunnel MTU.

The main findings were:

1. Standard IPv4 GRE encapsulation adds 24 bytes of overhead.
2. With a 1500-byte underlay MTU, the calculated GRE tunnel IP MTU is 1476 bytes.
3. Configuring a tunnel IP MTU greater than the transport MTU can produce IOS warnings and may lead to fragmentation.
4. The DF bit affects whether fragmentation is permitted.
5. The end-to-end tests demonstrated the importance of selecting an appropriate tunnel MTU.

The lab also reinforced the importance of verifying tunnel interfaces and testing packet sizes rather than relying only on tunnel status.

**Note:** The results in this documentation are based on the actual ping tests and router outputs collected during the lab. Packet fragmentation itself was not independently verified using a packet capture.

## 16. Repository Structure

```text
Lab 04 - GRE Tunnel, MTU, Path MTU Discovery & IP Fragmentation/
│
├── README.md
├── topology/
│   └── topology.png
│
└── screenshots/
    ├── 01-R1-gre-tunnel-up.png
    ├── 01-R2-gre-tunnel-up.png
    ├── 02-underlay-mtu.png
    ├── 03-R1-gre-default-mtu.png
    ├── 03-R2-gre-default-mtu.png
    ├── 04-R1-gre-mtu-1500.png
    └── 05-R1-end-to-end-mtu.png
```

---

**Author:** Babiker Alnour Babiker Omer  
**Repository:** [CCNP Enterprise Labs](https://github.com/babiker-alnour/CCNP-Enterprise-Labs)
