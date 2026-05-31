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

