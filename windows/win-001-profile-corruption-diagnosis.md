# WIN-001 - Windows User Profile Corruption: Diagnosis and Recovery

**Article ID:** WIN-001
**Category:** Windows - User Profile Management
**Severity:** P3 (single user) | P2 (multiple users simultaneously)
**Cert alignment:** CompTIA A+
**Last verified:** 2026-07

---

## What This Article Covers

Windows user profile corruption is one of the most mishandled L1 tickets.
The symptom ("I can't log in" or "my desktop is blank") is identical to a dozen
other issues, so the first failure mode is treating it as a password or lockout
issue and wasting 20 minutes on AD before discovering the profile is the problem.
The second failure mode is deleting the profile without verifying data can be
recovered. This article prevents both.

A corrupted profile typically manifests as one of three presentations:
1. User logs in and gets a temporary profile ("You have been signed in with a
   temporary profile" notification in the bottom right)
2. User logs in but desktop, Start menu, and taskbar are all blank or missing
3. User cannot log in at all - login fails with "The User Profile Service failed
   the sign-in" error even with correct credentials

---

## Distinguishing Profile Corruption from Other Issues

Before assuming profile corruption, rule these out first:

| Symptom | Possible Cause | Quick Check |
|---------|---------------|------------|
| "Temporary profile" message | Profile corruption OR disk full | Check C: free space first — profile creation fails silently if disk is full |
| Blank desktop after login | Profile corruption OR GPO issue | Does another user on same machine get a blank desktop? If yes: GPO, not profile |
| Login fails entirely | Profile OR password OR AD | Does the user log in successfully on a different machine? If yes: local profile issue |
| Desktop loads but apps crash | Profile corruption OR roaming profile conflict | Is roaming profile configured? Conflicts cause app settings corruption |

---

## Step 1 — Identify Which Profile Is Loading

```powershell
# Run on the affected machine as Administrator
# This shows ALL profiles on the machine and their paths
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\*" |
    Select-Object PSChildName, ProfileImagePath, State |
    Format-Table -AutoSize

# Profile State values:
# 0  = Profile loaded and healthy
# 1  = Profile was loaded temporarily (temporary profile was created)
# 4  = Profile corrupt — do not delete yet
# 8  = Profile requires mandatory sync
# 256 = Profile being created (transient)
# 516 = Profile loaded but with errors
```

The PSChildName column shows the user's SID. Cross-reference with AD:
```powershell
# Convert SID to username
$SID = "S-1-5-21-[numbers]"
Get-ADUser -Filter { SID -eq $SID } | Select-Object Name, SamAccountName
```

---

