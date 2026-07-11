# Portfolio Project #1 — Small Office Network Design

## Overview
This project documents the design and implementation of a small office
network using Cisco Packet Tracer. The network consists of three
departments (Admin, Sales, and IT/Server) connected through a central
router, with each department assigned its own subnet using Variable
Length Subnet Masking (VLSM).

## Tools Used
- **Cisco Packet Tracer 9.0** — network simulation
- **Nmap** — supplementary host discovery (see Nmap lab)
- **Kali Linux** — scanning and verification environment

## Network Summary
| Department | Subnet | Gateway | Hosts |
|---|---|---|---|
| Sales | 192.168.10.0/27 | 192.168.10.1 | Up to 30 |
| Admin | 192.168.10.32/28 | 192.168.10.33 | Up to 14 |
| IT/Server | 192.168.10.48/29 | 192.168.10.49 | Up to 6 |

## Files
```
project1-small-office-network/
├── README.md
├── reports/
│   └── network-design-report.md   ← full design documentation
└── screenshots/
    ├── topology.png                ← full network topology
    ├── ping-results.png            ← successful cross-department ping
    └── router-config.png           ← router interface configuration
```

## Key Skills Demonstrated
- IPv4 subnetting and VLSM
- Network topology design
- Cisco IOS router interface configuration
- Inter-VLAN routing concepts
- Connectivity verification using ICMP (ping)

## Authorization & Disclaimer
This is a simulated lab environment built entirely within Cisco Packet
Tracer. No real network infrastructure was accessed or modified.
