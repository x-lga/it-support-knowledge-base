# LNX-002 - systemd Service Dependency Failures: Diagnosis and Resolution

**Article ID:** LNX-002
**Category:** Linux - systemd and Service Management
**Severity:** P2 (critical service down) | P1 (system fails to boot due to service failure)
**Cert alignment:** CompTIA A+
**Last verified:** 2026-07

---

## Why systemd Dependencies Create Non-Obvious Failures

systemd starts services in dependency order. Service A may require Service B
to be active before it starts. If Service B fails, Service A never starts —
and the error you see is on Service A, even though the root cause is Service B.
Following the visible error without understanding the dependency chain sends
the investigation in the wrong direction.

Additionally, systemd has multiple dependency types that behave differently:
- `Requires=` — hard dependency: if the required unit fails, this unit also fails
- `Wants=` — soft dependency: if the wanted unit fails, this unit continues anyway
- `After=` — ordering only: this unit starts after the named unit, no failure coupling
- `BindsTo=` — stricter than Requires: if the bound unit stops, this unit stops too

A service that has `After=network.target` but only `Wants=network-online.target`
may start before the network is actually ready — a common source of "service
starts fine manually but fails on boot" issues.

---

