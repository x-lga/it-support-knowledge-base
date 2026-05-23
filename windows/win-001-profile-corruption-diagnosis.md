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

## Step 2 — Examine the Profile for Data Integrity

Before attempting any repair, verify the profile data is not at risk:

```powershell
# Check the profile folder size and last modified date
$ProfilePath = "C:\Users\[username]"
$ProfileInfo = Get-ChildItem $ProfilePath -Recurse -ErrorAction SilentlyContinue |
    Measure-Object -Property Length -Sum
Write-Host "Profile size: $([math]::Round($ProfileInfo.Sum / 1MB, 0)) MB"
Write-Host "Profile folder exists: $(Test-Path $ProfilePath)"

# Check for the NTUSER.DAT file — the core of the profile
Test-Path "$ProfilePath\NTUSER.DAT"
# If False: the profile is missing its hive — reconstruction may be needed

# Check NTUSER.DAT is not locked (would indicate the user is currently logged on)
try {
    [System.IO.File]::Open("$ProfilePath\NTUSER.DAT",
        [System.IO.FileMode]::Open,
        [System.IO.FileAccess]::ReadWrite,
        [System.IO.FileShare]::None).Close()
    Write-Host "NTUSER.DAT is NOT locked — user is logged off"
} catch {
    Write-Host "NTUSER.DAT is LOCKED — user may still have an active session"
}
```

---

## Step 3 — Attempt Registry Repair (Preferred — Preserves Data)

Many "corrupted" profiles are actually registry hive loading failures that can
be resolved without touching the user's data:

```powershell
# Identify the SID for the affected user
$Username = "jsmith"
$UserSID = (Get-ADUser $Username).SID.Value
Write-Host "User SID: $UserSID"

# Check the ProfileList registry entry for this SID
$RegPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\$UserSID"
Get-ItemProperty $RegPath

# Look for a duplicate entry with .bak suffix — this is the most common corruption cause
# If both S-1-5-21-xxx-1234 AND S-1-5-21-xxx-1234.bak exist, Windows loaded
# the .bak (temporary) version instead of the real one
$BakPath = "$RegPath.bak"
if (Test-Path $BakPath) {
    Write-Host "DUPLICATE PROFILE ENTRY FOUND (.bak exists)"
    Write-Host "This is the most common profile corruption pattern"
    Write-Host ""
    Write-Host "Resolution steps:"
    Write-Host "1. Rename the current key (without .bak) to a safe backup"
    Write-Host "2. Rename the .bak key to remove the .bak suffix"
    Write-Host "3. Set the State value to 0 in the corrected key"
    Write-Host "4. Reboot and test"

    # Manual steps (cannot be fully scripted safely — registry key rename requires regedit):
    # In regedit.exe:
    # Navigate to: HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList
    # Right-click S-1-5-21-...-1234 → Rename to S-1-5-21-...-1234.corrupt
    # Right-click S-1-5-21-...-1234.bak → Rename to S-1-5-21-...-1234
    # Open the renamed key → Set State = 0
    # Close regedit, reboot, test login
}
```

---

## Step 4 — Create a New Profile and Migrate Data

Use this path only when Step 3 fails or when the profile data itself is corrupt:

```powershell
# Step A: Create a new temporary admin account to log in with
# (Cannot migrate profile data while logged in as the affected user)
$TempPass = ConvertTo-SecureString "TempAdmin@2026!" -AsPlainText -Force
New-LocalUser -Name "TempMigration" -Password $TempPass -FullName "Temp Migration"
Add-LocalGroupMember -Group "Administrators" -Member "TempMigration"

# Step B: Log off all sessions of the affected user
# (Profile folders cannot be copied while files are in use)
query user | Select-String $Username
# If sessions exist: logoff [session ID]

# Step C: After logging in as TempMigration, copy the critical data
# Do NOT copy the entire profile — this copies the corruption with it
# Copy only the user data directories
$SourceProfile = "C:\Users\jsmith"
$TempBackup    = "C:\ProfileBackup\jsmith"
New-Item -ItemType Directory -Path $TempBackup -Force

$DataFolders = @("Desktop", "Documents", "Downloads", "Pictures",
                  "Videos", "Music", "AppData\Roaming\Microsoft\Outlook",
                  "AppData\Roaming\Microsoft\Signatures",
                  "AppData\Local\Microsoft\Outlook\*.ost")

foreach ($Folder in $DataFolders) {
    $Source = Join-Path $SourceProfile $Folder
    $Dest   = Join-Path $TempBackup   $Folder
    if (Test-Path $Source) {
        Copy-Item -Path $Source -Destination $Dest -Recurse -Force -ErrorAction SilentlyContinue
        Write-Host "Copied: $Folder"
    }
}
Write-Host "Data backup complete: $TempBackup"

# Step D: Delete the corrupted profile entry from the registry
# (This forces Windows to create a fresh profile on next login)
Remove-Item -Path $RegPath -Recurse -Confirm:$false
Write-Host "Registry profile entry removed for $UserSID"
# Do NOT delete C:\Users\jsmith yet — keep until the new profile is verified

# Step E: Have the user log in — Windows creates a fresh profile
# Step F: Copy the backed-up data into the new profile
# Step G: After user confirms everything is working — delete C:\Users\jsmith.OLD
```

---

## Known Edge Cases

**Edge case 1 — Roaming profile conflicts:**
If the user has a roaming profile configured (GPMC → User Configuration → Folder
Redirection), the local profile may look corrupt but the issue is a sync conflict
between the local cache and the server copy. Check the Event Log for
`User Profile Service` source errors. Source `Event ID 1500` with error code
`0x80070057` = roaming profile path invalid or inaccessible.

**Edge case 2 — AV locking NTUSER.DAT:**
Some antivirus products scan NTUSER.DAT on login and hold a file lock that prevents
the profile service from loading it. Check whether the issue correlates with AV
signature updates. Temporary workaround: exclude NTUSER.DAT from on-access scanning
(must be approved by security team before implementing).

