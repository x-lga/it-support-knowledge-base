# M365-004 - Conditional Access: Diagnosing Sign-In Failures

**Article ID:** M365-004
**Category:** Microsoft 365 - Entra ID Security
**Severity:** P2 (user blocked from critical resource) | P1 (multiple users blocked)
**Cert alignment:** CompTIA Security+, AZ-104
**Last verified:** 2026-07

---

## Why CA Failures Are Handled Incorrectly at L1

Conditional Access failures have result code 53003 in the Entra ID sign-in log.
The most common L1 mistake is treating this as an account issue and attempting a
password reset or MFA reset. Neither helps - the issue is a policy evaluation
result, not an authentication failure. The user's credentials are correct; the
device, location, or session does not meet the policy requirements.

The second mistake is disabling the CA policy to "fix" the issue. This removes
a security control, potentially violating compliance requirements, and the
action is irreversible from L1 scope (re-enabling requires knowing the original
configuration).

The correct L1 action is: identify which policy blocked the sign-in and why,
determine whether the user should legitimately have access from this context,
and either guide the user to a compliant path or escalate with complete information.

---

## Step 1 - Read the Sign-In Log

```
Azure Portal → Entra ID → Monitoring → Sign-in Logs →
  Filter:
    User    : [username]
    Status  : Failure
    Date    : Last 24 hours

Click the failed entry. Key fields:

Basic Info tab:
  Status         : Failure
  Failure reason : "Access has been blocked by Conditional Access policies"
  Error code     : 53003 (CA block) or 50097 (MFA required) or 70011 (app scope)

Conditional Access tab:
  Policy name    : [Which policy applied]
  Result         : Success / Failure / Not Applied / Report-only

  For each policy listed:
  If Result = Failure → THIS is the blocking policy
  Expand it to see which grant control failed:
    - Require MFA (user did not complete MFA)
    - Require compliant device (device not marked Compliant in Intune)
    - Require Entra ID joined device (device not joined)
    - Require approved client app (sign-in from unapproved application)
    - Require terms of use acceptance (user has not accepted ToU)
```

---

## Step 2 - Match the Block Reason to the Resolution

| CA Grant Control That Failed | L1 Resolution Path |
|-----------------------------|-------------------|
| Require MFA - not satisfied | Guide user through MFA setup (Authenticator app). If user lost their phone: follow MFA reset procedure, verify identity first. |
| Require compliant device | Check Intune compliance status for this device. If device is Non-Compliant: identify which setting is failing (Settings → Accounts → Access work or school → Info → Sync, then Intune portal → Device). |
| Require Entra ID joined device | The device is not joined to Entra ID. Options: join the device (Settings → Accounts → Access work or school → Connect) or request an exemption from IT admin. |
| Require approved client app | User is attempting to access from a browser or app not on the approved list. Guide them to use the Microsoft 365 apps or Outlook mobile. |
| Compliant network location | Sign-in is from an IP not in the Named Locations list. Common cause: user on a new VPN exit node or working from a coffee shop. Escalate to L2 to add location or create exemption. |
| User risk too high | Entra ID Identity Protection has flagged this account as high risk. This requires L2 Security involvement - do NOT attempt to clear the risk flag at L1. |

---

## Step 3 - Escalation Package for CA Blocks

When you cannot resolve at L1 (device join required, location not permitted,
user risk flag), provide L2 with:

```
CA BLOCK ESCALATION
══════════════════════════════════════════════════
User UPN      :
Sign-in time  :
Application   : (what were they trying to access)
Device name   : (from sign-in log)
Device ID     : (from sign-in log — GUID)
Client IP     : (from sign-in log)
──────────────────────────────────────────────────
Blocking policy    : (name of the CA policy)
Failed grant control: (which control failed)
User risk level    : (Low / Medium / High — from sign-in log)
Device compliance  : (Compliant / Not Compliant / Unknown — from Intune)
──────────────────────────────────────────────────
Business justification:
  [Why does this user need access from this device/location?]
  [Is this a new device? Travelling? Remote working from new location?]
══════════════════════════════════════════════════
```


---

