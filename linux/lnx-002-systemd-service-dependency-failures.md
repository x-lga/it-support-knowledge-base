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

## Step 1 - Identify the Failed Service and Its State

```bash
# Show all failed units — start here
systemctl --failed
# Output columns:
#   UNIT    — service name
#   LOAD    — Loaded (unit file found) or not-found
#   ACTIVE  — failed / inactive / active
#   SUB     — specific substates: failed, dead, running, exited
#   DESCRIPTION

# Get the full status of a specific failed service
systemctl status nginx.service
# Reads: Active state, exit code, last log lines, PID, memory, cgroup

# Extended log output for the failed service (most useful for root cause)
journalctl -u nginx.service -n 50 --no-pager
# -u: unit filter
# -n 50: last 50 lines
# --no-pager: output everything without interactive paging (better for copy-paste)

# Show logs since the last boot only (removes noise from previous sessions)
journalctl -u nginx.service -b --no-pager
# -b: current boot only
```

---

## Step 2 - Trace the Dependency Chain

```bash
# Show what a service depends on (what must be running before it starts)
systemctl list-dependencies nginx.service
# Tree view showing all dependencies with their current state
# Red = failed, green = active, white = inactive

# Show what depends on a service (what will fail if this service fails)
systemctl list-dependencies nginx.service --reverse
# This is critical when a shared service (like database or network) fails
# It shows every service that will be affected

# Show the full unit file including all directives
systemctl cat nginx.service
# Look at: Requires=, Wants=, After=, Before=, BindsTo=
# These tell you the exact dependency model

# Check the ordering: does nginx start AFTER the network is actually ready?
# Difference between network.target and network-online.target:
#   network.target      = network interfaces are UP (may not have IP yet)
#   network-online.target = network is fully configured and reachable
# A service needing to reach a database must use network-online.target, not network.target
```

---


