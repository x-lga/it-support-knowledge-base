# TEMPLATE - Knowledge Article

**HOW TO USE THIS TEMPLATE:**
Copy this file, rename it to match the article ID format
(e.g., win-006-new-topic.md), fill in every section, and
delete all template instructions (lines in [square brackets])
before publishing. Do not leave [placeholder] text in finished articles.

---

# [ARTICLE-ID] - [Title: Problem Statement in Plain Language]

**Article ID:** [CATEGORY-NNN - e.g., WIN-006, LNX-004, AZ-005]
**Category:** [Technology - Subtopic — e.g., Windows - Registry | Azure - Networking]
**Severity:** [P1/P2/P3/P4 as appropriate - include both ends of the range if variable]
**Cert alignment:** [Which certs this knowledge supports - A+, Net+, Sec+, AZ-104, etc.]
**Last verified:** [YYYY-MM]
**Author:** [Your name or initials]
**Reviewed by:** [Reviewer name or "Not yet reviewed"]

---

## What This Article Covers

[2-4 sentences maximum. Answer: what problem does this article help with, and why
is this problem worth documenting? If this is a common ticket type, say so. If
this is a difficult problem that gets misdiagnosed, explain what it gets confused
with and why this article prevents that confusion.]

[Do NOT repeat the title. Do NOT write "this article explains how to...". Start
with the problem context, not with what the article does.]

---

## [Context Heading - Why This Is Harder Than It Looks / Common Misunderstandings]

[This section provides the background knowledge that makes the diagnostic steps
make sense. Every article should have one. Without this, the reader follows steps
without understanding why - and cannot adapt when the steps do not match their
specific situation.]

[Examples of good context sections:
- Explaining why a symptom is ambiguous (same symptom, multiple causes)
- Explaining a technical concept that the diagnostic steps depend on
- Describing what the correct resolution is NOT (common wrong approaches)
- Providing a decision table or classification framework before the steps begin]

---

## Step 1 - [First Diagnostic or Preparatory Action]

[Each step should have:]
[1. A brief statement of WHY this step comes first]
[2. The actual commands or portal clicks — fully written out, nothing omitted]
[3. What the output means — what does "good" look like, what does "bad" look like]
[4. What to do based on the output — which step to go to next]

```[language]
# [Explain what this code block does in a comment at the top]
# [Every significant line should have a comment explaining it]
[code here]
```

[If the step has a decision point - "if X then Y, if Z then W" — use a table:]

| If you see | It means | Do this |
|-----------|---------|---------|
| [output A] | [meaning] | [action] |
| [output B] | [meaning] | [action] |

---

## Step 2 - [Second Action]

[Follow the same pattern. Number steps sequentially.]
[Steps should be in the order they would actually be performed - not in order of
how interesting they are to write.]

---

## Step N - [Resolution or Escalation]

[Every article must end with either a clear resolution or a clear escalation
criteria. Never leave the reader without a defined endpoint.]

[If escalating: provide the exact escalation package content - what information
the L2 engineer needs. Do not just say "escalate to L2."]

---


## Known Edge Cases

[This section is mandatory. Every issue has edge cases that break the standard
procedure. Document them here.]

[Format:]
**[Edge case name - what makes it different]:**
[What the different situation looks like, and how the resolution differs.
If the resolution is the same, this probably is not a separate edge case.]

---

## Post-Resolution Checklist

[Optional but recommended for P1/P2 articles. A checklist of things to verify
after resolution - confirms the fix actually worked and nothing was left incomplete.]

- [ ] [Verification step 1]
- [ ] [Verification step 2]
- [ ] [User notified / confirmed working]
- [ ] [Ticket updated with root cause and resolution notes]

---

## Related Articles

[Links to other articles in this knowledge base that are related. Use article IDs.]
- [ARTICLE-ID]: [Brief description of why it is related]


---

