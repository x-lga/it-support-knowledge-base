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

## Step 1 - Test the Certificate Chain from Multiple Perspectives

```bash
# Test from a Linux machine (uses the OS trust store — different from Windows)
openssl s_client -connect banking.contoso.com:443 -showcerts

# The output shows:
# Certificate chain:
# 0 s:/CN=banking.contoso.com         (Server certificate)
#   i:/CN=Contoso Intermediate CA
# 1 s:/CN=Contoso Intermediate CA     (Intermediate — should be here)
#   i:/CN=Contoso Root CA
# 2 s:/CN=Contoso Root CA             (Root — sometimes omitted)
#   i:/CN=Contoso Root CA
#
# If only certificate 0 is shown: the server is not sending the full chain
# This works in browsers (they cache the intermediate) but fails in API clients

# Verify the chain validates correctly
openssl s_client -connect banking.contoso.com:443 </dev/null 2>&1 | \
    grep -E "Verify return code"
# "Verify return code: 0 (ok)" = valid chain
# "Verify return code: 21 (unable to verify the first certificate)" = broken chain
```

---

## Step 2 - Fix a Missing Intermediate Certificate

The most common chain error is a server not including the intermediate CA
certificate in its TLS configuration. The server presents only the leaf
certificate, leaving clients to find the intermediate themselves.

**For Azure Application Gateway:**
```bash
# Check current certificate configuration
az network application-gateway ssl-cert list \
    --gateway-name appgw-contoso-prod \
    --resource-group rg-networking-prod \
    --output table

# Update with a full chain certificate (PFX with full chain)
# The PFX must include: Leaf cert + Intermediate CA + Root CA (optional)
# Clients who cannot find intermediary certs will fail without this

az network application-gateway ssl-cert create \
    --gateway-name appgw-contoso-prod \
    --resource-group rg-networking-prod \
    --name banking-cert-fullchain \
    --cert-file ./banking-fullchain.pfx \
    --cert-password "[pfx-password]"
```


