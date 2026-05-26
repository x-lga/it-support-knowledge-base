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
