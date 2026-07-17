# Portfolio Project #2 — Café Network Design

## Overview
This project documents the design and implementation of a café network
using Cisco Packet Tracer. The network separates customer WiFi traffic
from internal business systems (Staff, POS/Till, Back Office, and
Management) using two routers and VLSM subnetting, reflecting
real-world café network security requirements.

## Tools Used
- **Cisco Packet Tracer 9.0** — network simulation
- **Cisco 2911 Router (x2)** — inter-department routing
- **Cisco 2960-24TT Switch (x5)** — department switching
- **WRT300N Wireless Router** — customer WiFi access point

## Network Summary
| Department | Subnet | Gateway | Hosts |
|---|---|---|---|
| Customer WiFi | 10.0.0.0/26 | 10.0.0.1 | Up to 62 |
| Staff Devices | 10.0.0.64/28 | 10.0.0.65 | Up to 14 |
| POS/Till | 10.0.0.80/29 | 10.0.0.81 | Up to 6 |
| Back Office | 10.0.0.88/29 | 10.0.0.89 | Up to 6 |
| Router Link | 10.0.0.100/30 | — | Point-to-point |

## Key Skills Demonstrated
- IPv4 subnetting and VLSM
- Dual-router network design
- Static routing between routers
- Network security zone separation
- Guest WiFi isolation from internal systems
- Cisco IOS interface and routing configuration

## Files
```
project2-cafe-network/
├── README.md
├── reports/
│   └── cafe-network-report.md
└── screenshots/
    ├── topology.png
    ├── ping-results.png
    └── router-config.png
```

## Authorization & Disclaimer
This is a simulated lab environment built entirely within Cisco
Packet Tracer. No real network infrastructure was accessed or modified.
