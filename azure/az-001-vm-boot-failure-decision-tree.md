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

## Path B - Boot Diagnostics Shows OS-Level Issue

```bash
# Pull the Boot Diagnostics screenshot via CLI
az vm boot-diagnostics get-boot-log \
    --resource-group rg-prod \
    --name vm-app-01

# Or view the screenshot in the portal:
# VM → Help → Boot Diagnostics → Screenshot

# Access Serial Console (OS-level terminal without network)
# Portal → VM → Help → Serial Console
# This works even when the VM is not reachable via RDP/SSH
```
**Screenshot interpretation:**

| Screenshot Content | Diagnosis | L1 Action |
|-------------------|-----------|-----------|
| Windows login screen or desktop | OS healthy — issue is network/firewall | Check NSG, check Bastion, check JIT |
| "Preparing Automatic Repair" | Windows Update issue or filesystem | Escalate to L2 |
| Blue screen with stop code | Driver or kernel issue | Note stop code, escalate |
| `chkdsk` running automatically | Filesystem corruption detected | Let it complete, monitor (may take 30–60 min) |
| GRUB menu (Linux) | GRUB misconfigured or kernel issue | Escalate to L2 |
| Black screen, no cursor | VM stuck in initialisation | Stop + Start (not Restart) |

---

## Path C - VM Running But Unreachable (RDP/SSH Fails)

```bash
# Step 1: IP Flow Verify — is the NSG blocking the port?
az network watcher test-ip-flow \
    --resource-group rg-prod \
    --vm vm-app-01 \
    --direction Inbound \
    --protocol TCP \
    --local 10.1.1.4:3389 \
    --remote [your-ip]:12345

# If Access = Deny: NSG rule is blocking — check and update NSG rules

# Step 2: Is JIT active? Check if a JIT request is needed
az security jit-policy show \
    --name default \
    --resource-group rg-prod \
    --vm vm-app-01

# Step 3: Try Serial Console for direct OS access
# Portal → VM → Help → Serial Console
# In Windows SAC:
#   SAC> cmd
#   SAC> ch -si 1
#   C:\> net start TermService   (restart RDP service)
#   C:\> netsh advfirewall show allprofiles state
```


---



