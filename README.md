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


**What is unique about these articles:**
M365-001 documents the exact Autodiscover lookup order and why SCP (Active
Directory Service Connection Point) causes hybrid migrations to break for
domain-joined machines even after mailboxes move to the cloud. M365-002 provides
the Call Analytics portal path and the exact Microsoft quality thresholds for
packet loss, jitter, and latency - plus the VPN split tunnelling tracert test
that identifies hairpinning in under 2 minutes. M365-003 explains inheritance
breaking as the root cause of 80% of SharePoint access tickets and provides PnP
PowerShell commands that show exactly where inheritance broke and why. M365-004
is explicit that password reset and MFA reset do not fix CA blocks and documents
exactly which grant control failed and what the correct L1 path is for each one.

---

### Azure (4 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| AZ-001 | [Azure VM Boot Failure: Complete Decision Tree](azure/az-001-vm-boot-failure-decision-tree.md) | AZ-104 |
| AZ-002 | [Entra ID Token Expiry and App Consent Failures](azure/az-002-entra-id-token-and-consent-issues.md) | AZ-104, Sec+ |
| AZ-003 | [Azure Backup Job Failure: Diagnosis and Recovery](azure/az-003-azure-backup-job-failure-guide.md) | AZ-104 |
| AZ-004 | [Azure Cost Spike Investigation Checklist](azure/az-004-cost-spike-investigation-checklist.md) | AZ-104, AZ-900 |


**What is unique about these articles:**
AZ-001 uses a decision tree instead of a linear procedure because VM boot failures
have four distinct root causes requiring completely different paths - a linear
procedure wastes time applying steps for the wrong cause. AZ-002 documents the
full token type taxonomy and lifetimes, explaining why users get prompted to
re-authenticate (refresh token expiry) vs why applications break silently
(access token expiry handled automatically). AZ-003 maps every common Azure Backup
error code to its cause and fix, including the VSS writer state check that most
backup troubleshooting guides omit. AZ-004 classifies cost spikes into four
categories before investigation begins, reducing investigation time by routing
directly to the relevant Azure CLI commands.

---

### Networking (3 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| NET-001 | [802.1X Authentication Failures: Wired and Wireless](networking/net-001-802-1x-authentication-failure.md) | Net+, Sec+ |
| NET-002 | [Asymmetric Routing: Diagnosis and Impact](networking/net-002-asymmetric-routing-diagnosis.md) | Net+ |
| NET-003 | [DHCP Scope Exhaustion: Diagnosis and Emergency Response](networking/net-003-dhcp-exhaustion-and-scope-management.md) | Net+, A+ |

**What is unique about these articles:**
NET-001 explains why 802.1X failures are invisible (the device connects physically
but has no IP) and documents all NPS Event 6273 reason codes - the information
that turns a 45-minute investigation into a 5-minute lookup. NET-002 covers
asymmetric routing as an Azure hybrid environment problem, including the UDR
diagnostic commands and why a missing UDR on the return-path subnet is the most
common cause in Azure deployments specifically. NET-003 distinguishes DHCP
exhaustion from DHCP server failure before any investigation, preventing the
wrong diagnostic path.

---

### Security (3 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| SEC-001 | [Ransomware: Initial Triage for L1](security/sec-001-ransomware-initial-triage.md) | Sec+ |
| SEC-002 | [Insider Threat: Behavioural and Technical Indicators](security/sec-002-insider-threat-indicators.md) | Sec+ |
| SEC-003 | [SSL/TLS Certificate Chain Errors in Production](security/sec-003-ssl-tls-certificate-chain-errors.md) | Sec+, AZ-104 |

**What is unique about these articles:**
SEC-001 opens with the three rules before any other action - specifically Rule 1
(do not shut down) and the technical reasons why shutting down destroys forensic
evidence and can cause permanent data loss in some ransomware strains. SEC-002
exists in an L1 knowledge base because insider threats are first noticed by L1
engineers, not by automated alerts - this article defines exactly what to observe,
document, and escalate without making the mistake of confronting the subject.
SEC-003 explains why the same certificate works in Chrome but fails in curl -
the browser certificate cache distinction that makes chain errors so confusing.

---

### Linux (3 articles)

| Article ID | Title | Cert Alignment |
|-----------|-------|---------------|
| LNX-001 | [Filesystem Corruption: Recovery Without Data Loss](linux/lnx-001-filesystem-corruption-recovery.md) | A+ |
| LNX-002 | [systemd Service Dependency Failures](linux/lnx-002-systemd-service-dependency-failures.md) | A+ |
| LNX-003 | [High iowait Diagnosis: Finding What Is Hammering the Disk](linux/lnx-003-high-iowait-diagnosis.md) | A+, Net+ |

**What is unique about these articles:**
LNX-001 opens with the critical warning to check SMART data before running fsck -
because running fsck on a failing disk can cause more data loss than the original
corruption. It also covers btrfs corruption as an edge case specifically because
running ext4 fsck on btrfs makes things dramatically worse, which is not obvious.
LNX-002 documents the difference between the four dependency types (Requires,
Wants, After, BindsTo) and the `network.target` vs `network-online.target`
distinction that causes "works manually, fails on boot" issues - the most common
and least understood systemd problem. LNX-003 explains what iowait actually
measures (CPU time wasted waiting for disk, not disk utilisation) and provides
the iotop + pidstat + lsof combination for identifying exactly which process is
responsible.

---

### Templates (3 templates)

| Template | Purpose |
|---------|---------|
| [knowledge-article-template.md](templates/knowledge-article-template.md) | Template for all new knowledge articles with inline instructions for every section |
| [known-error-record-template.md](templates/known-error-record-template.md) | Template for ITIL 4 Known Error Records - problems with identified root cause and no permanent fix |
| [post-incident-review-template.md](templates/post-incident-review-template.md) | Full PIR template with timeline table, root cause analysis, contributing factors, response assessment, action items, and knowledge base update triggers |

---

### Meta (2 documents)

| Document | Purpose |
|---------|---------|
| [how-to-contribute.md](meta/how-to-contribute.md) | Step-by-step contribution guide including article ID naming convention, branch workflow, pull request description requirements, and what not to contribute |
| [article-quality-standards.md](meta/article-quality-standards.md) | Eight quality standards every article must meet before publication, with failing and passing examples for each standard |

---

## Certification Coverage

| Cert Domain | Articles Covering It |
|-------------|-------------------|
| CompTIA A+ - Operating Systems | WIN-001, WIN-002, WIN-003, WIN-005, LNX-001, LNX-002, LNX-003 |
| CompTIA A+ - Networking | NET-003, LNX-003 |
| CompTIA A+ - Security | WIN-003, WIN-004, SEC-001 |
| CompTIA Network+ - Infrastructure | NET-001, NET-002, NET-003 |
| CompTIA Network+ - Network Operations | NET-002, M365-002, LNX-003 |
| CompTIA Network+ - Security | NET-001, SEC-003 |
| CompTIA Security+ - Threats and Vulnerabilities | SEC-001, SEC-002 |
| CompTIA Security+ - Identity and Access Management | M365-004, AZ-002 |
| CompTIA Security+ - PKI and Cryptography | WIN-004, SEC-003, WIN-003 |
| Microsoft AZ-900 - Cloud Concepts | M365-001, M365-002, M365-003, AZ-004 |
| Microsoft AZ-104 - Identity | M365-004, AZ-002 |
| Microsoft AZ-104 - Compute | AZ-001 |
| Microsoft AZ-104 - Storage | AZ-003, AZ-004 |
| Microsoft AZ-104 - Monitoring | AZ-003, AZ-004 |
| ITIL 4 - Incident Management | All articles (P1/P2/P3 classification, escalation packages) |
| ITIL 4 - Problem Management | templates/known-error-record-template.md |
| ITIL 4 - Knowledge Management | templates/knowledge-article-template.md, meta/ |
| ITIL 4 - Continual Improvement | templates/post-incident-review-template.md |

---

## Role Coverage

| Role | Most Relevant Articles |
|------|----------------------|
| IT Support L1/L2 | WIN-001–005, NET-003, SEC-001, SEC-002, M365-001–004 |
| NOC Engineer | NET-001, NET-002, LNX-003, AZ-001, AZ-003 |
| SOC Analyst | SEC-001, SEC-002, SEC-003, M365-004, AZ-002 |
| Azure Cloud Support Engineer | AZ-001–004, M365-004, SEC-003 |
| Systems Administrator | WIN-002–005, LNX-001–003, NET-001 |
| MSP Engineer | All articles — breadth across all domains is the MSP requirement |
| Junior Cloud Administrator | AZ-001–004, M365-001–004, SEC-003 |

---




