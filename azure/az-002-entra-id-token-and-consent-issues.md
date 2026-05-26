# AZ-002 - Entra ID: Token Expiry and App Consent Failures

**Article ID:** AZ-002
**Category:** Azure - Entra ID / Application Identity
**Severity:** P2 (application authentication failing)
**Cert alignment:** AZ-104, CompTIA Security+
**Last verified:** 2026-07

---

## Token and Consent Issues - Why They Are Confusing

Entra ID authentication involves multiple types of tokens with different
lifetimes, and application consent involves both admin consent and user consent
that can interact in unexpected ways. These issues tend to generate support tickets
where the symptom is vague ("the app stopped working" or "suddenly getting
login prompts again") and the cause is invisible to the user.

Understanding the token landscape prevents both misdiagnosis and over-reaction
(revoking all sessions when only an access token needs refreshing).

**Token types and their default lifetimes:**

| Token Type | Default Lifetime | What It Does |
|-----------|-----------------|-------------|
| Access token | 1 hour | Authorises API calls - short-lived by design |
| Refresh token | 90 days (users), 24 hours (clients) | Used to get new access tokens silently |
| ID token | 1 hour | Contains user identity claims - used by the app to know who the user is |
| Session cookie (SSO) | Configurable - default "Until browser closed" | Browser-based SSO state |

**Why users suddenly get re-auth prompts:**
Usually because the refresh token expired (90-day window of inactivity) or
was explicitly revoked (Revoke Sessions action in Entra ID). The access token
expiring every hour is silent - the app uses the refresh token to get a new
access token automatically.

---

## Step 1 - Check the Sign-In Log for Token-Related Failures

```
Azure Portal → Entra ID → Sign-in Logs →
  Filter by application name and status = Failure

Key error codes for token/consent issues:
  AADSTS65001  — User has not consented to use the app
  AADSTS65004  — User declined to consent
  AADSTS70008  — Token request expired
  AADSTS70011  — Scope (permission) requested is invalid for this app
  AADSTS70043  — Refresh token has expired
  AADSTS700082 — Refresh token was revoked (admin ran Revoke Sessions)
  AADSTS90094  — Admin consent required (app needs admin-level permissions)
```

---

## Step 2 - Admin Consent for Enterprise Applications

When an application requires permissions that a user cannot self-consent to
(such as reading all users in the directory), admin consent is required:

```
Azure Portal → Entra ID → Enterprise Applications →
  [Application Name] → Permissions →
  Grant admin consent for [tenant name]

This grants all the permissions the app has requested to all users in the tenant.
After admin consent, users can sign in without individual consent prompts.
```

For applications where per-user consent is appropriate (not admin-wide):
```
Azure Portal → Entra ID → Enterprise Applications →
  [Application Name] → Properties →
  User assignment required: Yes → Assign individual users or groups
  (This prevents anyone in the tenant from signing in — only assigned users can)
```

---

## Step 3 - Diagnose Service Principal and App Registration Issues

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Application.Read.All", "User.Read.All"

# Check if the app registration exists and is enabled
$AppId = "your-application-client-id"
$App = Get-MgApplication -Filter "appId eq '$AppId'"

if (-not $App) {
    Write-Host "ERROR: Application registration not found for AppId: $AppId"
    Write-Host "The app may have been deleted or the wrong AppId is configured."
} else {
    Write-Host "App found: $($App.DisplayName)"
    Write-Host "App ID    : $($App.AppId)"
    Write-Host "Object ID : $($App.Id)"

    # Check the enterprise application (service principal) in the tenant
    $SP = Get-MgServicePrincipal -Filter "appId eq '$AppId'"
    if ($SP) {
        Write-Host "Service Principal: $($SP.DisplayName) — Enabled: $($SP.AccountEnabled)"
    } else {
        Write-Host "WARNING: No Service Principal found — the app has not been consented to in this tenant"
    }
}
```


---




