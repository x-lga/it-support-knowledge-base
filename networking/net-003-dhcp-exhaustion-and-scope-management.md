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

## Step 2 - Emergency Response - Free Up Leases

```powershell
# Find stale leases (devices not seen recently)
$Scope    = "10.10.10.0"
$StaleDay = (Get-Date).AddDays(-7)

# List leases where the device has not communicated recently
$StaleLeases = Get-DhcpServerv4Lease -ScopeId $Scope |
    Where-Object {
        $_.LeaseExpiryTime -gt (Get-Date) -and   # Not expired
        $_.HostName -eq ""                        # No hostname resolved = likely stale
    }

Write-Host "Potentially stale leases: $($StaleLeases.Count)"
$StaleLeases | Select-Object IPAddress, ClientId, LeaseExpiryTime | Format-Table

# Remove a specific stale lease to free the IP
# WARNING: Only remove leases for devices confirmed offline
Remove-DhcpServerv4Lease -ScopeId $Scope -ClientId "aa-bb-cc-dd-ee-ff"
```

---

## Step 3 - Expand the Scope (Requires Change Authorisation)

Expanding a DHCP scope is a network change and requires a Change ticket:

```powershell
# Current scope: 10.10.10.100 – 10.10.10.200 (100 addresses)
# Expand to: 10.10.10.100 – 10.10.10.250 (150 addresses)
# PREREQUISITE: The expanded range must be within the subnet and not in use as statics

# Check no devices are using the IPs in the new range first
$NewRangeStart = "10.10.10.201"
$NewRangeEnd   = "10.10.10.250"
# Ping sweep to verify range is empty:
201..250 | ForEach-Object {
    $IP = "10.10.10.$_"
    if (Test-Connection $IP -Count 1 -Quiet -TimeoutSeconds 1) {
        Write-Host "$IP is RESPONDING — do NOT include in DHCP range" -ForegroundColor Red
    }
}

# Expand the scope
Set-DhcpServerv4Scope `
    -ScopeId      "10.10.10.0" `
    -EndRange     "10.10.10.250"

# Verify
Get-DhcpServerv4Scope -ScopeId "10.10.10.0" | Select-Object StartRange, EndRange
```


---


