# WIN-002 - WMI Repository Corruption: Diagnosis and Rebuild

**Article ID:** WIN-002
**Category:** Windows - WMI / CIM
**Severity:** P2 (management tools failing, monitoring broken)
**Cert alignment:** CompTIA A+
**Last verified:** 2026-07

---

## Why WMI Corruption Matters

Windows Management Instrumentation (WMI) is the plumbing that everything else
depends on: System Center agents, Defender for Endpoint sensors, monitoring tools
(PRTG, Zabbix), PowerShell CIM cmdlets, remote management, hardware inventory,
and application deployment tools all communicate through WMI. When the WMI
repository is corrupt, none of these work - but the machine appears healthy to
the user because WMI failures are silent from the desktop perspective.

Common symptoms that actually have WMI as the root cause:
- PowerShell `Get-CimInstance` or `Get-WmiObject` fails with unexpected errors
- Defender for Endpoint reporting shows the machine as inactive despite being online
- SCCM/Intune enrollment succeeds but hardware inventory never populates
- PRTG WMI sensors show "Access denied" or "WMI not available" even with correct credentials
- `winmgmt /verifyrepository` returns "WMI repository is INCONSISTENT"
- Windows Task Scheduler or Event Viewer fails to display events

---

