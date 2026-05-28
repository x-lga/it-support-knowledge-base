# NET-003 - DHCP Scope Exhaustion: Diagnosis and Emergency Response

**Article ID:** NET-003
**Category:** Networking - DHCP
**Severity:** P1 (new devices cannot get IPs — service impact)
**Cert alignment:** CompTIA Network+, CompTIA A+
**Last verified:** 2026-07

---

## DHCP Exhaustion vs DHCP Failure

These are different issues with similar symptoms. Distinguish quickly:

| Test | DHCP Failure (server down) | DHCP Exhaustion (scope full) |
|------|--------------------------|----------------------------|
| Existing device can renew lease? | No - DHCP server unreachable | Yes - renewal extends an existing lease |
| New device gets an IP? | No | No - pool is full |
| DHCP server shows in Services? | No | Yes - service is running |
| Leases report shows available? | N/A | 0 available |

---

## Step 1 - Check Scope Utilisation

```powershell
# On the DHCP server
Import-Module DHCPServer

# Check all scopes and their utilisation
Get-DhcpServerv4Scope | ForEach-Object {
    $Stats = Get-DhcpServerv4ScopeStatistics -ScopeId $_.ScopeId
    [PSCustomObject]@{
        ScopeId     = $_.ScopeId
        Name        = $_.Name
        Total       = $Stats.TotalAddresses
        InUse       = $Stats.AddressesInUse
        Available   = $Stats.AddressesFree
        PctUsed     = [math]::Round(($Stats.AddressesInUse / $Stats.TotalAddresses) * 100, 1)
    }
} | Format-Table -AutoSize

# Critical: Any scope at 90%+ needs immediate attention
# At 100%: new devices cannot join the network
```

---
