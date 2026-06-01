# How to Contribute to This Knowledge Base

## Who Should Contribute

Every engineer who resolves a ticket that required investigation beyond the
standard runbooks should consider whether that investigation should become a
knowledge base article. The contribution bar is not "did I discover something
nobody has ever seen?" - it is "did I spend more than 30 minutes on something
that the next engineer could resolve in 5 minutes if they had documentation?"

If yes: write the article.

Contributing to this knowledge base is not optional overhead. It is part of
the job. Every hour spent writing an article that saves five engineers 30 minutes
each returns 2.5 hours to the team. Across a year of tickets, a culture of
documentation is the difference between a team that improves and a team that
stays at the same capability level.

---

## The Contribution Process

### Step 1 - Identify the gap

A gap exists when:
- You resolved a ticket using knowledge that is not documented here
- You found an article that is incomplete, incorrect, or missing an edge case
- A recurring issue has no article (check the article ID list first)
- A template exists but no article was created from it after a PIR

Do not create an article that is already covered adequately elsewhere. Check the
article list first and search for key terms.

---

### Step 2 - Choose the correct article type

| Situation | Article Type | Template to Use |
|-----------|-------------|----------------|
| Resolved a specific type of ticket | Knowledge Article | knowledge-article-template.md |
| Root cause found, permanent fix pending | Known Error Record | known-error-record-template.md |
| Completed a P1 or significant P2 incident review | Post-Incident Review | post-incident-review-template.md |

---

### Step 3 - Write the article

Copy the appropriate template from the `templates/` directory.
Rename it using the correct article ID format:

```
[CATEGORY]-[NNN]-[brief-slug].md

Examples:
  windows/win-006-appcrash-on-login.md
  azure/az-005-function-app-cold-start.md
  linux/lnx-004-cron-job-not-running.md
  networking/net-004-spanning-tree-loop.md
  security/sec-004-pass-the-hash-detection.md
  microsoft-365/m365-005-teams-guest-access.md
```

The NNN number continues from the highest existing article in that category.
Check the existing files before assigning a number.

Fill in every section of the template. Delete template instructions. Do not
publish an article with `[placeholder]` text remaining.

---

### Step 4 - Meet the article quality standards

Before submitting, verify the article meets the standards in
`meta/article-quality-standards.md`. Specifically:

- Every command block is complete and runnable as written
- Every command has a comment explaining what it does and what the output means
- The article has at least one "Known Edge Cases" section
- The article does not simply repeat information from official documentation
  — it adds context about what can go wrong, what things get confused with,
  or what the documentation does not mention
- The article was tested: you ran the commands and they produced the stated output

---

### Step 5 - Submit via pull request

```bash
# Clone the repo (if not already cloned)
git clone https://github.com/YOUR-USERNAME/it-support-knowledge-base.git
cd it-support-knowledge-base

# Create a branch for your article
git checkout -b add-win-006-appcrash-on-login

# Add the article
cp templates/knowledge-article-template.md windows/win-006-appcrash-on-login.md
# Edit the file
git add windows/win-006-appcrash-on-login.md
git commit -m "Add WIN-006: Application crash on login — AppCompatFlags registry fix"

# Push and open a pull request
git push origin add-win-006-appcrash-on-login
```

Pull request description must include:
- The article ID and title
- What ticket type or incident prompted the article
- Whether the commands were tested and on which OS/version
- Any sections that are incomplete and why

---





