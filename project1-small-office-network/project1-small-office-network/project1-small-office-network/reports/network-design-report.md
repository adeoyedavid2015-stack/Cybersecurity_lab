# Network Design Report — Small Office Network

## 1. Objective
Design and implement a small office network for three departments
(Admin, Sales, and IT/Server) using proper IPv4 subnetting (VLSM),
a central router for inter-department routing, and managed switches
for each department. Verify connectivity between all departments
using ICMP ping tests.

## 2. Environment
- **Simulation Tool:** Cisco Packet Tracer 9.0
- **Router:** Cisco 2911
- **Switches:** Cisco 2960-24TT (x3)
- **End Devices:** PCs, Laptops, Server
- **Base Address Block:** 192.168.10.0/24

## 3. Network Requirements
| Department | Devices | Hosts Required |
|---|---|---|
| Sales | PC2, Laptop1, PC3 | 3 (planned for up to 30) |
| Admin | PC1, PC0, Laptop0 | 3 (planned for up to 14) |
| IT/Server | PC6, Server0 | 2 (planned for up to 6) |

## 4. Subnetting Design (VLSM)
Subnets were assigned largest-first to minimize address waste,
following VLSM (Variable Length Subnet Masking) principles.

| Department | Hosts Needed | CIDR | Subnet Mask | Network Address | Usable Range | Broadcast |
|---|---|---|---|---|---|---|
| Sales | 30 | /27 | 255.255.255.224 | 192.168.10.0 | .1 – .30 | .31 |
| Admin | 14 | /28 | 255.255.255.240 | 192.168.10.32 | .33 – .46 | .47 |
| IT/Server | 6 | /29 | 255.255.255.248 | 192.168.10.48 | .49 – .54 | .55 |

Each subnet block was assigned sequentially with no gaps or overlaps,
starting where the previous broadcast address ended.

## 5. IP Address Assignments

### Router Interfaces
| Interface | Department | IP Address | Subnet Mask |
|---|---|---|---|
| GigabitEthernet0/0 | Admin | 192.168.10.33 | 255.255.255.240 |
| GigabitEthernet0/1 | Sales | 192.168.10.1 | 255.255.255.224 |
| GigabitEthernet0/2 | IT/Server | 192.168.10.49 | 255.255.255.248 |

### End Devices
| Device | Department | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| PC1 | Admin | 192.168.10.34 | 255.255.255.240 | 192.168.10.33 |
| PC0 | Admin | 192.168.10.35 | 255.255.255.240 | 192.168.10.33 |
| Laptop0 | Admin | 192.168.10.36 | 255.255.255.240 | 192.168.10.33 |
| PC2 | Sales | 192.168.10.2 | 255.255.255.224 | 192.168.10.1 |
| Laptop1 | Sales | 192.168.10.3 | 255.255.255.224 | 192.168.10.1 |
| PC3 | Sales | 192.168.10.4 | 255.255.255.224 | 192.168.10.1 |
| PC6 | IT/Server | 192.168.10.50 | 255.255.255.248 | 192.168.10.49 |
| Server0 | IT/Server | 192.168.10.51 | 255.255.255.248 | 192.168.10.49 |

## 6. Network Topology
```
                    [Admin Switch — Switch0]
                    PC1  PC0  Laptop0
                          |
[IT/Server Switch — Switch3] — [Router1 (2911)] — [Sales Switch — Switch1]
PC6  Server0                                       PC2  Laptop1  PC3
```

The router acts as the central routing device, with one interface
per department subnet. Each switch connects all devices within its
department to the router.

## 7. Router Configuration (Cisco IOS)
```
enable
configure terminal

interface GigabitEthernet0/0
ip address 192.168.10.33 255.255.255.240
no shutdown
exit

interface GigabitEthernet0/1
ip address 192.168.10.1 255.255.255.224
no shutdown
exit

interface GigabitEthernet0/2
ip address 192.168.10.49 255.255.255.248
no shutdown
exit

end
write memory
```

## 8. Connectivity Testing
Inter-department ping tests were performed to verify that the router
was correctly forwarding traffic between subnets.

| Source | Destination | Result |
|---|---|---|
| PC1 (Admin) | PC6 (IT/Server) | ✅ Successful |
| PC0 (Admin) | Laptop1 (Sales) | ✅ Successful |
| Laptop0 (Admin) | PC2 (Sales) | ✅ Successful |

All cross-department pings were successful, confirming that:
- All end devices are correctly configured
- All router interfaces are up and correctly addressed
- The router is successfully forwarding traffic between all three subnets

**Note on first-ping timeout:** The first ICMP packet in each test
timed out due to ARP (Address Resolution Protocol) resolution. This
is expected behavior — the sending device first broadcasts an ARP
request to discover the router's MAC address before it can forward
the ping. Subsequent packets reply successfully once the ARP table
is populated. This is normal and not indicative of a network fault.

## 9. Industry Methodology Alignment
This lab exercise maps directly to real-world network design and
implementation practices:

**VLSM in production networks:** Assigning different subnet sizes
per department is standard practice in enterprise network design.
It conserves address space and allows efficient use of the available
IP block — critical in environments using private addressing with
many departments.

**Router-on-a-stick / inter-VLAN routing:** The design used here
(one physical router interface per subnet) mirrors how smaller
office networks route between departments. Larger networks achieve
the same result using VLANs and Layer 3 switches, but the underlying
routing logic is identical.

**Gateway as first usable address:** Reserving the first usable
address of each subnet for the router interface is a widely followed
convention in network engineering. It makes configuration consistent
and predictable across large networks.

**Connectivity verification with ICMP:** Using ping to verify
end-to-end reachability after configuration is standard practice
in network commissioning. In a real deployment, this step would
be documented in a test plan and signed off before handover.

**ARP behavior:** Understanding that the first ping timeout is ARP
resolution rather than a network fault is an important distinction
in real troubleshooting. Misreading ARP timeouts as connectivity
failures is a common junior mistake that this lab helped clarify.

## 10. Lessons Learned
- VLSM requires subnetting largest-to-smallest to avoid wasting
  address space — starting with the smallest subnet first leads to
  fragmentation and inefficient allocation.
- Router interfaces must be explicitly brought up with `no shutdown`
  in Cisco IOS — they are administratively down by default, which
  is a deliberate security posture.
- Physical link state (green in Packet Tracer) does not guarantee
  Layer 3 connectivity — a link can be physically up but still fail
  to route if the interface IP is not configured correctly.
- ARP resolution adds latency to the first packet on any new path —
  this is not a fault but a normal part of IP networking behavior.
