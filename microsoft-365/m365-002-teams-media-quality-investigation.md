# M365-002 - Microsoft Teams: Media Quality Investigation

**Article ID:** M365-002
**Category:** Microsoft 365 - Teams
**Severity:** P3 (audio/video quality) | P2 (calls dropping entirely)
**Cert alignment:** CompTIA Network+, AZ-900
**Last verified:** 2026-07

---

## Why Teams Media Quality Issues Are Hard to Diagnose

Teams audio and video quality complaints are the most common "soft" M365 ticket
and the most frequently mishandled. The symptom ("call quality is bad") has
dozens of possible causes spanning the user's device, the local network, the
ISP, the corporate WAN, the Teams service, and the far-end participant's setup.

Without a structured approach, these tickets consume hours of back-and-forth
with no resolution. This article provides the tooling and methodology to identify
the actual cause in under 30 minutes.

**Key insight:** Microsoft Teams Real-Time Transport Protocol (RTP) media goes
peer-to-peer when possible, or via Microsoft transport relays. Audio and video
packets have strict latency requirements. Quality issues almost always trace
to one of: insufficient bandwidth, packet loss, high jitter, or routing that
adds latency (VPN hairpinning is a frequent offender).

---

## Step 1 - Pull the Call Analytics Report

Microsoft provides per-call quality data in the Teams Admin Centre:

```
Teams Admin Centre (admin.teams.microsoft.com) →
  Users → [User Name] → Meetings & Calls tab →
  Click on the specific meeting or call → Call Analytics

The report shows per-participant:
  Audio quality       : Good / Poor / No audio
  Video quality       : Good / Poor
  Screen share quality: Good / Poor
  Packet loss (%)     : Audio, Video
  Round-trip latency  : ms
  Jitter              : ms
  Connection type     : WiFi / Ethernet / Cellular
  IP addresses        : Internal and external
  Codec               : Audio and video codec used
```
**Quality thresholds (Microsoft definitions):**

| Metric | Good | Poor |
|--------|------|------|
| Audio packet loss | < 2.5% | > 2.5% |
| Round-trip latency | < 500ms | > 500ms |
| Jitter | < 30ms | > 30ms |
| Video packet loss | < 5.0% | > 5.0% |

---

## Step 2 - Identify the Problem Scope

The Call Analytics report shows data for ALL participants. Use this to scope:

| Pattern | Cause | L1 Action |
|---------|-------|-----------|
| Only the reporting user shows Poor metrics | Local device, local network, or local ISP | Investigate the user's endpoint and local network |
| All participants show Poor metrics | Meeting room equipment, central relay, or broad network issue | Check the meeting room network connection |
| One specific remote participant shows Poor | Their end has the problem | Advise the other participant to investigate their setup |
| Intermittent across multiple users at the same time | Corporate WAN or ISP event | Check network monitoring for the affected time window |

---
