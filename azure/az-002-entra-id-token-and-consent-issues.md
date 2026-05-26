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

