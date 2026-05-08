# INV-001 — Brute Force Detection Investigation

**Date:** Lab simulation  
**Severity:** Medium  
**Status:** Resolved  
**Analyst:** Denin Sajan

---

## Summary

During routine SIEM monitoring, Zabbix triggered an alert for a spike in failed authentication attempts originating from a single host within the corporate LAN segment. Investigation confirmed this was a simulated brute force attack against the Windows Domain Controller. The attack was traced, documented, and used to tune detection thresholds.

---

## Alert Details

| Field | Value |
|---|---|
| Alert Name | Failed Authentication Spike |
| Trigger Threshold | >5 failed logins in 10 minutes |
| Source Host | `10.0.1.15` (WIN10-WS03) |
| Target Host | `10.0.2.10` (LAB-DC01) |
| Event IDs Observed | 4625 (Failed Logon), 4740 (Account Lockout) |
| Alert Time | Simulated scenario |
| Duration | 4 minutes |

---

## Investigation Steps

### Step 1 — Alert Triage
Zabbix dashboard showed repeated triggering of the "Failed Authentication Spike" rule. Initial triage confirmed the alert was not a false positive — the volume and pattern were inconsistent with normal user behaviour.

### Step 2 — Log Collection
Pulled Windows Security Event logs from `LAB-DC01` using Zabbix agent log forwarding. Filtered for Event ID 4625 within the alert timeframe.

**Key log entries observed:**
```
Event ID: 4625
Account Name: Administrator
Failure Reason: Unknown username or bad password
Source Network Address: 10.0.1.15
Logon Type: 3 (Network)
Repeated: 47 times in 4 minutes
```

### Step 3 — Source Host Investigation
Pivoted to investigate `WIN10-WS03` (`10.0.1.15`):
- Reviewed running processes — identified Hydra (simulated attacker tool) running under a test user account
- Confirmed this was the originating host of the brute force activity
- No lateral movement observed — attack contained to authentication attempts only

### Step 4 — Timeline Reconstruction

| Time | Event |
|---|---|
| T+00:00 | First failed login attempt from 10.0.1.15 |
| T+00:47 | Zabbix alert triggered (threshold exceeded) |
| T+01:30 | Account lockout triggered for Administrator (GPO) |
| T+04:00 | Attack activity ceased |
| T+04:30 | Investigation initiated |

### Step 5 — Containment (Simulated)
In a real environment, next steps would be:
1. Isolate `WIN10-WS03` from the network
2. Escalate to L2 analyst with full timeline and log evidence
3. Initiate password reset for targeted accounts
4. Review other accounts for compromise

---

## Root Cause

Simulated brute force attack using Hydra from an internal endpoint against the Domain Controller. The attack was possible because:
1. No rate limiting on RDP/SMB authentication attempts beyond the GPO lockout threshold
2. Common Administrator account targeted — should be renamed or disabled

---

## Detection Rule Assessment

The existing detection rule (>5 failed logins in 10 minutes) successfully triggered the alert. However, initial testing revealed the threshold was generating false positives from legitimate users who forget their passwords.

**Rule tuned to:** >10 failed logins in 5 minutes from a single source IP — reduced FP rate while maintaining detection sensitivity.

---

## Lessons Learned

1. Account lockout GPO (lock after 5 attempts) worked correctly and limited the attack scope
2. Renaming or disabling the built-in Administrator account would reduce brute force targeting
3. Network segmentation prevented the attack from reaching servers in other VLANs
4. Zabbix alert timing (47 seconds after threshold) is acceptable for this attack type

---

## Evidence

- Zabbix alert screenshot: `screenshots/INV-001-zabbix-alert.png`
- Windows Event Log export: `evidence/INV-001-event-logs.evtx`
- Wireshark capture of authentication traffic: `evidence/INV-001-traffic.pcap`
