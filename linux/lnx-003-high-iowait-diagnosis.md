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

**Cause 4 - Backup job running (expected but impacting performance):**

```bash
# Check if a backup process is the cause
sudo iotop -o | grep -E "rsync|tar|cp|dd|bacula|veeam|borg"

# If a backup is running at an unexpected time or with unexpected intensity:
# Check cron jobs
sudo crontab -l
crontab -l -u root
cat /etc/cron.d/*

# Limit backup I/O priority to reduce impact on other workloads
# Using ionice — set the backup process to best-effort class 3 (lowest priority)
sudo ionice -c 3 -p [backup-PID]
# Or when starting a new backup job:
ionice -c 3 rsync -av /data/ /backup/
```

---

## Step 4 - Measure Disk Health and Performance Baseline

```bash
# Check raw disk read speed (sequential read — measure baseline performance)
sudo hdparm -Tt /dev/sda
# Timing cached reads: should be > 1 GB/s (RAM speed)
# Timing buffered disk reads: SSD should be 200–500 MB/s, HDD 80–150 MB/s
# If significantly below baseline: disk hardware is degraded

# Check disk queue depth
cat /sys/block/sda/queue/nr_requests
# Default is 128. Higher queue depth can help SSDs (they handle parallel I/O well)
# Does not help HDDs (they are inherently sequential)

# Check current disk scheduler
cat /sys/block/sda/queue/scheduler
# Options: [mq-deadline] none bfq kyber
# For SSDs: 'none' or 'mq-deadline' are appropriate
# For HDDs: 'bfq' (Budget Fair Queueing) provides better interactive performance

# Change scheduler (immediate, resets on reboot)
echo mq-deadline | sudo tee /sys/block/sda/queue/scheduler

# Make persistent via udev rule:
echo 'ACTION=="add|change", KERNEL=="sda", ATTR{queue/scheduler}="mq-deadline"' | \
    sudo tee /etc/udev/rules.d/60-scheduler.rules
```

---

## Step 5 - Resolution Summary and Escalation Criteria

**Resolve at L1:**
- Log file growing out of control: truncate, fix logrotate, restart the logging service
- Backup job causing iowait: renice/ionice the backup process, reschedule to off-hours
- Swap pressure from a known application: identify and restart the memory-leaking process

**Escalate to L2:**
- iowait is high and the cause cannot be identified after Steps 2–3
- SMART data shows disk errors (bad sectors, reallocated sectors, pending sectors)
- `%util` on the device is 100% with no obvious single process causing it
- iowait is consistently high across multiple reboots despite no identifiable process


**Escalation package:**
```
Output of: iostat -x 2 5
Output of: sudo iotop -o -P (5 second capture)
Output of: sudo pidstat -d 2 5
Output of: sudo smartctl -a /dev/[disk]
Output of: free -h
Output of: df -h
Time of day and whether this is constant or periodic
Any recent changes: new software installed, cron jobs added, disk replaced
```

---

## Known Edge Cases

**High iowait on a VM in Azure (or other cloud):**
Cloud VMs share physical storage with other tenants. "Noisy neighbour" I/O
contention can cause elevated iowait even when the VM itself is not doing
anything. Check the Azure VM metric "OS Disk Queue Depth" in Azure Monitor —
if the queue depth is 0 but iowait is high, the issue is at the hypervisor layer,
not within the VM. Escalate to a cloud support ticket.

**iowait high but iotop shows nothing:**
Kernel threads (kworker, kswapd) doing I/O do not always appear clearly in iotop.
Use `sudo cat /proc/diskstats` and compare two snapshots 5 seconds apart to see
raw I/O counters per disk. If writes are occurring but no process is visible,
it may be journalling overhead or dirty page writeback - check `/proc/meminfo`
for `Dirty:` value. A very high dirty page count means the kernel is flushing
a large backlog to disk.


---



