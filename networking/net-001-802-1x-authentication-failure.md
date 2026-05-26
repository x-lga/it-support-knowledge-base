# NET-001 - 802.1X Authentication Failures: Wired and Wireless

**Article ID:** NET-001
**Category:** Networking - Network Access Control
**Severity:** P2 (user cannot connect to network) | P1 (multiple users blocked)
**Cert alignment:** CompTIA Network+, CompTIA Security+
**Last verified:** 2026-07

---

## What 802.1X Is and Why Its Failures Are Invisible

802.1X is the standard for port-based network access control. It requires
devices to authenticate before network access is granted. A switch or wireless
access point acts as the authenticator — the device (supplicant) must prove
its identity to a RADIUS server before the port is opened.

Failures are invisible to the user: the device appears to connect physically
(link lights are on, Wi-Fi association succeeds) but has no IP address or
limited connectivity. The user sees "No internet access" rather than an
authentication error — because 802.1X failure typically results in the port
being placed in a guest VLAN or dropped entirely.

---

