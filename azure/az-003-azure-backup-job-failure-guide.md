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

