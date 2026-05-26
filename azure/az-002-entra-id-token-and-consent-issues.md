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



