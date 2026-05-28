# NET-003 - DHCP Scope Exhaustion: Diagnosis and Emergency Response

**Article ID:** NET-003
**Category:** Networking - DHCP
**Severity:** P1 (new devices cannot get IPs — service impact)
**Cert alignment:** CompTIA Network+, CompTIA A+
**Last verified:** 2026-07

---

## DHCP Exhaustion vs DHCP Failure

These are different issues with similar symptoms. Distinguish quickly:

| Test | DHCP Failure (server down) | DHCP Exhaustion (scope full) |
|------|--------------------------|----------------------------|
| Existing device can renew lease? | No - DHCP server unreachable | Yes - renewal extends an existing lease |
| New device gets an IP? | No | No - pool is full |
| DHCP server shows in Services? | No | Yes - service is running |
| Leases report shows available? | N/A | 0 available |

---
