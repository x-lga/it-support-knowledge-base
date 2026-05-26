# AZ-004 - Azure Cost Spike Investigation Checklist

**Article ID:** AZ-004
**Category:** Azure - Cost Management and Governance
**Severity:** P3 (unexpected spend) | P2 (significant budget breach)
**Cert alignment:** AZ-104, AZ-900
**Last verified:** 2026-07

---

## The Four Categories of Azure Cost Spikes

Every Azure cost spike falls into one of these categories. Identifying which
one immediately before running the full investigation:

1. **Resource left running** - a VM, SQL MI, or App Service that should have
   been stopped or deleted was left in a running/allocated state
2. **New resource unexpectedly deployed** - automation, a developer, or a
   misconfigured service created resources without oversight
3. **Data transfer or egress** - unexpected data movement (outbound bandwidth,
   cross-region, or storage operations) that accumulates invisibly
4. **Service tier escalation** - a resource was resized, a reservation expired
   (reverted to PAYG), or a tier was changed inadvertently

---

## Checklist: Investigate a Cost Spike in Under 20 Minutes

### Step 1 - Open Cost Analysis (2 minutes)

```
Azure Portal → Cost Management + Billing → Cost Analysis

Set:
  Scope     : Subscription or resource group under investigation
  Date range: Current month vs last month
  Granularity: Daily
  Group by  : Service name first, then Resource

Look for:
  Which day did the spike begin? (Narrows the change window)
  Which service shows the largest increase?
  Is it a new charge line (new service) or an increase in an existing line?
```

---

### Step 2 - Check Activity Log for Changes on That Date (3 minutes)

```bash
# Find all resource creations on the spike date
az monitor activity-log list \
    --start-time "2026-07-10T00:00:00Z" \
    --end-time   "2026-07-11T00:00:00Z" \
    --query "[?status.value=='Succeeded' && contains(operationName.value, 'write')].{
        time:eventTimestamp,
        caller:caller,
        operation:operationName.localizedValue,
        resource:resourceId}" \
    --output table | head -50
```

---

### Step 3 - Category-Specific Investigation

**Category 1: Resource left running**
```bash
# List all running VMs and their SKUs
az vm list --query "[?powerState!='deallocated'].{Name:name,Size:hardwareProfile.vmSize,RG:resourceGroup}" -o table

# List all SQL MI instances
az sql mi list --query "[].{Name:name,Tier:sku.tier,vCores:vCores,RG:resourceGroup}" -o table

# Look for resources not in the expected list — something unexpected running
```

**Category 2: New resource deployed unexpectedly**
```bash
# List resources created in the last 7 days
az resource list \
    --query "[?createdTime>='2026-07-07'].{Name:name,Type:type,RG:resourceGroup,Created:createdTime}" \
    --output table
```

**Category 3: Data transfer or egress**
```
Azure Portal → Storage Account → [Account] → Monitoring → Metrics
  Metric: Egress
  Timeframe: Last 30 days, daily granularity
  Look for: Sudden spike in outbound data

  If spike is on a Blob account: check for unexpected large file reads
  If spike is on outbound bandwidth: check VM Network Out metrics
```


