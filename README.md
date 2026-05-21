# it-support-knowledge-base

A structured, production-quality IT support knowledge base covering Windows
advanced troubleshooting, Microsoft 365 diagnosis, Azure support procedures,
networking edge cases, security incident triage, and Linux administration. Built
the way a real MSP or in-house IT team would actually build one - institutional
knowledge that turns one-off troubleshooting wins into repeatable outcomes.

Every article was written as if a capable but new engineer needs to resolve
the issue at 2am with no one to ask. Precise enough to follow without guesswork.
Contextual enough to understand why each step works. Honest about the edge
cases the official documentation does not mention.

---

## What makes this different from a documentation summary

Most "knowledge bases" are documentation summaries - paraphrased vendor docs
with screenshots. This is not that.

Every article here starts from a real ticket type that gets mishandled because
the symptom is ambiguous, the standard procedure breaks in a specific context,
or the information needed to resolve it quickly only exists in senior engineers'
heads. The articles capture what takes months of hands-on experience to learn:
which problem looks like which other problem, when the standard procedure
does not apply and why, and what the output of a diagnostic command actually
means when it does not match the healthy baseline.

---

## Repository Contents

### Windows (5 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| WIN-001 | [User Profile Corruption: Diagnosis and Recovery](windows/win-001-profile-corruption-diagnosis.md) | A+ |
| WIN-002 | [WMI Repository Corruption: Diagnosis and Rebuild](windows/win-002-wmi-repository-rebuild.md) | A+ |
| WIN-003 | [BitLocker Recovery: Retrieval, Validation, and Post-Recovery](windows/win-003-bitlocker-recovery-procedure.md) | A+, Sec+ |
| WIN-004 | [Windows Certificate Store: Diagnosis and Repair](windows/win-004-certificate-store-troubleshooting.md) | A+, Sec+ |
| WIN-005 | [Shadow Copy and VSS: Recovery Procedures and Failures](windows/win-005-shadow-copy-and-vss-recovery.md) | A+ |


**What is unique about these articles:**
WIN-001 documents the `.bak` registry key duplicate - the most common profile
corruption pattern - and why copying the entire profile perpetuates corruption.
WIN-002 documents the critical distinction between WMI permission failures (DCOM)
and WMI repository corruption, because they present identically but require
completely different fixes. WIN-003 covers why recovery mode is not a sign of
attack and documents every common legitimate trigger with the specific TPM PCR
event IDs. WIN-004 covers private key permission failures - the least documented
certificate issue. WIN-005 is explicit about what shadow copies cannot protect
against, including the specific ransomware command that deletes them.

---

### Microsoft 365 (4 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| M365-001 | [Autodiscover Failure: Why Outlook Cannot Find the Mailbox](microsoft-365/m365-001-autodiscover-failure-diagnosis.md) | A+, AZ-900 |
| M365-002 | [Teams Media Quality Investigation](microsoft-365/m365-002-teams-media-quality-investigation.md) | Net+, AZ-900 |
| M365-003 | [SharePoint Permission Inheritance Breaks](microsoft-365/m365-003-sharepoint-permission-inheritance.md) | AZ-900 |
| M365-004 | [Conditional Access Sign-In Failure Diagnosis](microsoft-365/m365-004-conditional-access-sign-in-failure.md) | Sec+, AZ-104 |


