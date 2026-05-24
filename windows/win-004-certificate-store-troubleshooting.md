# WIN-004 - Windows Certificate Store: Diagnosis and Repair

**Article ID:** WIN-004
**Category:** Windows - PKI and Certificate Management
**Severity:** P2 (HTTPS applications failing, authentication broken)
**Cert alignment:** CompTIA A+, CompTIA Security+
**Last verified:** 2026-07

---

## Why Certificate Issues Are Frequently Misdiagnosed

Certificate problems manifest as generic application errors: "The connection
is not secure," "SSL handshake failed," "The credentials supplied to the
package were not recognised," or simply an application that crashes on startup.
Without understanding the Windows certificate store, these are treated as
network problems or application bugs.

Certificate store issues fall into four distinct categories, each with
a different resolution path:

| Category | Description | Common Symptom |
|----------|-------------|----------------|
| Missing root CA | The issuing root certificate is not in the Trusted Root store | "Certificate issued by untrusted authority" |
| Expired certificate | A certificate in the chain has passed its expiry date | "Certificate has expired" even for valid websites |
| Missing intermediate CA | Chain is incomplete - middle certificate absent | SSL handshake fails intermittently |
| Private key mismatch | Certificate exists but private key is missing or inaccessible | Authentication failure, code signing fails |

---

## Step 1 - Identify the Certificate Store Structure

```powershell
# View certificates in all major stores
# Personal store (machine certificates — used for machine auth, SSL)
Get-ChildItem -Path "Cert:\LocalMachine\My" |
    Select-Object Subject, Issuer, NotBefore, NotAfter, Thumbprint |
    Format-Table -AutoSize

# Trusted Root Certification Authorities
Get-ChildItem -Path "Cert:\LocalMachine\Root" |
    Select-Object Subject, Thumbprint, NotAfter |
    Format-Table -AutoSize

# Intermediate Certification Authorities
Get-ChildItem -Path "Cert:\LocalMachine\CA" |
    Select-Object Subject, Issuer, Thumbprint, NotAfter |
    Format-Table -AutoSize

# Current user personal certificates (for per-user auth like smart cards)
Get-ChildItem -Path "Cert:\CurrentUser\My" |
    Select-Object Subject, NotAfter, Thumbprint |
    Format-Table -AutoSize
```

---

