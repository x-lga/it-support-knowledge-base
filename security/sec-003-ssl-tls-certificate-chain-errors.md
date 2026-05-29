# SEC-003 - SSL/TLS Certificate Chain Errors in Production

**Article ID:** SEC-003
**Category:** Security - PKI and TLS
**Severity:** P2 (service degraded for some clients) | P1 (service unavailable)
**Cert alignment:** CompTIA Security+, AZ-104
**Last verified:** 2026-07

---

## Why Certificate Chain Errors Affect Some Clients but Not Others

This is the most confusing aspect of certificate chain errors. A certificate
might work perfectly in Chrome on Windows but fail in curl on Linux, or work
on one server but fail on another. This happens because the trust store is
different on every system - what one OS considers a trusted root, another may
not have.

The second confusing behaviour: intermediary certificates are often not included
in the server's certificate configuration. Browsers cache intermediary certificates
from previous visits and OCSP stapling, so they often "just work." Curl, OpenSSL,
and automated tools do not have this cache - they require the full chain.

---

