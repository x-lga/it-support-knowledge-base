# SEC-002 - Insider Threat: Behavioural and Technical Indicators

**Article ID:** SEC-002
**Category:** Security - Insider Threat Detection
**Severity:** P2 (investigation required) | P1 (active exfiltration suspected)
**Cert alignment:** CompTIA Security+
**Last verified:** 2026-07

---

## Why This Article Exists in an L1 Knowledge Base

Insider threats are rarely detected by automated alerts. They are detected when
an L1 engineer notices something that does not fit a pattern - a user accessing
files they have never accessed before, an account generating unusual outbound
traffic, or a departing employee making copies of data. Without knowing what to
look for, these signals go unrecorded and uninvestigated.

This article defines the indicators an L1 engineer might encounter and describes
exactly what to do when one is observed. The action is always to document and
escalate - never to confront the employee or take independent action.

---

## Why This Article Exists in an L1 Knowledge Base

Insider threats are rarely detected by automated alerts. They are detected when
an L1 engineer notices something that does not fit a pattern — a user accessing
files they have never accessed before, an account generating unusual outbound
traffic, or a departing employee making copies of data. Without knowing what to
look for, these signals go unrecorded and uninvestigated.

This article defines the indicators an L1 engineer might encounter and describes
exactly what to do when one is observed. The action is always to document and
escalate — never to confront the employee or take independent action.

---

## Technical Indicators That May Surface During L1 Support

**File access anomalies:**
```kql
// KQL — Log Analytics: User accessing files they have not accessed before
// (Requires Azure File auditing or Defender for Identity)
CloudAppEvents
| where TimeGenerated > ago(24h)
| where AccountDisplayName == "jsmith@contoso.com"
| where ActionType in ("FileDownloaded", "FileCopied", "FileAccessed")
| summarize FileCount = count(), DistinctFiles = dcount(ObjectName)
    by AccountDisplayName, bin(TimeGenerated, 1h)
| where DistinctFiles > 50   // Unusual volume of distinct files in 1 hour
| order by DistinctFiles desc
```
