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

