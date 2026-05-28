# NET-001 - 802.1X Authentication Failures: Wired and Wireless

**Article ID:** NET-001
**Category:** Networking - Network Access Control
**Severity:** P2 (user cannot connect to network) | P1 (multiple users blocked)
**Cert alignment:** CompTIA Network+, CompTIA Security+
**Last verified:** 2026-07

---

## What 802.1X Is and Why Its Failures Are Invisible

802.1X is the standard for port-based network access control. It requires
devices to authenticate before network access is granted. A switch or wireless
access point acts as the authenticator — the device (supplicant) must prove
its identity to a RADIUS server before the port is opened.

Failures are invisible to the user: the device appears to connect physically
(link lights are on, Wi-Fi association succeeds) but has no IP address or
limited connectivity. The user sees "No internet access" rather than an
authentication error — because 802.1X failure typically results in the port
being placed in a guest VLAN or dropped entirely.

---

## Step 1 - Identify Whether 802.1X Is the Cause

```powershell
# On the affected Windows machine:
# Check if 802.1X authentication is failing (not just missing IP)
Get-EventLog -LogName System -Source "Microsoft-Windows-Wired-AutoConfig",
    "Microsoft-Windows-Wireless-AutoConfig" -Newest 20 |
    Select-Object TimeGenerated, Source, EventID, Message |
    Format-List

# Event IDs for 802.1X failure:
# 15513 — Authentication failed (generic)
# 15514 — Supplicant authentication timed out
# 15519 — RADIUS server unreachable
# 15508 — Certificate validation failed (machine certificate)

# Check what VLAN the machine is on (if placed in guest VLAN, access is restricted)
Get-NetIPConfiguration | Select-Object InterfaceAlias, IPv4Address, IPv4DefaultGateway
# If IP starts with 169.254.x.x: DHCP failed — port likely blocked
# If IP is in a restricted range (e.g., 10.99.x.x when corporate is 10.10.x.x): guest VLAN
```

---

## Step 2 - Common Failure Scenarios and Resolution

**Scenario A - Machine certificate expired or missing:**
802.1X in enterprise environments typically uses machine certificates for
authentication. If the certificate expired or was not renewed (due to offline
machine or certificate policy issue):

```powershell
# Check the machine's computer certificate store for a valid certificate
Get-ChildItem -Path "Cert:\LocalMachine\My" |
    Where-Object { $_.EnhancedKeyUsageList.FriendlyName -contains "Client Authentication" } |
    Select-Object Subject, NotAfter, Thumbprint

# A missing or expired certificate here = 802.1X will fail
# Resolution: re-enroll the certificate via GPO or certlm.msc → Enroll
# If offline: temporarily bypass 802.1X (connect via a non-802.1X port for enrollment)
```

**Scenario B - NPS/RADIUS server unreachable:**
```powershell
# Test reachability of RADIUS server(s)
Test-NetConnection -ComputerName "nps01.contoso.local" -Port 1812   # RADIUS auth
Test-NetConnection -ComputerName "nps01.contoso.local" -Port 1813   # RADIUS accounting

# If unreachable: check NPS service on the server
# Get-Service -Name "IAS" -ComputerName nps01 | Select-Object Status
```


