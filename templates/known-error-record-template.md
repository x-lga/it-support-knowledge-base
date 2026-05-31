# TEMPLATE - Known Error Record (KER)

**What a Known Error Record is:**
A Known Error Record documents a problem for which the root cause has been
identified but a permanent fix has not yet been implemented or is not planned.
It is different from a knowledge article: a knowledge article describes a
resolved problem with a working fix. A Known Error Record describes an unresolved
or partially-resolved problem where a workaround exists.

In ITIL 4, Known Error Records live in the Known Error Database (KEDB) and are
referenced during Incident Management when the same issue recurs — the engineer
picks up the workaround from the KER and applies it without re-investigating,
reducing resolution time.


**When to create a Known Error Record:**
- The root cause of a recurring incident has been identified
- A permanent fix exists but cannot be implemented yet (awaiting change window,
  vendor patch, budget approval, or risk sign-off)
- No permanent fix is available (vendor limitation, end-of-life component, by design)
- The workaround reliably restores service but does not prevent recurrence

---

# [KER-ID] - Known Error Record: [Brief Problem Title]

**KER ID:** [KER-YYYYMMDD-NNN - e.g., KER-20260715-001]
**Created:** [YYYY-MM-DD]
**Created by:** [Name]
**Last updated:** [YYYY-MM-DD]
**Status:** [Open | Workaround Available | Permanent Fix Pending | Closed]

---

## Problem Summary

**One-sentence description:**
[Describe the problem in plain language. This is what engineers will search for
when they encounter this issue. Write it the way the symptom presents, not the
way the root cause is described - engineers search by symptom, not root cause.]

**Affected systems:**
[List the specific systems, services, or components affected. Include version
numbers where relevant - the same issue may affect v1.x but not v2.x.]

**Impact when triggered:**
[Describe the business or operational impact. What does the user experience?
What service is degraded or unavailable? How long does the impact last before
the workaround can be applied?]

---

## Problem History

**First occurrence:** [Date and incident ID if available]
**Recurrence frequency:** [How often this typically reoccurs - daily, weekly, monthly, irregular]
**Total occurrences to date:** [Number]
**Incident IDs:** [List related incident IDs for pattern analysis]

---

## Root Cause

**Root cause identified:** [Yes / Partially / No]
**Root cause description:**
[Describe the technical root cause in as much detail as is known. If partially
identified, describe what is known and what is still unclear.]

**Why a permanent fix has not been implemented:**
[One of: Vendor patch pending | Change window not yet available | No vendor fix exists
| Risk of fix exceeds risk of recurrence | Fix cost not yet approved | By design (accepted risk)]


**Vendor reference / bug ID:** [Vendor case number, public CVE, or bug tracker ID if applicable]

---

## Workaround

**Workaround available:** [Yes / No / Partial]
**Workaround restores full service:** [Yes / No - if No, describe what is not restored]
**Workaround duration:** [How long the workaround holds before the issue recurs]

**Workaround procedure:**

[Write the workaround steps in the same level of detail as a knowledge article.
Do not refer to another document - duplicate the steps here so the KER is
self-contained. Engineers pick up a KER during an active incident and do not
have time to navigate to other documents.]

```[language]
[Workaround commands or steps]
```

**Post-workaround verification:**
[What to check to confirm the workaround was successful.]

---

## Permanent Fix Plan

**Fix planned:** [Yes / No / Under evaluation]
**Fix description:** [What the permanent fix will be - software update, config change, hardware replacement, etc.]
**Target implementation date:** [Date or "TBD"]
**Responsible team / person:** [Who owns the fix implementation]
**Change record ID:** [If a change has been raised, include the ID]
**Blockers:** [What is preventing implementation of the permanent fix]

---

