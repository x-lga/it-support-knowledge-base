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

## Step 3 - Common Failure Patterns and Their Resolutions

**Pattern 1 - Service fails because a dependency is not yet ready (race condition):**

```bash
# Symptom: Service starts fine manually but fails on boot
# Diagnosis: Service starts before its dependency is fully ready
systemctl show nginx.service | grep -E "After|Wants|Requires"
# If After=network.target but the service tries to connect to a database:
# the network may be up but DNS not yet resolving, or the DB server not yet accepting connections

# Fix: Add network-online.target to the After= and Wants= directives
# Edit the service unit file (override, not direct edit):
sudo systemctl edit nginx.service
# This creates /etc/systemd/system/nginx.service.d/override.conf
# Add:
[Unit]
After=network-online.target
Wants=network-online.target

# Reload and test
sudo systemctl daemon-reload
sudo systemctl restart nginx.service
sudo systemctl status nginx.service
```

**Pattern 2 - Service fails due to permission error on a file or socket:**

```bash
# Symptom: journalctl shows "Permission denied" on a specific file
journalctl -u myapp.service -b | grep -i "permission\|denied\|EPERM\|EACCES"

# Check the file permissions
ls -la /var/run/myapp.sock
ls -la /var/log/myapp/
ls -la /etc/myapp/myapp.conf

# Check what user the service runs as
systemctl show myapp.service | grep "User\|Group"
# If User=myapp, the service runs as the myapp user
# The directories and files it accesses must be owned by or readable by that user

# Fix ownership:
sudo chown -R myapp:myapp /var/log/myapp/
sudo chmod 750 /var/log/myapp/

# For socket files created at runtime:
# Check RuntimeDirectory in the unit file:
systemctl cat myapp.service | grep RuntimeDirectory
# RuntimeDirectory=myapp creates /run/myapp owned by the service user at start
```

**Pattern 3 - Service keeps restarting (restart loop):**

```bash
# Symptom: service shows "activating" repeatedly or Active: failed (Result: exit-code)
systemctl status myapp.service
# Check the restart counter
systemctl show myapp.service | grep NRestarts

# Check the ExecStart command actually exists and is executable
systemctl cat myapp.service | grep ExecStart
ls -la /usr/bin/myapp   # Or wherever the binary is

# Check if systemd is killing it due to timeout
journalctl -u myapp.service -b | grep -E "Timeout|killed|SIGTERM|SIGKILL"
# If "start operation timed out": the service is taking too long to report ready
# Fix: Increase TimeoutStartSec in an override:
sudo systemctl edit myapp.service
# Add:
[Service]
TimeoutStartSec=120

# Check the restart policy — is it restarting too aggressively?
systemctl show myapp.service | grep -E "Restart=|RestartSec="
# Restart=always with RestartSec=0 means it restarts instantly — fills the journal
# Consider: RestartSec=5 to add a 5-second delay between restarts
```

---

## Step 4 - System Fails to Reach Target (Boot Hangs or Emergency Shell)

When the system boots to an emergency shell or hangs at boot with a timeout message:

```bash
# From the emergency shell, identify which unit failed to start
systemctl --failed
journalctl -xb   # Full boot log with explanatory annotations (-x flag)

# Common boot blockers:
# 1. /etc/fstab entry for a disk that does not exist or is not mounted
#    Fix: boot from live USB, edit /etc/fstab, comment out the problematic line
#    Add 'nofail' option to non-critical mounts: UUID=xxx /data ext4 defaults,nofail 0 2

# 2. A service with WantedBy=multi-user.target that always fails
#    Fix: disable the service temporarily
#    systemctl disable --now myapp.service
#    Then investigate and fix the service before re-enabling

# 3. Network interface name changed (predictable interface names issue)
#    Fix: check /etc/netplan/*.yaml or /etc/network/interfaces
#    The interface name in the config must match the actual interface name
ip link show   # Shows actual interface names
```

---

## Step 5 - Useful systemd Diagnostic Commands Reference

```bash
# Show the boot timeline — which services took longest to start
systemd-analyze blame
# Shows each service sorted by startup time — identify bottlenecks

# Generate an SVG visualisation of the boot dependency chain
systemd-analyze dot | dot -Tsvg > boot-dependencies.svg
# Open the SVG in a browser to see the complete dependency graph visually

# Show the critical chain — the sequence of units that determined total boot time
systemd-analyze critical-chain

# Check a unit file for syntax errors before reloading
systemd-analyze verify /etc/systemd/system/myapp.service

# Show all properties of a unit (comprehensive)
systemctl show nginx.service | less

# List all unit files and their enabled/disabled state
systemctl list-unit-files --type=service | grep -v disabled | head -40

# Mask a service (stronger than disable — prevents it from being started at all)
sudo systemctl mask problematic.service
# Unmask: sudo systemctl unmask problematic.service
```

---

