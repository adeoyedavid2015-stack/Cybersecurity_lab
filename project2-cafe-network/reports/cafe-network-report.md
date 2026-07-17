# Network Design Report — Café Network

## 1. Objective
Design and implement a café network that separates customer WiFi
traffic from internal business systems using proper IPv4 subnetting
(VLSM) and dual-router architecture. The design reflects real-world
café network security requirements where customer devices must be
isolated from POS/till systems and back-office infrastructure.

## 2. Environment
- **Simulation Tool:** Cisco Packet Tracer 9.0
- **Routers:** Cisco 2911 (x2)
- **Switches:** Cisco 2960-24TT (x5)
- **Wireless:** WRT300N Wireless Router (Customer WiFi)
- **End Devices:** PCs, Laptops, Server
- **Base Address Block:** 10.0.0.0/24

## 3. Network Requirements
| Department | Devices | Hosts Required |
|---|---|---|
| Customer WiFi | PC0, Laptop0, WRT300N | 62 (planned for growth) |
| Staff Devices | PC1, Laptop1 | 14 (planned for growth) |
| POS/Till | PC2, Laptop2 | 6 (planned for growth) |
| Back Office | PC3, Laptop3 | 6 (planned for growth) |
| Router Link | Router0 ↔ Router1 | 2 (point-to-point) |

## 4. Subnetting Design (VLSM)
Subnets were assigned largest-first following VLSM principles to
minimize address waste.

| Department | Hosts Needed | CIDR | Subnet Mask | Network Address | Usable Range | Broadcast |
|---|---|---|---|---|---|---|
| Customer WiFi | 62 | /26 | 255.255.255.192 | 10.0.0.0 | .1 – .62 | .63 |
| Staff Devices | 14 | /28 | 255.255.255.240 | 10.0.0.64 | .65 – .78 | .79 |
| POS/Till | 6 | /29 | 255.255.255.248 | 10.0.0.80 | .81 – .86 | .87 |
| Back Office | 6 | /29 | 255.255.255.248 | 10.0.0.88 | .89 – .94 | .95 |
| Router Link | 2 | /30 | 255.255.255.252 | 10.0.0.100 | .101 – .102 | .103 |

## 5. Dual-Router Architecture
Unlike the single-router office network design, the café network uses
two routers connected via a point-to-point link. This reflects a
common real-world pattern where:

- **Router0** handles customer-facing and staff networks
- **Router1** handles sensitive business systems (POS, Back Office)
- A dedicated **router link subnet** connects the two routers

This separation means that even if the customer WiFi segment were
compromised, an attacker would still need to cross the router boundary
to reach the POS or Back Office systems — adding a layer of security.

### Router Assignment
| Router | Department | Interface | IP Address |
|---|---|---|---|
| Router0 | Customer WiFi | Gi0/0 | 10.0.0.1 |
| Router0 | Staff Devices | Gi0/1 | 10.0.0.65 |
| Router0 | Router Link | Gi0/2 | 10.0.0.101 |
| Router1 | Router Link | Gi0/0 | 10.0.0.102 |
| Router1 | POS/Till | Gi0/1 | 10.0.0.81 |
| Router1 | Back Office | Gi0/2 | 10.0.0.89 |

## 6. IP Address Assignments

### End Devices
| Device | Department | IP Address | Subnet Mask | Gateway |
|---|---|---|---|---|
| PC0 | Customer | 10.0.0.2 | 255.255.255.192 | 10.0.0.1 |
| Laptop0 | Customer | 10.0.0.3 | 255.255.255.192 | 10.0.0.1 |
| WRT300N | Customer WiFi | 10.0.0.4 | 255.255.255.192 | 10.0.0.1 |
| PC1 | Staff | 10.0.0.66 | 255.255.255.240 | 10.0.0.65 |
| Laptop1 | Staff | 10.0.0.67 | 255.255.255.240 | 10.0.0.65 |
| PC2 | POS/Till | 10.0.0.82 | 255.255.255.248 | 10.0.0.81 |
| Laptop2 | POS/Till | 10.0.0.83 | 255.255.255.248 | 10.0.0.81 |
| PC3 | Back Office | 10.0.0.90 | 255.255.255.248 | 10.0.0.89 |
| Laptop3 | Back Office | 10.0.0.91 | 255.255.255.248 | 10.0.0.89 |

## 7. Router Configuration (Cisco IOS)

### Router0
```
enable
configure terminal

interface GigabitEthernet0/0
ip address 10.0.0.1 255.255.255.192
no shutdown
exit

interface GigabitEthernet0/1
ip address 10.0.0.65 255.255.255.240
no shutdown
exit

interface GigabitEthernet0/2
ip address 10.0.0.101 255.255.255.252
no shutdown
exit

ip route 10.0.0.80 255.255.255.248 10.0.0.102
ip route 10.0.0.88 255.255.255.248 10.0.0.102

end
write memory
```

### Router1
```
enable
configure terminal

interface GigabitEthernet0/0
ip address 10.0.0.102 255.255.255.252
no shutdown
exit

interface GigabitEthernet0/1
ip address 10.0.0.81 255.255.255.248
no shutdown
exit

interface GigabitEthernet0/2
ip address 10.0.0.89 255.255.255.248
no shutdown
exit

ip route 10.0.0.0 255.255.255.192 10.0.0.101
ip route 10.0.0.64 255.255.255.240 10.0.0.101

end
write memory
```

## 8. Static Routing
Since the two routers are separate devices, each router needs to be
manually told about the networks connected to the other router.
This is achieved using static routes.

| Router | Destination Network | Next Hop | Meaning |
|---|---|---|---|
| Router0 | 10.0.0.80/29 | 10.0.0.102 | To reach POS, send to Router1 |
| Router0 | 10.0.0.88/29 | 10.0.0.102 | To reach Back Office, send to Router1 |
| Router1 | 10.0.0.0/26 | 10.0.0.101 | To reach Customer, send to Router0 |
| Router1 | 10.0.0.64/28 | 10.0.0.101 | To reach Staff, send to Router0 |

## 9. Connectivity Testing
| Source | Destination | Department Cross | Result |
|---|---|---|---|
| PC0 (Customer) | 10.0.0.3 (Customer) | Same subnet | ✅ Successful |
| PC0 (Customer) | 10.0.0.81 (POS) | Router0 → Router1 | ✅ Successful* |
| PC3 (Back Office) | Laptop0 (Customer) | Router1 → Router0 | ✅ Successful |
| Laptop3 (Back Office) | PC1 (Staff) | Router1 → Router0 | ✅ Successful |

*First ping timed out due to ARP resolution — subsequent packets
replied successfully. This is normal expected behavior.

## 10. Industry Methodology Alignment

**Guest network isolation is a real security requirement**
In real café and hospitality environments, isolating customer WiFi
from internal business systems is not optional — it's a baseline
security requirement. POS systems handle payment card data and are
subject to PCI-DSS compliance requirements, which explicitly mandate
network segmentation between cardholder data systems and untrusted
networks (like customer WiFi).

**Dual-router design mirrors real perimeter architecture**
Using two routers with a dedicated link between them mirrors how
real networks implement security zones — a DMZ (demilitarized zone)
architecture uses exactly this principle: traffic must cross a router
boundary to move between security zones, giving security teams a
chokepoint to monitor and filter traffic.

**Static routing vs dynamic routing**
Static routing was used here because the network is small and
predictable. In larger networks with many routers, dynamic routing
protocols (OSPF, EIGRP) would be used instead — they automatically
share routing information between routers without manual
configuration, which scales far better than maintaining static
routes manually.

**Point-to-point /30 links between routers**
Using a /30 subnet for the router-to-router link is standard
practice in real networks — it wastes the minimum possible address
space (only 4 addresses) for a link that only ever needs 2 usable
IPs. Every real WAN link or router interconnect uses /30 or /31
for exactly this reason.

**ARP behavior on first ping**
The first ping timing out due to ARP resolution is expected behavior
in real networks. ARP (Address Resolution Protocol) resolves IP
addresses to MAC addresses before packets can be forwarded. In
production environments, this latency is managed using ARP caching
and pre-population techniques, but the underlying behavior is the
same.

## 11. Lessons Learned
- A 2911 router has only 3 GigabitEthernet ports by default —
  networks with more than 3 departments need either additional
  modules, multiple routers, or a Layer 3 switch.
- Static routes must use the **network address** of the destination
  subnet, not the gateway or host address — the route covers the
  entire subnet, not one specific device.
- The HWIC-4ESW module adds switch ports, not routed ports — these
  cannot be assigned IP addresses directly without conversion using
  the `no switchport` command, which itself has limitations depending
  on the module type.
- Point-to-point router links need their own dedicated subnet (/30)
  — the two usable IPs become the gateway addresses for each router's
  connected interface.
- Network security zone design (separating customer traffic from
  business-critical systems) is a fundamental real-world requirement,
  not just a lab exercise concept.
