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
