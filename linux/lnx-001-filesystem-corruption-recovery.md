# LNX-001 - Linux Filesystem Corruption: Recovery Without Data Loss

**Article ID:** LNX-001
**Category:** Linux - Filesystem Management
**Severity:** P1 (system cannot boot or data inaccessible)
**Cert alignment:** CompTIA A+
**Last verified:** 2026-07

---

## Understanding When Filesystem Corruption Occurs

Linux filesystems rarely corrupt from normal operation. When corruption occurs,
it is almost always caused by one of:
1. Unclean shutdown (power loss, hard reset) while writes were in progress
2. Failing disk hardware (most common underlying cause)
3. Out-of-space condition that interrupted a write operation
4. Memory errors that corrupted data before it was written to disk

**Critical distinction:** fsck repairs filesystem metadata errors. It cannot
recover data that was lost due to hardware failure. Always check disk health
(SMART data) alongside filesystem repair — running fsck on a failing disk risks
losing more data during the repair than was lost in the original corruption.

---

## Step 1 - Assess Disk Health Before Attempting Repair

```bash
# Check SMART status on all disks
sudo smartctl -a /dev/sda
# Look for: Reallocated_Sector_Ct (any non-zero value = bad sectors present)
# Look for: Current_Pending_Sector (sectors with read errors awaiting reallocation)
# Overall health line: "SMART overall-health self-assessment test result: PASSED"
# or FAILED — FAILED means stop, escalate, restore from backup

# If smartctl is not installed:
sudo apt install smartmontools    # Debian/Ubuntu
sudo yum install smartmontools    # RHEL/CentOS

# Confirm which filesystem is on which partition before proceeding
lsblk -f
# Shows: NAME, FSTYPE, LABEL, UUID, MOUNTPOINT for every block device

df -T
# Shows: mounted filesystems with their type (ext4, xfs, btrfs, etc.)
```

---

## Step 2 - Run fsck (Filesystem Check and Repair)

**Important:** fsck must be run on an **unmounted** filesystem.
If the filesystem is mounted, fsck will refuse to run or warn strongly against it.

```bash
# For ext4 filesystem — most common on Ubuntu/Debian systems
sudo umount /dev/sda1
# If unmounting fails because the filesystem is the root (/):
# Reboot into recovery mode (hold Shift during boot → Advanced options →
# Recovery mode → Root shell with read-only filesystem)
# OR boot from a live USB and mount nothing — run fsck from there

# Run fsck with automatic repair (-y answers YES to all repair prompts)
# Use -y only when you understand the risks — it may delete orphaned inodes
sudo fsck -y /dev/sda1

# For more conservative repair that shows what it would do first:
sudo fsck -n /dev/sda1    # Dry run — reports issues without fixing
sudo fsck -p /dev/sda1    # Auto-repair only safe, unambiguous issues

# For XFS filesystems (CentOS/RHEL default):
sudo xfs_repair /dev/sda1
# Note: xfs_repair requires the filesystem to be unmounted AND clean
# If xfs_repair fails: try xfs_repair -L /dev/sda1 (zeroes the log — last resort)

# Verify repair was successful
sudo fsck -n /dev/sda1
# If output shows "clean" with no errors: repair succeeded
```

---


