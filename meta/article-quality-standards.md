# Article Quality Standards

These standards define what a finished, publishable knowledge base article looks
like. They exist to ensure that every article in this knowledge base can actually
be used during an active incident by an engineer who has never seen the issue before.

An article that fails these standards is worse than no article. An incomplete or
inaccurate article creates false confidence, sends engineers down the wrong path,
and takes time to undo. When in doubt: either meet the standard or do not publish.

---

## Standard 1 - Every Command Must Be Complete and Runnable

**What this means:**
Every command block in an article must be complete as written. An engineer must
be able to copy the command, replace only the explicitly marked variables
(formatted as `[REPLACE THIS]`), and run it without needing to figure out
missing arguments, missing import statements, or missing prerequisite steps.

**What this does NOT mean:**
Commands do not need to be universal. It is fine for a command to work only on
Ubuntu 22.04, or only on Windows Server 2022, or only on Azure subscriptions.
The article must state this clearly at the top.

**Test:** Read every command block and ask: could an engineer who has never seen
this command before run it successfully from this article alone? If not: the
article fails this standard.

**Failing example:**
```
Get-ADUser $Username | Select-Object *
```
This fails because `$Username` is not defined anywhere in the article.

**Passing example:**
```powershell
# Replace [USERNAME] with the SamAccountName of the affected user
$Username = "[USERNAME]"   # e.g., jsmith
Get-ADUser $Username | Select-Object DisplayName, SamAccountName, Enabled, LockedOut
```

---

## Standard 2 - Every Command Must Have a Comment Explaining the Output

**What this means:**
For every command where the output is not immediately obvious, the article must
explain what the expected output looks like for both healthy and unhealthy states,
and what the engineer should do based on each output.

**Why:** Engineers at 2am do not know what a healthy output looks like if they
have never run the command before. An article that shows the command but not
the expected output provides half the information needed.

**Test:** For every command in the article: does the article tell the reader what
output to expect and what it means?

---

## Standard 3 - The Article Adds Context Not Available in Official Documentation

**What this means:**
The article must contain at least one of the following that is not in the
official vendor documentation:
- An explanation of why a symptom is frequently misdiagnosed
- A table or decision framework for distinguishing between multiple causes
  that produce the same symptom
- A "Known Edge Cases" section documenting behaviour that official documentation
  does not mention
- An honest description of when the standard procedure does NOT work and why
- The information that was actually needed to resolve a real ticket that the
  documentation alone did not provide
