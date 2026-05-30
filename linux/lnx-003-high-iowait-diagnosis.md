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

