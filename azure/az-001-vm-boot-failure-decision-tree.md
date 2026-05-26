# AZ-001 - Azure VM Boot Failure: Complete Decision Tree

**Article ID:** AZ-001
**Category:** Azure - Virtual Machine Availability
**Severity:** P2 (non-critical VM) | P1 (production workload VM)
**Cert alignment:** AZ-104
**Last verified:** 2026-07

---

## Why a Decision Tree Works Better Than a Linear Procedure

Azure VM boot failures have four distinct root causes that require completely
different resolution paths. Running through a linear procedure wastes time because
you apply steps for root cause A when the problem is actually root cause B.
The decision tree routes you to the correct path in under 5 minutes.

---

## The Decision Tree

```
VM shows as Failed or cannot start
                │
                ▼
        Check Resource Health
        (Portal → VM → Help → Resource Health)
                │
    ┌───────────┴────────────┐
    │ Platform-initiated     │ User-initiated /
    │ unavailability         │ Unknown
    ▼                        ▼
 Azure platform issue    Check Activity Log for error code
 Wait and monitor.       (Portal → VM → Activity Log → Error entry)
 Open Azure support              │
 if > 30 minutes.        ┌───────┴────────────────────────────┐
                         │                                    │
                    AllocationFailed                   No error code /
                         │                             VM shows Running
                         ▼                             but unreachable
                  Stop (Deallocate)                          │
                  then Start                         Check Boot Diagnostics
                  (moves to new                      Screenshot + Serial Console
                   host cluster)                            │
                         │                    ┌─────────────┴──────────────┐
                    Still fails?              │                            │
                         │                OS healthy                BSOD / Boot loop /
                    Try different          (login screen)            Black screen
                    VM size or             Check NSG + JIT           │
                    region                 access port 3389          Escalate to L2
                                           or 22                     (disk repair,
                                                                      snapshot restore)
```

---

## Path A - AllocationFailed

```bash
# Stop (Deallocate) — this is different from Restart
# Restart keeps the VM on the same host cluster
# Deallocate releases it entirely — next Start finds a new cluster

az vm deallocate --resource-group rg-prod --name vm-app-01
# Wait 2-3 minutes for full deallocation

az vm start --resource-group rg-prod --name vm-app-01

# Verify state
az vm show --resource-group rg-prod --name vm-app-01 \
    --query "{Name:name, State:powerState}" -o table
```

If this still returns AllocationFailed, the capacity issue affects the entire
availability zone or region for this VM size:

```bash
# Option 1: Try a different VM size (compatible with the existing disk)
az vm resize --resource-group rg-prod --name vm-app-01 --size Standard_D2s_v5

# Option 2: Check available VM sizes in your region
az vm list-skus --location uksouth \
    --query "[?name=='Standard_D4s_v5'].{Name:name, Available:locationInfo[0].zones}" \
    -o table
```

---


