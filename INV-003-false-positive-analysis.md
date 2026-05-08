# INV-003 — False Positive Reduction Analysis

**Date:** Lab simulation  
**Type:** Detection Rule Optimisation  
**Analyst:** Denin Sajan

---

## Objective

After initial Zabbix SIEM deployment across the 12-VM lab environment, alert volume was too high to be operationally useful — many alerts were triggering on normal lab activity. This investigation documents the methodology used to identify, categorise, and reduce false positive alerts by 18% while maintaining detection coverage for genuine threats.

---

## Initial Alert Baseline

Over the first monitoring period, 2,500+ security events were generated. Of these, the following alert categories were reviewed:

| Alert Category | Total Alerts | Confirmed FP | FP Rate |
|---|---|---|---|
| Failed Authentication | 847 | 312 | 36.8% |
| New Network Connection | 634 | 498 | 78.5% |
| Service Installation | 203 | 89 | 43.8% |
| Privilege Escalation | 156 | 22 | 14.1% |
| Large File Transfer | 98 | 61 | 62.2% |
| Outbound to New IP | 312 | 287 | 92.0% |

**Overall false positive rate: ~43% before tuning**

---

## False Positive Categories Identified

### Category 1 — Legitimate Failed Logins
**Problem:** The "5 failed logins in 10 minutes" rule triggered frequently from users who mistyped passwords or locked themselves out legitimately.

**Resolution:** Raised threshold to 10 failed attempts in 5 minutes from a single source IP. Added exception for known service accounts.

**Impact:** Reduced false positives in this category by 31%

---

### Category 2 — Normal Outbound Connections
**Problem:** The "outbound to new IP" rule triggered on every Windows Update check, DNS lookup, and legitimate software call home.

**Resolution:** Built a whitelist of known-good IP ranges (Microsoft, Ubuntu update servers, NTP servers). Added 24-hour learning period before alerting on new outbound destinations.

**Impact:** Reduced false positives in this category by 67%

---

### Category 3 — Lab Maintenance Activity
**Problem:** Installing Zabbix agents, configuring GPOs, and routine admin tasks triggered service installation and privilege escalation alerts.

**Resolution:** Created a maintenance window suppression rule — alerts in these categories suppressed during defined maintenance periods. Added whitelisting for known admin accounts performing expected privileged actions.

**Impact:** Reduced false positives in this category by 28%

---

### Category 4 — Large File Transfers
**Problem:** VM snapshots and backup operations triggered large file transfer alerts constantly.

**Resolution:** Scoped the rule to flag transfers to external-facing interfaces only, excluding internal VM management traffic.

**Impact:** Reduced false positives in this category by 74%

---

## Tuning Results

After implementing the above changes across 6 detection rules:

| Metric | Before Tuning | After Tuning | Change |
|---|---|---|---|
| Daily alert volume | ~180 alerts/day | ~148 alerts/day | -18% |
| Confirmed false positives | ~78 per day | ~41 per day | -47% |
| True positive rate | 57% | 72% | +15% |
| Mean time to investigate | ~12 min/alert | ~8 min/alert | -33% |

**Overall false positive rate reduction: 18%**

---

## Methodology Notes

The key principle applied throughout this process: **never tune a rule without understanding why it fired.** Every false positive was investigated before being dismissed — the risk of suppressing a genuine alert because it resembled a false positive is higher than the cost of investigating a few extra alerts.

Each rule change was documented with:
1. The original rule definition
2. The specific false positive pattern that triggered the review
3. The change made and the rationale
4. Validation testing to confirm true positives were still detected after the change

---

## Lessons Learned

1. Baseline your environment before writing detection rules — what looks like an anomaly is often normal activity you haven't seen before
2. Whitelisting is more sustainable than raising thresholds — it keeps sensitivity high while eliminating known noise
3. Every tuning decision should be documented — undocumented rule changes create blind spots
4. False positive reduction and detection coverage are not opposites — careful tuning improves both
