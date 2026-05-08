# Lab Setup Guide

This document covers the steps taken to build the 12-VM enterprise security lab environment from scratch.

---

## Prerequisites

- Hypervisor: VirtualBox (free) or VMware Workstation
- Minimum host machine specs: 16GB RAM, 250GB free storage, quad-core CPU
- ISO files required:
  - Windows Server 2019 Evaluation (free from Microsoft)
  - Windows 10 (evaluation copy)
  - Ubuntu Server 22.04 LTS (free)
  - Kali Linux (free)

---

## Phase 1 — Network Design

Before spinning up any VMs, the network topology was planned on paper first.

**Decision:** Use four VLAN segments to simulate real network segmentation:
- Corporate LAN: `10.0.1.0/24`
- Server segment: `10.0.2.0/24`
- Linux segment: `10.0.3.0/24`
- DMZ: `10.0.4.0/24`

**Rationale:** Flat networks don't reflect enterprise environments and don't generate the inter-segment traffic needed to practice monitoring lateral movement or traffic anomalies.

In VirtualBox, this was achieved using Internal Networks with host-only adapters per segment.

---

## Phase 2 — Windows Server & Active Directory

### Install Windows Server 2019
1. Create a new VM — 4GB RAM, 60GB disk, 2 vCPUs
2. Install Windows Server 2019 (Desktop Experience)
3. Set a static IP: `10.0.2.10`
4. Rename the server: `LAB-DC01`

### Promote to Domain Controller
```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.local"
```

### Configure DNS
- DNS server automatically configured during AD DS promotion
- Added forward lookup zone for `lab.local`
- Verified resolution from Windows endpoints

### Create Organisational Units
```
lab.local
├── Computers
│   ├── Workstations
│   └── Servers
├── Users
│   ├── IT
│   ├── Finance
│   └── HR
└── Groups
```

---

## Phase 3 — Group Policy Objects

Key GPOs configured to generate realistic security events:

| GPO Name | Purpose | Applied To |
|---|---|---|
| Audit-Policy | Enable logon/logoff, process, and object access auditing | All computers |
| Password-Policy | Minimum 8 chars, complexity required | Domain |
| Account-Lockout | Lock after 5 failed attempts | Domain |
| Restrict-USB | Block removable media | Workstations |
| Disable-Guest | Disable guest account | All computers |

These GPO settings were essential for generating the Windows Security Event logs that Zabbix would later monitor.

---

## Phase 4 — Zabbix SIEM Deployment

### Install Zabbix Server (Ubuntu 22.04)
```bash
# Add Zabbix repository
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
apt update

# Install Zabbix server, frontend, agent
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Configure database
mysql -uroot -p
> create database zabbix character set utf8mb4 collate utf8mb4_bin;
> create user zabbix@localhost identified by 'your_password';
> grant all privileges on zabbix.* to zabbix@localhost;
```

### Deploy Zabbix Agents
Agents were installed on all Windows and Linux hosts to forward logs and performance metrics to the Zabbix server.

**Windows agent installation:**
- Downloaded Zabbix agent MSI from zabbix.com
- Configured `zabbix_agentd.conf` with server IP `10.0.2.20`
- Started Zabbix Agent service

**Linux agent:**
```bash
apt install zabbix-agent
nano /etc/zabbix/zabbix_agentd.conf
# Set: Server=10.0.2.20
systemctl restart zabbix-agent
```

---

## Phase 5 — Detection Rules & Alert Configuration

Initial detection rules created in Zabbix:

1. **Failed authentication spike** — trigger if >5 failed logins in 10 minutes on any host
2. **New service installation** — alert on Windows Event ID 7045
3. **Privileged account usage** — monitor Event ID 4672 (special privileges assigned)
4. **Outbound connection to new IP** — baseline normal outbound destinations, alert on new ones
5. **Large file transfer** — alert on unusual data volume leaving the network

Each rule required tuning after initial deployment to reduce false positives from normal lab activity — see [INV-003 False Positive Analysis](../investigations/INV-003-false-positive-analysis.md).

---

## Phase 6 — Attack Simulation

Once the environment was stable, attack scenarios were simulated from the Kali Linux VM:

- Network scanning (Nmap) — to verify detection of reconnaissance activity
- Brute force attempts — to test account lockout GPO and authentication alert rules
- Lateral movement simulation — to test inter-VLAN detection coverage
- C2 beaconing simulation — to test outbound connection anomaly detection

All simulations were conducted in the isolated lab environment with no external network access.
