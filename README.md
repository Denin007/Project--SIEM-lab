# Enterprise Security Lab — 12-VM SIEM & Network Monitoring Environment

A personal cybersecurity home lab built to simulate an enterprise network environment, practice real-world security monitoring workflows, and develop hands-on SOC analyst skills.

---

## 📋 Project Overview

This lab replicates a mid-sized enterprise network topology across 12 virtual machines — including Windows Server with Active Directory, Linux endpoints, a DMZ segment, and a centralised SIEM deployment. The primary goal was to move beyond theoretical knowledge and practice the actual workflows a SOC L1 analyst performs daily: alert triage, log investigation, false positive reduction, and incident documentation.

**Key outcomes:**
- Deployed and configured **Zabbix SIEM** for continuous security monitoring
- Monitored **2,500+ endpoint security events** across the environment
- Reduced false-positive alert rates by **18%** through detection rule tuning
- Traced and documented anomalous activity across Windows and Linux hosts
- Built investigation reports for each simulated security scenario

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    ENTERPRISE LAB                        │
│                                                          │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │ Windows      │    │ Zabbix SIEM  │                   │
│  │ Server 2019  │    │ (Monitoring) │                   │
│  │ (DC / AD)    │    └──────────────┘                   │
│  └──────────────┘                                        │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Win 10   │  │ Win 10   │  │ Win 10   │  (Endpoints) │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Ubuntu   │  │ Kali     │  │ Ubuntu   │  (Linux)     │
│  │ Server   │  │ Linux    │  │ Desktop  │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
│  ┌──────────────────────────────────────┐               │
│  │  pfSense Firewall  │  VLAN Segments  │               │
│  └──────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

**Network Segments:**
- `10.0.1.0/24` — Corporate LAN (Windows endpoints)
- `10.0.2.0/24` — Server segment (DC, SIEM)
- `10.0.3.0/24` — Linux segment
- `10.0.4.0/24` — DMZ (isolated)

---

## 🛠️ Technologies Used

| Category | Tool / Technology |
|---|---|
| SIEM | Zabbix 6.x |
| Network Analysis | Wireshark |
| Reconnaissance | Nmap |
| Directory Services | Windows Server Active Directory |
| Policy Management | Group Policy Objects (GPO) |
| Network Segmentation | VLANs |
| OS Platforms | Windows Server 2019, Windows 10, Ubuntu 22.04, Kali Linux |
| Hypervisor | VirtualBox / VMware Workstation |
| Traffic Capture | Wireshark, tcpdump |

---

## 📁 Repository Structure

```
enterprise-lab-siem/
├── README.md                    # This file
├── docs/
│   ├── lab-setup-guide.md       # Step-by-step environment setup
│   ├── network-topology.md      # Detailed network design decisions
│   ├── zabbix-configuration.md  # SIEM setup and rule configuration
│   └── active-directory.md      # AD structure and GPO policies
├── configs/
│   ├── zabbix-alert-rules/      # Detection rule configurations
│   ├── gpo-policies/            # Group Policy Object exports
│   └── network-config/          # VLAN and firewall rule notes
├── investigations/
│   ├── INV-001-brute-force.md   # Brute force detection investigation
│   ├── INV-002-lateral-movement.md  # Lateral movement trace
│   ├── INV-003-false-positive-analysis.md  # FP reduction methodology
│   └── INV-004-c2-traffic.md    # C2 communication pattern analysis
├── reports/
│   ├── false-positive-reduction-report.md  # 18% FP reduction analysis
│   └── monthly-alert-summary.md
└── screenshots/
    └── README.md                # Screenshot index
```

---

## 🔍 Key Investigations

### INV-001 — Brute Force Detection
Traced repeated failed authentication attempts across Windows endpoints using Zabbix event correlation. Identified the source host, documented the event timeline, and tuned detection rules to reduce noise from legitimate failed logins.
→ [View investigation](investigations/INV-001-brute-force.md)

### INV-002 — Lateral Movement Simulation
Simulated lateral movement between network segments using common techniques. Monitored Wireshark captures and Windows event logs to trace the movement path and identify detection gaps in the SIEM ruleset.
→ [View investigation](investigations/INV-002-lateral-movement.md)

### INV-003 — False Positive Reduction
Analysed 2,500+ SIEM alerts over a monitoring period to identify recurring false positives. Tuned 6 detection rules, reducing false-positive alert volume by 18% while maintaining detection coverage for genuine threats.
→ [View investigation](investigations/INV-003-false-positive-analysis.md)

### INV-004 — C2 Traffic Pattern Analysis
Used Wireshark to capture and analyse simulated C2 communication patterns — identifying beaconing behaviour, unusual outbound connection frequency, and protocol anomalies consistent with command-and-control activity.
→ [View investigation](investigations/INV-004-c2-traffic.md)

---

## 📊 Results & Metrics

| Metric | Result |
|---|---|
| Total VMs deployed | 12 |
| Security events monitored | 2,500+ |
| False positive rate reduction | 18% |
| Detection rules created/tuned | 14 |
| Investigations documented | 4 |
| Attack scenarios simulated | 6 |

---

## 📖 Documentation

- [Lab Setup Guide](docs/lab-setup-guide.md) — How to replicate this environment
- [Network Topology](docs/network-topology.md) — Design decisions and rationale
- [Zabbix Configuration](docs/zabbix-configuration.md) — SIEM setup, agents, and rule creation
- [Active Directory](docs/active-directory.md) — Domain structure and GPO policies

---

## 🎯 Skills Developed

- **SIEM deployment and administration** — configuring agents, defining triggers, managing dashboards
- **Alert triage and investigation** — distinguishing true positives from false positives under realistic alert volumes
- **Detection rule tuning** — reducing noise without sacrificing coverage
- **Network traffic analysis** — reading packet captures for security-relevant patterns
- **Incident documentation** — writing structured investigation reports with evidence, timeline, and conclusions
- **Windows Server administration** — Active Directory, DNS, DHCP, GPO management
- **Linux command line** — log analysis, service management, network diagnostics

---

## 🔗 Related Projects

- [Penetration Testing Assessment](https://github.com/denin-sajan) — MSc security assessment using Nmap, Burp Suite, and Metasploit
- [Automated Threat Detection](https://github.com/denin-sajan) — MSc dissertation, 98.84% malware classification accuracy

---

## 📫 Contact

**Denin Sajan** — Cybersecurity Professional | SOC Analyst in Training

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Denin%20Sajan-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/denin-sajan)
[![GitHub](https://img.shields.io/badge/GitHub-denin--sajan-181717?style=flat&logo=github)](https://github.com/denin-sajan)
[![Email](https://img.shields.io/badge/Email-antodenin%40gmail.com-D14836?style=flat&logo=gmail)](mailto:antodenin@gmail.com)
