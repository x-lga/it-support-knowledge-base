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


