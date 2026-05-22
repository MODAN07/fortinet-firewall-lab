# Enterprise Firewall Lab — FortiGate & pfSense 🔥

> Hands-on enterprise firewall deployment lab running on VMware ESXi. Covers FortiGate zone-based policies, IPS, SSL-VPN, and pfSense high-availability failover configuration — mirroring real-world enterprise perimeter security.

![FortiGate](https://img.shields.io/badge/FortiGate-EE3124?style=flat)
![pfSense](https://img.shields.io/badge/pfSense-212121?style=flat)
![VMware](https://img.shields.io/badge/VMware_ESXi-607078?style=flat&logo=vmware)

---

## 📌 Lab Overview

This home lab replicates enterprise perimeter security configurations using virtualised firewall appliances on VMware ESXi. Two parallel labs run simultaneously:

1. **FortiGate Lab** — Full UTM configuration including IPS, application control, web filtering, SSL-VPN
2. **pfSense HA Lab** — High-availability pair with CARP failover, pfBlockerNG, and Suricata IDS

---

## 🏗️ Lab Topology

```
                         Internet (Simulated WAN)
                               │
              ┌────────────────┴─────────────────┐
              │                                   │
     ┌────────▼────────┐              ┌───────────▼───────────┐
     │   FortiGate VM  │              │   pfSense Primary      │
     │  (Eval License) │              │                        │
     │                 │              │   pfSense Secondary    │
     │  LAN: 10.10.0.0 │              │   (CARP HA Failover)  │
     │  DMZ: 10.20.0.0 │              │                        │
     │  VPN: 10.30.0.0 │              │   LAN: 192.168.1.0     │
     └────────┬────────┘              └───────────┬────────────┘
              │                                   │
    ┌─────────┼────────────────┐      ┌───────────┼──────────────┐
    │         │                │      │           │              │
  ┌─▼──┐  ┌──▼──┐  ┌────┐  ┌─▼─┐  ┌─▼──┐    ┌──▼──┐        ┌──▼──┐
  │LAN │  │ DMZ │  │VPN │  │Mgmt│  │LAN │    │ DMZ │        │ IDS │
  │VMs │  │ Web │  │Clnt│  │Seg │  │VMs │    │ Web │        │ Tap │
  └────┘  └─────┘  └────┘  └───┘  └────┘    └─────┘        └─────┘
```

---

## ⚙️ FortiGate Configuration Highlights

### Zone-Based Security Policy

```
Zones Configured:
- WAN     → Untrusted (internet)
- LAN     → Trusted (internal users)
- DMZ     → Semi-trusted (web servers, public-facing)
- VPN     → Remote access users
- MGMT    → Firewall/device management only

Policy Matrix:
- LAN → WAN:  Allow (with web filter + app control)
- LAN → DMZ:  Allow specific services only
- WAN → DMZ:  Allow HTTP/HTTPS only (web server)
- VPN → LAN:  Allow post-authentication
- * → MGMT:   Deny all (management plane isolation)
```

### IPS (Intrusion Prevention System)
- Enabled FortiGuard IPS signature set — Extended DB
- Tuned profiles per zone: aggressive for WAN-facing, monitor-only for LAN
- Custom IPS signature created for internal policy enforcement

### SSL-VPN Configuration
- Split tunnelling enabled — only RFC1918 routes pushed to clients
- Certificate-based auth + LDAP group membership validation
- MFA via FortiToken enforced for all VPN users
- IP pool: `10.30.0.0/24` with per-user IP binding

### Web Filtering
- FortiGuard category-based filtering on LAN → WAN
- Custom URL blocklist for known C2 domains (from threat feeds)
- HTTPS inspection enabled (deep SSL inspection for high-risk categories)

---

## ⚙️ pfSense HA Configuration Highlights

### CARP High Availability
```
Primary:   192.168.1.1  (active)
Secondary: 192.168.1.2  (standby)
CARP VIP:  192.168.1.254 (shared — clients point here)

Failover tested:
- Manual: Primary shutdown → Secondary promotes in <3 seconds
- State sync: Active connections survive failover (pfsync enabled)
```

### Suricata IDS Rules
- ET Open ruleset enabled on WAN interface
- Custom rules written for lab environment:
  - Detect port scans from external hosts
  - Alert on DNS queries to known malicious domains
  - Flag unusual outbound connection volumes

### pfBlockerNG
- IP reputation blocking — 15+ threat feed lists subscribed
- DNSBL (DNS blackhole) blocking ad/tracker domains
- GeoIP blocking — restricted inbound to specific regions

---

## 🔬 Lab Exercises Completed

| Exercise | Firewall | Outcome |
|---|---|---|
| Zone segmentation design | FortiGate | Isolated DMZ web server from LAN traffic |
| IPS signature tuning | FortiGate | Reduced false positives from 120/day to 8/day |
| SSL-VPN remote access setup | FortiGate | Split tunnel VPN with MFA working |
| CARP HA failover | pfSense | Sub-3-second failover, stateful connection preserved |
| Suricata rule authoring | pfSense | Custom rule detected simulated C2 beacon |
| Deep SSL inspection | FortiGate | Detected malware in HTTPS stream (test file) |
| Policy migration simulation | Both | Migrated 50-rule FortiGate policy to pfSense format |

---

## 📚 Key Takeaways

- FortiGate's GUI policy editor is intuitive but can mask complexity — always review the CLI output to confirm what was actually written
- CARP HA requires identical interface names and careful VLAN tagging — mismatched configs cause split-brain
- Deep SSL inspection breaks several legitimate apps (certificate pinning) — maintain an exemption list
- pfBlockerNG DNSBL significantly improves network hygiene with minimal performance impact
- IPS requires continuous tuning — a set-and-forget approach generates alert fatigue

---

## 🖥️ Lab Infrastructure

| Component | Spec |
|---|---|
| Hypervisor | VMware ESXi (home lab) |
| FortiGate | FortiGate-VM Eval (v7.x) |
| pfSense | pfSense CE 2.7.x |
| Test VMs | Ubuntu 22.04 + Windows Server 2022 |
| Network | Virtual switches (vSwitch) per zone |

---

## 👤 Author

**Moses Daniel** — Cloud Security & DevSecOps Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin)](https://www.linkedin.com/in/moses-daniel-a8a80861/)
