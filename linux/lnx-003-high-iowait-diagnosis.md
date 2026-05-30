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

## Step 2 - Identify Which Process Is Causing the I/O

```bash
# Method 1: iotop — shows per-process disk I/O in real time (most useful)
sudo iotop -o -P
# -o: only show processes that are actively doing I/O (reduces noise)
# -P: show processes, not threads
# Columns: TID, PRIO, USER, DISK READ, DISK WRITE, SWAPIN, IO>, COMMAND
# Sort by DISK WRITE column — the top entry is your culprit

# If iotop is not installed:
sudo apt install iotop          # Debian/Ubuntu
sudo yum install iotop          # RHEL/CentOS

# Method 2: pidstat — per-process I/O statistics
sudo pidstat -d 2 10
# -d: disk I/O stats, 2: 2-second intervals, 10: 10 measurements
# Shows: UID, PID, kB_rd/s (KB read/sec), kB_wr/s (KB write/sec), Command

# Method 3: lsof for a specific process — which files is it reading/writing?
sudo lsof -p [PID] | grep -E "REG|DIR"
# Shows every file the process has open — identifies WHICH files are being accessed
```

---

## Step 3 - Investigate Common Causes

**Cause 1 - Log file growing out of control:**

```bash
# Find which log files are growing fastest
sudo inotifywait -m /var/log -r -e close_write 2>/dev/null | head -30
# Shows every file write in /var/log — the most frequently updated file is the problem

# Check sizes of all log files
du -sh /var/log/* | sort -rh | head -20

# Check if logrotate is working
ls -lh /var/log/syslog*   # Should show rotated files (syslog.1, syslog.2.gz, etc.)
# If no rotated files exist: logrotate may not be configured for this log
cat /etc/logrotate.d/rsyslog   # Check the rotation config
```

**Cause 2 - Swap being used heavily (disk I/O is swap read/write):**

```bash
# Check swap usage
free -h
# If swap Used is significant: the system is short of RAM
# Processes are being swapped to disk — this causes high iowait

# Find which processes are using the most memory (causing swap pressure)
ps aux --sort=-%mem | head -15

# Check the swappiness setting (higher = more aggressive swapping)
cat /proc/sys/vm/swappiness
# Default is 60. For servers with plenty of RAM: set to 10
# This makes the kernel prefer RAM over swap
sudo sysctl vm.swappiness=10   # Immediate (resets on reboot)
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf   # Persistent
```

**Cause 3 - Database performing a large query or index rebuild:**

```bash
# For MySQL/MariaDB — check active queries
mysql -u root -p -e "SHOW PROCESSLIST\G" | grep -A5 "State: Copying\|State: Sorting\|State: Writing"
# State: Copying to tmp table = large sort/join operation reading lots of data
# State: Writing to net = slow client pulling large result set

# For PostgreSQL — check active queries with I/O stats
sudo -u postgres psql -c "
SELECT pid, state, wait_event_type, wait_event, query
FROM pg_stat_activity
WHERE wait_event_type = 'IO'
ORDER BY query_start;
"
```


