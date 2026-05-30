# LNX-003 - High iowait Diagnosis: Finding What Is Hammering the Disk

**Article ID:** LNX-003
**Category:** Linux - Performance Analysis
**Severity:** P2 (system sluggish, applications slow) | P1 (system unresponsive)
**Cert alignment:** CompTIA A+, CompTIA Network+
**Last verified:** 2026-07

---

## What iowait Actually Means

iowait is the percentage of time the CPU is idle AND waiting for disk I/O to
complete. It is frequently misunderstood as a measure of disk utilisation - it
is not. It is a measure of CPU time wasted waiting for the disk.

iowait of 30% means the CPU was idle for 30% of the measured period specifically
because it was waiting for the disk to respond. This matters because:
- iowait appears as high CPU utilisation in tools like `top` (it shows in the CPU
  idle category but is distinct from genuinely idle)
- Systems with high iowait feel completely unresponsive even though CPU utilisation
  for applications may be low - the CPU cannot do anything useful because it is
  blocked waiting for disk
- High iowait can mask what is actually causing it - the process causing disk
  saturation may not be the one using the most CPU

---

## Step 1 - Confirm iowait Is the Issue

```bash
# Check current iowait percentage
top
# Press '1' to show per-CPU breakdown
# Look at the 'wa' column in the CPU line:
# %Cpu(s): 2.0 us, 0.5 sy, 0.0 ni, 55.0 id, 40.0 wa, 0.0 hi, 0.0 si, 0.0 st
#                                              ^^^^^^ this is iowait
# wa > 10% is elevated. wa > 25% is significant. wa > 50% is severe.

# More precise iowait measurement with iostat
iostat -x 2 5
# -x: extended stats, 2: 2-second intervals, 5: 5 measurements
# Key columns:
#   %iowait  — CPU time waiting for I/O (same as 'wa' in top)
#   %util    — percentage of time the device was busy (100% = device is saturated)
#   await    — average time (ms) for I/O requests to complete
#   svctm    — average service time (ms) — if much lower than await: queue is backing up
#   r/s, w/s — reads and writes per second

# If %util on a device is consistently 100%: the disk itself is the bottleneck
# If await >> svctm: requests are queueing — disk cannot keep up with the rate
```

---


