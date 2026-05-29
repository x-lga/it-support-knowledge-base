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

## Step 3 - Recover from a System That Will Not Boot

When the root filesystem itself is corrupt and the system cannot boot to a shell:

```bash
# OPTION A: Recovery mode (systems with GRUB)
# 1. On boot, hold Shift (BIOS) or press Esc (UEFI) to show GRUB menu
# 2. Select: Advanced options for Ubuntu → Recovery mode
# 3. From recovery menu: select "root — Drop to root shell prompt"
# 4. Remount root filesystem as read-write to run fsck:
mount -o remount,ro /
fsck -y /dev/sda1   # Replace with your root partition

# OPTION B: Boot from live USB (more reliable for severe corruption)
# 1. Boot from Ubuntu or Debian live USB
# 2. Open terminal — do NOT mount the corrupted filesystem
# 3. Identify the corrupted partition:
lsblk -f
# 4. Run fsck:
sudo fsck -y /dev/sda1
# 5. If successful, mount and verify:
sudo mount /dev/sda1 /mnt
ls /mnt   # Confirm directory structure looks intact
sudo umount /mnt

# OPTION C: Single-user mode (older RHEL/CentOS systems)
# At GRUB menu: press 'e' to edit boot entry
# Find the line starting with 'linux' — append: single
# Press Ctrl+X to boot
# System boots to single-user root shell
# Run: fsck -y /dev/sda1
```

---

## Step 4 - Recover Specific Files Using testdisk / photorec

When fsck completes but specific files are missing or inaccessible:

```bash
# Install testdisk (includes photorec for file recovery)
sudo apt install testdisk

# Run testdisk to recover deleted or lost partitions
sudo testdisk /dev/sda
# Interactive menu: Analyse → Quick Search → found partitions → Write

# Run photorec for file-level recovery (recovers by file signature, not filesystem)
sudo photorec /dev/sda1
# Choose destination directory on a DIFFERENT disk — never recover to the same disk
# photorec ignores filenames (recovers by magic bytes) — best for photos, docs, PDFs
```

---

## Step 5 - Prevent Future Corruption

```bash
# Check for scheduled filesystem checks (ext4)
sudo tune2fs -l /dev/sda1 | grep -E "Mount count|Maximum mount count|Check interval"
# Mount count: how many times mounted since last fsck
# Maximum mount count: -1 means disabled (check by count is off by default on modern systems)
# Modern ext4 relies on the journal — manual fsck scheduling is less necessary

# Enable journal on an ext4 filesystem (should already be enabled)
sudo tune2fs -l /dev/sda1 | grep "Filesystem features" | grep -c "has_journal"
# Should return 1 — if 0: enable with sudo tune2fs -O has_journal /dev/sda1

# For XFS — ensure journal is not disabled
sudo xfs_info /dev/sda1 | grep "log"
# Should show: log= with a block count — if log=0: XFS logging is disabled

# Check /etc/fstab for correct options
cat /etc/fstab
# Each entry should have the pass (last column) set correctly:
# Root filesystem (/): pass = 1
# Other local filesystems: pass = 2
# 0 means no fsck at boot — acceptable for network or removable filesystems
```

---

## Known Edge Cases

**ext4 journal replay vs full fsck:**
After an unclean shutdown, ext4 replays the journal automatically on the next
mount. This is not the same as fsck. Journal replay fixes metadata that was
mid-write. fsck checks the entire filesystem structure. If journal replay
completes but applications still report I/O errors, run a full fsck.


**btrfs corruption:**
btrfs has its own repair tool and the procedure differs significantly from ext4.
Do not run `fsck` on a btrfs filesystem - use `sudo btrfs check --repair /dev/sda1`.
Running ext4 fsck on btrfs will make things dramatically worse.




