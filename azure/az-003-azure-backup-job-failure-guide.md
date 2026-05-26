# AZ-003 - Azure Backup Job Failure: Diagnosis and Recovery

**Article ID:** AZ-003
**Category:** Azure - Backup and Recovery
**Severity:** P2 (backup failing - RPO at risk)
**Cert alignment:** AZ-104
**Last verified:** 2026-07

---

## Why Backup Failures Require Prompt Attention

A single backup failure is a warning. Multiple consecutive failures mean
the RPO has been breached - if a failure now occurs, recovery will not
be to the committed recovery point. For a 15-minute RPO SQL MI deployment,
missed backup jobs are a compliance issue, not just an operational one.

Azure Backup failure codes fall into three categories:
1. **Agent/connectivity failures** - the VM cannot reach the backup service
2. **VSS/application consistency failures** - backup triggered but data could not be made consistent
3. **Policy/configuration issues** - the backup policy or vault is misconfigured

---

## Step 1 - Identify the Error Code

```
Azure Portal → Recovery Services Vault → [Vault Name] →
  Backup Jobs → Failed

Click the failed job to see:
  Error code : e.g., UserErrorGuestAgentStatusUnavailable
  Error message: Plain language explanation
  Recommended action: What Microsoft suggests
```

**Most common error codes and their causes:**

| Error Code | Cause | Resolution |
|-----------|-------|-----------|
| `UserErrorGuestAgentStatusUnavailable` | Azure Guest Agent is stopped or not communicating | Restart the Azure Guest Agent service on the VM |
| `UserErrorVmNotInDesiredState` | VM is deallocated (stopped) | Start the VM before backup runs, or configure backup to work with deallocated VMs |
| `ExtensionOperationFailed` | Backup extension encountered an error | Check VM extension health; reinstall backup extension |
| `UserErrorUnsupportedDiskSizeForSnapshot` | Disk size exceeds the snapshot limit | Check disk size; Premium disks > 4 TB have specific requirements |
| `BackupOperationFailed` | Generic failure | Check VSS writer status on the VM |
| `UserErrorDiskIsNot512AndIsBeingConverted` | Disk is being converted (sector size change) | Wait for disk conversion to complete; retry backup |

---

## Step 2 - Fix Azure Guest Agent Issues

```powershell
# Connect to the VM via Bastion, then run:
# Check Azure Guest Agent (Windows)
Get-Service -Name "WindowsAzureGuestAgent", "WindowsAzureTelemetryService",
    "WaAppAgent" | Select-Object Name, Status, StartType

# If any are stopped:
Start-Service -Name "WindowsAzureGuestAgent" -ErrorAction SilentlyContinue
Start-Service -Name "WaAppAgent" -ErrorAction SilentlyContinue

# Check Guest Agent version (minimum 2.7.41491.971 for current Azure Backup)
$GuestAgentPath = "C:\WindowsAzure\GuestAgent*"
Get-ChildItem $GuestAgentPath | Sort-Object LastWriteTime -Descending | Select-Object -First 1

# If Guest Agent needs reinstall:
# Download latest: https://go.microsoft.com/fwlink/?LinkID=394789
# Uninstall existing: Add/Remove Programs → Microsoft Azure VM Agent
# Install new version
# Restart both services above
```

---

## Step 3 - Fix VSS Writer Failures (for Application-Consistent Backups)

Application-consistent backup for SQL Server and Exchange requires VSS writers
to be healthy. Backup fails if a writer is in a Failed state:

```powershell
# On the VM — check all VSS writer states
$VSS = & vssadmin list writers
Write-Host $VSS

# Writers in Failed state need their associated service restarted
# SQL Server writer → restart "SQL Server VSS Writer" service
Get-Service -Name "SQLWriter" | Restart-Service -Force

# System writer (and others) → restart VSS service
Get-Service -Name "VSS" | Restart-Service -Force
Start-Sleep -Seconds 15

# Re-check
& vssadmin list writers
```

---

## Step 4 - Trigger a Manual Backup After Resolution

```bash
# After fixing the underlying issue, trigger an on-demand backup to verify
az backup protection backup-now \
    --resource-group rg-banking-prod \
    --vault-name rsv-contoso-prod \
    --container-name "IaasVMContainer;iaasvmcontainerv2;rg-banking-prod;vm-app-01" \
    --item-name "vm;iaasvmcontainerv2;rg-banking-prod;vm-app-01" \
    --backup-management-type AzureIaasVM \
    --retain-until $(date -d "+30 days" '+%d-%m-%Y')

# Monitor the job
az backup job list \
    --resource-group rg-banking-prod \
    --vault-name rsv-contoso-prod \
    --output table \
    --query "[?status!='Completed']"
```


---





