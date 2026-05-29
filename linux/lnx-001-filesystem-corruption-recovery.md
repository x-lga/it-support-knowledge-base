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

