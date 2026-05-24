# WIN-005 - Shadow Copy and VSS: Recovery Procedures and Common Failures

**Article ID:** WIN-005
**Category:** Windows - Volume Shadow Copy Service
**Severity:** P2 (data recovery needed, backup unavailable)
**Cert alignment:** CompTIA A+
**Last verified:** 2026-07

---

## What Shadow Copies Are and Are Not

Volume Shadow Copy (VSS) creates point-in-time snapshots of volumes. They are
useful for quickly recovering accidentally deleted or overwritten files without
a formal backup restore. They are NOT a backup substitute:

- Shadow copies live on the same physical disk as the data they protect
- If the disk fails, both the data and the shadow copies are lost
- Ransomware typically deletes shadow copies as one of its first actions
  (`vssadmin delete shadows /all /quiet` is in almost every ransomware strain)
- Shadow copies do not span across drives (each volume has its own shadow copies)

---

## Step 1 - List Available Shadow Copies

```powershell
# List all shadow copies on all volumes
$ShadowCopies = Get-WmiObject Win32_ShadowCopy
foreach ($Shadow in $ShadowCopies) {
    $Age = [math]::Round(((Get-Date) - [Management.ManagementDateTimeConverter]::ToDateTime($Shadow.InstallDate)).TotalDays, 1)
    Write-Host ""
    Write-Host "Shadow Copy ID : $($Shadow.ID)"
    Write-Host "Volume         : $($Shadow.VolumeName)"
    Write-Host "Created        : $([Management.ManagementDateTimeConverter]::ToDateTime($Shadow.InstallDate))"
    Write-Host "Age            : $Age days ago"
}

# Quick check using vssadmin
& vssadmin list shadows /for=C:
```

---

## Step 2 - Recover a Specific File or Folder from Shadow Copy

```powershell
# Mount a shadow copy as a drive letter for easy file access
$ShadowID = "{GUID-of-shadow-copy}"   # From Step 1 output
$Shadow   = Get-WmiObject Win32_ShadowCopy | Where-Object { $_.ID -eq $ShadowID }

# Create a symbolic link to access the shadow copy contents
$ShadowPath = $Shadow.DeviceName + "\"
$LinkPath   = "C:\ShadowMount"

cmd /c "mklink /d $LinkPath $ShadowPath"
Write-Host "Shadow copy mounted at: $LinkPath"
Write-Host "Browse to $LinkPath to find and copy files"
Write-Host ""
Write-Host "When done: cmd /c 'rmdir $LinkPath'"

# Navigate to the shadow copy (e.g., find deleted document)
# $LinkPath\Users\jsmith\Documents\deleted-file.docx
# Copy it to the desired location
```

---

## Step 3 - Diagnose VSS Writer Failures

VSS failures during backup jobs typically come from a VSS writer (a component
that ensures data consistency during snapshot). When a backup job fails with
a VSS error, this is the diagnostic path:

```cmd
:: List all VSS writers and their current state
vssadmin list writers

:: Healthy output for each writer:
:: Writer name: 'Microsoft Exchange Writer'
::   Writer Id: {xxx}
::   State: [1] Stable
::   Last error: No error
::
:: Unhealthy output:
::   State: [8] Failed
::   Last error: Timed out
```

```powershell
# Restart VSS writers that are in Failed or Waiting state
# The writers are tied to Windows services — restarting the service resets the writer

# Common writer → service mappings:
$WriterServiceMap = @{
    "System Writer"                   = "CryptSvc"
    "ASR Writer"                      = "VSS"
    "COM+ REGDB Writer"               = "VSS"
    "Registry Writer"                 = "VSS"
    "Shadow Copy Optimization Writer" = "VSS"
    "WMI Writer"                      = "Winmgmt"
    "Microsoft Exchange Writer"       = "MSExchangeIS"
    "SQL Server Writer"               = "MSSQLSERVER"
}

# Restart the VSS service itself (resets all built-in writers)
Restart-Service -Name "VSS" -Force
Start-Sleep -Seconds 10

# Verify writers recovered
& vssadmin list writers
```

---

