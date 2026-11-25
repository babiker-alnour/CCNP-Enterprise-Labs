
# GRE Tunnel, MTU, Path MTU Discovery & IP Fragmentation

![CCNP Enterprise](https://img.shields.io/badge/Track-CCNP%20Enterprise-blue)
![Cisco IOS](https://img.shields.io/badge/Platform-Cisco%20IOS-red)
![Topic](https://img.shields.io/badge/Topic-GRE%20%7C%20MTU%20%7C%20Fragmentation-green)
![Status](https://img.shields.io/badge/Status-In%20Progress-orange)

## 1. Project Overview

This lab explores GRE tunneling, Maximum Transmission Unit (MTU), IP fragmentation, and the Don't Fragment (DF) bit using Cisco IOS routers.

The objective is to understand how GRE encapsulation affects packet size, investigate fragmentation behavior, and troubleshoot MTU-related connectivity problems.

This is a practical CCNP Enterprise lab focused on network configuration, packet analysis, and troubleshooting.

## 2. Lab Objectives

- Perform extended ping tests using different packet sizes.
- Understand the DF bit and its effect on IP fragmentation.
- Configure and verify a GRE tunnel.
- Investigate the default GRE tunnel MTU.
- Modify the tunnel IP MTU to 1500 bytes.
- Compare ping results with DF enabled and disabled.
- Observe IP fragmentation.
- Troubleshoot connectivity between R1 and R3.
- Identify and resolve MTU-related problems.

## 3. Network Topology

**Topology description:**

R1 connects to an ISP router. The ISP connects to R2. R2 connects to SW1, which connects to R3.

The GRE tunnel is established between R1 and R2.

### Network addressing

| Device | Interface | IP address |
|---|---|---|
| R1 | FastEthernet0/0 | 203.0.111.1/24 |
| ISP | FastEthernet0/0 | 203.0.111.100/24 |
| ISP | FastEthernet1/0 | 203.0.112.100/24 |
| R2 | FastEthernet0/0 | 203.0.112.2/24 |
| R2 | FastEthernet1/0 | 192.168.23.2/24 |
| R3 | FastEthernet0/0 | 192.168.23.3/24 |

### GRE tunnel addressing

| Device | Interface | IP address |
|---|---|---|
| R1 | Tunnel0 | 192.168.12.1/30 |
| R2 | Tunnel0 | 192.168.12.2/30 |

## 4. Exercise 1 – Extended Ping and DF Bit

### Objective

Perform an extended ping from R1 to R2's physical WAN address.

Configure:
- DF bit: Enabled
- Minimum packet size: 1496 bytes
- Maximum packet size: 1505 bytes
- Increment: 1 byte
- Timeout: 1 second

### Command

```cisco
R1# ping
```

Extended ping parameters were entered interactively.

Destination:

```text
203.0.112.2
```

### Actual results

| Packet size | DF bit | Result |
|---:|---|---|
| 1496 | Enabled | Success |
| 1497 | Enabled | Success |
| 1498 | Enabled | Success |
| 1499 | Enabled | Success |
| 1500 | Enabled | Success |
| 1501 | Enabled | Failed |
| 1502 | Enabled | Failed |
| 1503 | Enabled | Failed |
| 1504 | Enabled | Failed |
| 1505 | Enabled | Failed |

**Test summary:**

- Successful packets: 5/10
- Failed packets: 5/10
- Maximum successful packet size: 1500 bytes.

### Technical analysis

The physical underlay interfaces operate with a 1500-byte MTU.

When the DF bit is enabled, routers cannot fragment an oversized IPv4 packet.

Therefore, packets exceeding the outgoing interface MTU cannot be forwarded without fragmentation.

The test demonstrates the packet-size limitation of the physical underlay path.

## 5. Exercise 2 – GRE Tunnel Configuration and MTU Verification

### Objective

Verify the existing GRE tunnel between R1 and R2 and investigate its MTU.

### GRE configuration

**R1:**

```cisco
interface Tunnel0
 ip address 192.168.12.1 255.255.255.252
 tunnel source 203.0.111.1
 tunnel destination 203.0.112.2
```

**R2:**

```cisco
interface Tunnel0
 ip address 192.168.12.2 255.255.255.252
 tunnel source 203.0.112.2
 tunnel destination 203.0.111.1
```

### Verification commands

```cisco
show interfaces tunnel0
show ip interface tunnel0
```

### Actual R1 results

From `show interfaces tunnel0`:

```text
MTU 17916 bytes
Tunnel transport MTU 1476 bytes
```

From `show ip interface tunnel0`:

```text
MTU is 1476 bytes
```

### Actual R2 results

From `show interfaces tunnel0`:

```text
MTU 17916 bytes
Tunnel transport MTU 1476 bytes
```

### MTU analysis

The GRE tunnel uses additional headers when encapsulating packets.

For a standard IPv4 GRE tunnel:

- Underlay MTU: 1500 bytes
- Outer IPv4 header: 20 bytes
- GRE header: 4 bytes

Calculation:

```text
1500 - 20 - 4 = 1476 bytes
```

The effective tunnel IP MTU is therefore 1476 bytes.

**Important distinction:**

The logical interface MTU reported by `show interfaces tunnel0` is 17916 bytes in this IOS environment. It should not be confused with the effective IP MTU reported by `show ip interface tunnel0`.

## 6. Exercise 3 – Tunnel MTU and IP Fragmentation

### Objective

Configure both GRE tunnel interfaces with an IP MTU of 1500 bytes and investigate the effect on packet fragmentation.

### Configuration on R1

```cisco
configure terminal
interface Tunnel0
 ip mtu 1500
end
```

### Configuration on R2

```cisco
configure terminal
interface Tunnel0
 ip mtu 1500
end
```

### Router warning

Both routers generated the following warning:

```text
%Warning: IP MTU value set 1500 is greater
than the current transport value 1476,
fragmentation may occur
```

IOS also generated:

```text
%TUN-4-MTUCONFIGEXCEEDSTRMTU_IPV4:
Tunnel0 IPv4 MTU configured 1500
exceeds tunnel transport MTU 1476
```

### Verification

Command:

```cisco
show ip interface tunnel0
```

R1:

```text
MTU is 1500 bytes
```

R2:

```text
MTU is 1500 bytes
```

**Analysis:**

The configured tunnel IP MTU is now larger than the tunnel transport MTU.

This creates a potential fragmentation condition when packets are encapsulated and transmitted over the physical underlay.

### Test 1 – 1500-byte ping without DF

Source: R1

Destination: R2 Tunnel0 (192.168.12.2)

Packet size: 1500 bytes

DF bit: Disabled

**Result:** Pending practical verification.

### Test 2 – 1500-byte ping with DF

Source: R1

Destination: R2 Tunnel0 (192.168.12.2)

Packet size: 1500 bytes

DF bit: Enabled

**Result:** Pending practical verification.

### Fragmentation analysis

The purpose of these tests is to compare how the routers handle oversized GRE-encapsulated packets when fragmentation is permitted versus when the DF bit prevents fragmentation.

Packet captures can be used to inspect:

- Outer IPv4 packet size.
- Fragment offsets.
- More Fragments (MF) flag.
- Don't Fragment (DF) flag.
- Number of fragments.

### Restore default tunnel MTU

After completing both tests, restore the default configuration on R1 and R2:

```cisco
configure terminal
interface Tunnel0
 no ip mtu
end
```

Verify:

```cisco
show ip interface tunnel0
```

Expected effective IP MTU:

```text
MTU is 1476 bytes
```

## 7. Exercise 4 – Troubleshooting R1 to R3

### Objective

Investigate packet delivery between R1 and R3 using different DF-bit settings.

### Initial tests

**Test 1:**

```text
Source: R1
Destination: R3 (192.168.23.3)
Packet size: 1476 bytes
DF bit: Enabled
```

**Test 2:**

```text
Source: R1
Destination: R3 (192.168.23.3)
Packet size: 1476 bytes
DF bit: Disabled
```

### Troubleshooting methodology

The following commands will be used:

```cisco
show ip route
show interfaces tunnel0
show ip interface tunnel0
show interfaces
show ip traffic
traceroute 192.168.23.3
```

Troubleshooting process:

1. Verify normal connectivity.
2. Check routing tables on R1, R2, and R3.
3. Verify GRE tunnel status.
4. Check physical interface MTUs.
5. Check tunnel IP MTU.
6. Investigate DF-bit behavior.
7. Inspect fragmentation.
8. Identify the root cause.
9. Implement the solution.
10. Repeat the original tests.

### Root cause

Pending practical investigation.

### Solution

Pending practical investigation.

### Final verification

| Test | Before fix | After fix |
|---|---|---|
| R1 to R3, 1476 bytes, DF enabled | Pending | Pending |
| R1 to R3, 1476 bytes, DF disabled | Pending | Pending |
| Normal connectivity | Successful | Pending |

## 8. Baseline Connectivity Verification

Before changing the MTU configuration, connectivity was verified from R1.

| Destination | Purpose | Result |
|---|---|---|
| 203.0.112.2 | Physical connectivity to R2 | 100% success |
| 192.168.12.2 | GRE tunnel connectivity | 100% success |
| 192.168.23.3 | Remote network connectivity | 100% success |

The initial GRE tunnel was operational before the MTU experiments.

## 9. Technical Concepts Learned

This lab demonstrates practical knowledge of:

- GRE encapsulation.
- IPv4 packet structure.
- Maximum Transmission Unit (MTU).
- Tunnel IP MTU.
- Transport MTU.
- DF-bit behavior.
- IP fragmentation and reassembly.
- Path MTU Discovery.
- Static routing.
- ICMP troubleshooting.
- Cisco IOS verification commands.
- Network troubleshooting methodology.

## 10. Lab Environment

- Network simulator: GNS3
- Network operating system: Cisco IOS
- Routers: R1, ISP, R2, R3
- Switch: SW1
- Tunnel technology: GRE over IPv4
- Underlay MTU: 1500 bytes

## 11. Repository Structure

```text
ccnp-enterprise-gre-mtu-fragmentation/
│
├── README.md
│
├── topology/
│   └── topology.png
│
├── configurations/
│   ├── R1.txt
│   ├── ISP.txt
│   ├── R2.txt
│   └── R3.txt
│
├── screenshots/
│   ├── 01-topology.png
│   ├── 02-extended-ping.png
│   ├── 03-gre-mtu-default.png
│   ├── 04-mtu-1500-warning.png
│   ├── 05-fragmentation-test.png
│   ├── 06-df-bit-test.png
│   ├── 07-troubleshooting.png
│   └── 08-final-verification.png
│
└── notes/
    └── troubleshooting.md
```

## 12. Conclusion

This lab investigates the relationship between GRE encapsulation, MTU limitations, and IPv4 fragmentation.

The experiments demonstrate why tunnel MTU configuration must consider the physical underlay MTU and the additional headers introduced by GRE.

The final objective is to identify and resolve the connectivity problem between R1 and R3 through practical troubleshooting and packet analysis.

---

**Author:** Babiker Alnour Babiker Omer  
**Learning Track:** CCNP Enterprise  
**Project:** GRE Tunnel, MTU and IP Fragmentation
