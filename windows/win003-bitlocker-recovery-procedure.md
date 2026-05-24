# WIN-003 - BitLocker Recovery: Retrieval, Validation, and Post-Recovery Steps

**Article ID:** WIN-003
**Category:** Windows - Encryption and Security
**Severity:** P2 (user locked out of encrypted drive)
**Cert alignment:** CompTIA A+, CompTIA Security+
**Last verified:** 2026-07

---

## When BitLocker Recovery Mode Triggers

BitLocker enters recovery mode when it detects a condition that could indicate
tampering with the boot environment. Common legitimate triggers (not attacks):

| Trigger | Why It Happens | Frequency |
|---------|---------------|-----------|
| BIOS/UEFI firmware update | Changes measurements in TPM PCR registers | Very common - any firmware update |
| Boot order change | BIOS setting change alters TPM boot measurements | Common - any BIOS entry |
| Adding/removing RAM | Some UEFI implementations re-measure hardware config | Occasional |
| Hardware change (NIC, GPU) | PCR 2 measures hardware configuration | Occasional |
| Windows Update (certain updates) | Boot components are updated - TPM measurement changes | Common with feature updates |
| Enabling or disabling Secure Boot | Secure Boot state is measured by TPM | Less common |
| Entering BIOS setup (some systems) | BIOS entry triggers a recovery boot on next restart | Hardware-specific |
| Incorrect PIN entry (×5) | Lockout threshold reached | User-driven |

**What recovery mode is NOT:**
Recovery mode is not an indication the drive was attacked or data was stolen.
It is a pre-emptive lock that requires proof of authorisation before allowing
access. The recovery key IS the proof of authorisation.

---

