# Zabbix SIEM Configuration

This document covers the Zabbix SIEM setup decisions, agent configuration, and detection rule structure used in the enterprise lab environment.

---

## Architecture Decision

Zabbix was chosen over other SIEM options for this lab because:
1. Free and open source — no licensing cost for a personal lab
2. Supports agent-based monitoring across both Windows and Linux
3. Flexible trigger/alert system that mirrors enterprise SIEM rule logic
4. Web-based dashboard accessible from any host on the network

---

## Server Specifications

| Setting | Value |
|---|---|
| OS | Ubuntu Server 22.04 LTS |
| IP Address | 10.0.2.20 |
| Hostname | LAB-SIEM01 |
| Zabbix Version | 6.0 LTS |
| Database | MySQL 8.0 |
| RAM allocated | 4GB |
| Storage | 40GB |

---

## Monitored Hosts

| Hostname | IP | OS | Agent Version |
|---|---|---|---|
| LAB-DC01 | 10.0.2.10 | Windows Server 2019 | Zabbix Agent 6.0 |
| LAB-WEB01 | 10.0.2.15 | Ubuntu 22.04 | Zabbix Agent 6.0 |
| WIN10-WS01 | 10.0.1.11 | Windows 10 | Zabbix Agent 6.0 |
| WIN10-WS02 | 10.0.1.12 | Windows 10 | Zabbix Agent 6.0 |
| WIN10-WS03 | 10.0.1.15 | Windows 10 | Zabbix Agent 6.0 |
| UBUNTU-WS01 | 10.0.3.11 | Ubuntu 22.04 | Zabbix Agent 6.0 |
| KALI-ATTACK | 10.0.3.20 | Kali Linux | No agent (attacker) |

---

## Detection Rules Created

### Rule 1 — Failed Authentication Spike
```
Name: Failed Auth Spike
Trigger: count(Windows Security Event 4625) > 10 in 5 minutes from single source IP
Severity: High
Action: Alert + email notification
```

### Rule 2 — Account Lockout
```
Name: Account Lockout Detected
Trigger: Windows Security Event 4740 detected
Severity: Medium
Action: Alert
```

### Rule 3 — New Service Installation
```
Name: New Service Installed
Trigger: Windows Security Event 7045 detected
Severity: Medium
Action: Alert (suppressed during maintenance windows)
```

### Rule 4 — Privileged Account Usage
```
Name: Special Privilege Assigned
Trigger: Windows Security Event 4672 from non-admin account
Severity: High
Action: Alert
```

### Rule 5 — Outbound to New Destination
```
Name: New Outbound Connection
Trigger: New destination IP not in whitelist, sustained for >60 seconds
Severity: Low (first occurrence), Medium (repeated)
Action: Alert after 24-hour learning period
```

### Rule 6 — Large Data Transfer
```
Name: Large Outbound Transfer
Trigger: >500MB transferred to external-facing interface in 1 hour
Severity: Medium
Action: Alert
```

---

## Alert Tuning Log

| Date | Rule | Change Made | Reason |
|---|---|---|---|
| Week 1 | Failed Auth | Raised threshold from 5 to 10 attempts | Legitimate password mistakes triggering alert |
| Week 1 | Outbound IP | Added Microsoft/Ubuntu IP whitelist | Windows Update causing constant alerts |
| Week 2 | Service Install | Added maintenance window suppression | Admin tasks triggering during setup |
| Week 2 | Large Transfer | Scoped to external interfaces only | VM snapshots triggering alert |
| Week 3 | Outbound IP | Added 24-hour learning period | New lab software triggering on first run |
| Week 3 | Privilege Use | Whitelisted known admin accounts | Routine admin tasks triggering |

**Result: 18% reduction in overall alert volume, 47% reduction in confirmed false positives**
