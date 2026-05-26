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

