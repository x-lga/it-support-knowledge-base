# SEC-001 - Ransomware: Initial Triage for L1

**Article ID:** SEC-001
**Category:** Security - Incident Response
**Severity:** P1 always
**Cert alignment:** CompTIA Security+
**Last verified:** 2026-07

---

## The Three Rules Before You Do Anything Else

**Rule 1: Do not shut down the machine.**
Ransomware often runs entirely or partially in memory. Powering off destroys
volatile memory that forensics needs. Some ransomware strains also encrypt
partially and the encryption can be reversed from memory artefacts. If the
machine has BitLocker and ransomware has encrypted the keys, powering off
may lock you out of the disk permanently.


**Rule 2: Isolate immediately - from the network, not from power.**
Disconnect the ethernet cable and disable WiFi. Every second of network
connectivity allows the ransomware to spread laterally, encrypt more files,
or exfiltrate data. Isolation stops the spread. Power-off destroys evidence.

**Rule 3: Call L2 Security by voice before doing anything further.**
Ransomware is a P1 that requires a security incident response, not a support
ticket. Your role is to isolate and report, not to investigate and remediate
independently. Attempting remediation without L2 involvement frequently makes
the situation worse - recovery tools run against the wrong strain, encryption
spreads further, or forensic evidence is destroyed.

---
