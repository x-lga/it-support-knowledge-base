# M365-001 - Autodiscover Failure: Why Outlook Cannot Find the Mailbox

**Article ID:** M365-001
**Category:** Microsoft 365 - Exchange / Outlook
**Severity:** P2 (user cannot configure Outlook or keeps losing connection)
**Cert alignment:** CompTIA A+, AZ-900
**Last verified:** 2026-07

---

## What Autodiscover Does and Why It Fails

Autodiscover is the mechanism Outlook uses to find Exchange server settings
(mailbox location, EWS endpoint, OAB location, certificate) automatically.
When it fails, Outlook either cannot be configured at all, or repeatedly prompts
for credentials, or loses connectivity after a period of time.

Autodiscover failures in Microsoft 365 hybrid environments are particularly common
because there are multiple valid Autodiscover sources and Outlook tries them in
a specific order - if the wrong one wins, the configuration is incorrect.

**Autodiscover lookup order (Outlook 2019 and later):**
1. SCP (Service Connection Point) lookup in Active Directory - for domain-joined machines
2. Root domain DNS lookup: `https://contoso.com/autodiscover/autodiscover.xml`
3. Autodiscover subdomain: `https://autodiscover.contoso.com/autodiscover/autodiscover.xml`
4. DNS SRV record: `_autodiscover._tcp.contoso.com`
5. HTTP redirect from local provider
6. Microsoft 365 cloud endpoint (Office 365 fallback)

In a hybrid environment, if step 1 (SCP) returns the on-premises Exchange server
but the user's mailbox has been migrated to M365, Outlook connects to the wrong
endpoint and authentication fails or the mailbox is not found.

---

## Step 1 - Run the Autodiscover Test

```powershell
# In Outlook: Hold Ctrl and right-click the Outlook icon in the system tray
# Select "Test Email AutoConfiguration"
# This shows exactly which Autodiscover source Outlook used and what it returned

# Alternatively, test from PowerShell using Microsoft's Remote Connectivity Analyzer
# Navigate to: https://testconnectivity.microsoft.com
# Select: Office 365 → Outlook Connectivity → Run the test
```

The test output shows:
- Which Autodiscover method succeeded
- The Exchange server URL returned
- Whether it points to on-premises or cloud
- Any certificate errors encountered

---

## Step 2 - Identify Whether SCP Is Pointing to the Wrong Server

```powershell
# Run on a domain-joined machine or on the DC
# This shows what Autodiscover SCP value AD is returning
Import-Module ActiveDirectory

# Find all Autodiscover SCP records in AD
Get-ADObject -Filter { servicePrincipalName -like "exchangeAB/*" } `
    -SearchBase (Get-ADRootDSE).configurationNamingContext `
    -Properties ServiceBindingInformation |
    Select-Object Name, ServiceBindingInformation

# In a fully M365-migrated environment, the SCP should point to
# https://autodiscover-s.outlook.com/autodiscover/autodiscover.xml
# If it still points to the old Exchange server, Outlook on domain-joined machines
# will always try the on-premises endpoint first
```

---

## Step 3 - Fix the SCP Record for M365-Only Environments

```powershell
# Update the SCP to point to Microsoft 365 Autodiscover
# Run on the Exchange server or with Exchange admin tools

# This requires Exchange Management Shell or the Exchange Admin Centre
# In Exchange Admin Centre:
# Hybrid → Hybrid Setup Wizard → Configure Hybrid → this updates SCP automatically

# Manual SCP update via ADSI:
# In adsiedit.msc:
# Navigate to: Configuration → Services → Microsoft Exchange → [Org] → Administrative Groups
#   → [Group] → Servers → [Server] → Protocols → Autodiscover
# Find the serviceBindingInformation attribute
# Change from: https://[on-prem-exchange]/autodiscover/autodiscover.xml
# Change to:   https://autodiscover-s.outlook.com/autodiscover/autodiscover.xml

# After changing SCP, force Outlook to re-run Autodiscover:
# Close Outlook → Delete Outlook profile → Recreate profile (Autodiscover will find M365)
```

---


