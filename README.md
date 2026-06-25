# Nmap Scan Report — Basic Service & OS Detection Scan

## 1. Objective
Identify open ports, running services, and operating system fingerprint
of a lab target using Nmap's SYN scan, service version detection, and
OS detection features, as practice for reconnaissance and host
enumeration.

## 2. Environment & Authorization
- **Target:** `192.168.91.1` (VMware virtual network gateway / host
  machine, accessed via a VMware host-only/NAT adapter)
- **Scanner host:** Kali Linux (Nmap 7.95)
- **Authorization:** This is a self-owned home lab environment. The
  target is the VMware virtual adapter on my own machine — no
  third-party systems were scanned.

## 3. Command Used
```
sudo nmap -sS -sV -O 192.168.91.1
```
| Flag | Purpose |
|------|---------|
| `-sS` | TCP SYN ("stealth") scan — half-open scan, faster and less likely to be logged by basic services than a full connect scan |
| `-sV` | Probes open ports to determine service/version |
| `-O`  | Attempts OS fingerprinting based on TCP/IP stack behavior |
| `sudo` | Required for `-sS` and `-O`, since both need raw socket access |

## 4. Raw Output
See [`scans/basic_scan.txt`](../scans/basic_scan.txt) for the full
terminal output.

## 5. Findings

| Port | State | Service | Version |
|------|-------|---------|---------|
| 2869/tcp | open | http | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |
| 5357/tcp | open | http | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |

**Other observations:**
- **998 ports filtered** (no response) — most ports are not reachable,
  suggesting a firewall is silently dropping probes rather than
  actively refusing connections.
- **MAC Address:** `00:50:56:C0:00:08` — identified as a VMware vendor
  prefix, confirming this is a virtual network adapter rather than
  physical hardware.
- **OS detection:** Nmap guessed Windows 11/10 (92% confidence) or
  FreeBSD 6.x (88% confidence), but explicitly flagged the result as
  unreliable since no closed TCP port was found to use as a
  fingerprinting baseline.
- **Network distance:** 1 hop — confirms the target is on the same
  local/virtual network segment as the scanning machine.

## 6. Analysis
- Ports 2869 and 5357 are associated with **Windows' built-in
  SSDP/UPnP HTTP API** (used for device discovery and WS-Management).
  These are commonly open by default on Windows hosts and are not
  unusual to find on a Windows-based VMware host.
- The **lack of any closed port** is the key limiting factor in this
  scan. Nmap's OS detection relies on comparing responses from at
  least one open and one closed port — without a closed port to
  compare against, the OS guess is a low-confidence estimate rather
  than a confirmed fingerprint.
- The high number of filtered ports indicates a firewall (likely
  Windows Defender Firewall, since this is a VMware host adapter on a
  Windows host) is configured to drop unsolicited packets rather than
  reject them, which is a sensible default security posture.
- No exploitable service versions were identified — the only open
  services are Microsoft's internal device-discovery HTTP API, which
  is not designed to be internet/network exposed but is expected on a
  default Windows configuration.

## 7. Recommendations
- If this were a production host (it is not, in this case): the
  SSDP/UPnP service (ports 2869/5357) should be disabled if not
  required, as UPnP services have a history of vulnerabilities and are
  rarely needed outside of home media/device discovery use cases.
- To get a more reliable OS fingerprint in future lab exercises,
  ensure the target has at least one closed (not filtered) TCP port
  reachable, or supply a known open + closed port combination with
  `--osscan-guess` or a more targeted port range.
- Consider following up with a `-sU` (UDP scan) since SSDP/UPnP
  primarily operates over UDP (port 1900) — the TCP-only scan here may
  be missing the more relevant discovery service.

## 8. Lessons Learned
- Reinforced understanding of the difference between a SYN scan
  (`-sS`) and full connect scan, and why `sudo` is required for raw
  socket operations.
- Learned that **OS fingerprinting accuracy depends heavily on having
  both open and closed ports** to compare against — an important
  caveat I hadn't fully appreciated before running this scan myself.
- Confirmed that the MAC OUI prefix (`00:50:56`) is a useful quick
  indicator of virtualized environments (VMware), independent of
  Nmap's own OS guess.
