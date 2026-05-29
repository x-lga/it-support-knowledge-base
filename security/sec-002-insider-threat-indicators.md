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

**Large outbound email with attachments:**
```kql
// M365 Defender — emails with large attachments sent by a specific user
EmailEvents
| where TimeGenerated > ago(7d)
| where SenderFromAddress == "jsmith@contoso.com"
| where AttachmentCount > 0
| summarize EmailCount = count(), TotalAttachments = sum(AttachmentCount)
    by SenderFromAddress, RecipientEmailAddress
| where TotalAttachments > 20
| order by TotalAttachments desc
```

**USB/removable device usage:**
```powershell
# Windows Event Log — check for removable storage insertions
# Requires USB storage auditing via GPO
Get-WinEvent -LogName "Microsoft-Windows-DriverFrameworks-UserMode/Operational" |
    Where-Object { $_.Id -eq 2003 -or $_.Id -eq 2100 } |
    Select-Object TimeCreated, Message |
    Select-Object -First 20
```

---

## Behavioural Indicators an L1 Engineer Might Observe

| Indicator | Example | Reason for Concern |
|-----------|---------|-------------------|
| Departing employee data access | User under notice accessing HR or financial files they do not normally touch | Pre-departure data collection |
| After-hours login combined with file bulk access | Login at 22:00 followed by 500+ file downloads | Normal employees do not bulk-download at night |
| Large email to personal address | 50 MB attachment sent to gmail.com from a work account | Exfiltration |
| Forwarding rules set up | Inbox rules forwarding all email to an external address | Established email exfiltration |
| New cloud sync tools installed | Dropbox, Google Drive, or a personal sync client installed on a work machine | Shadow IT exfiltration path |

---

## L1 Action When an Indicator Is Observed

**What to do:**
1. Document everything - exact timestamps, what was observed, where you saw it
2. Do NOT alert the employee or manager
3. Do NOT access the employee's files or email beyond what is needed to document
4. Raise a confidential ticket to L2 Security with all documentation attached
5. Follow L2 Security's instructions - do not take any further independent action


**What not to do:**
Do not discuss the observation with colleagues. Do not mention it to the employee's
manager unless explicitly instructed by L2 Security. Insider threat investigations
require confidentiality - premature disclosure allows the subject to destroy evidence.

The L1 engineer's role is observation and documentation. Investigation and response
belongs to L2 Security and HR.


---

