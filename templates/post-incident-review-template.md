# TEMPLATE - Post-Incident Review (PIR)

**What a Post-Incident Review is:**
A Post-Incident Review (also called a Post-Mortem or Major Incident Review) is
a structured retrospective conducted after every P1 incident and selected P2
incidents. Its purpose is to understand what happened, why it happened, whether
the response was effective, and what must change to prevent recurrence or improve
response next time.

The PIR is blameless. The goal is to improve systems and processes, not to assign
personal responsibility for mistakes. Engineers involved in the incident should
be able to speak honestly about what they did and what they missed without fear
of punishment. A culture where engineers hide mistakes to avoid blame produces
fewer PIRs and worse systems.


**When to conduct a PIR:**
- All P1 incidents - mandatory
- P2 incidents where the resolution took longer than the RTO
- P2 incidents that recurred within 30 days
- Any incident where data was lost or there is a risk of regulatory exposure
- Any incident where the workaround or fix was unclear or took significant investigation

**Timeline:** PIR should be completed within 5 business days of the incident closure.

---

# PIR - [Brief Incident Title]

**PIR ID:** [PIR-YYYYMMDD-NNN]
**Incident ID:** [INC-YYYYMMDD-NNN]
**Date of incident:** [YYYY-MM-DD HH:MM]
**Date PIR completed:** [YYYY-MM-DD]
**PIR facilitator:** [Name - should be someone not directly involved in the incident]
**Attendees:** [List all participants - include all engineers who worked the incident]

---

## Incident Summary

**One-sentence description:**
[What happened, in plain language. Written for an audience that was not involved.]

**Impact:**
[Who and what was affected. Number of users, services down, data at risk, financial
or reputational impact. Duration of impact from first detection to full restoration.]

**Severity at time of incident:** [P1 / P2]
**Duration of impact:** [HH:MM from first detection to service restoration]
**Was SLA/RTO met?** [Yes / No - if No: by how much was it missed?]

---

## Timeline

[A precise, time-ordered account of events. Include all significant actions —
detection, escalations, decisions, failed attempts, successful resolution steps,
communications sent. Times should be in UTC or a consistently stated timezone.]

| Time (UTC) | Event |
|-----------|-------|
| HH:MM | [First indication of the problem — monitoring alert, user report, etc.] |
| HH:MM | [First engineer notified / incident declared] |
| HH:MM | [Initial diagnosis attempted] |
| HH:MM | [Escalation to L2 if applicable] |
| HH:MM | [Root cause identified] |
| HH:MM | [Fix or workaround applied] |
| HH:MM | [Service restored] |
| HH:MM | [Incident closed] |
| HH:MM | [User / stakeholder notification sent] |

**Total time from first detection to resolution:** [HH:MM]
**Time from root cause identification to resolution:** [HH:MM]
**Escalation lag (time from impact start to L2 notification):** [HH:MM]

---

## Root Cause Analysis

**Root cause:**
[The technical or process cause of the incident. Be specific - "a configuration
change" is not a root cause. "A firewall rule added during change CHG-2026-0714
blocked port 1433 outbound from the application subnet, preventing the application
from reaching the SQL server" is a root cause.]



