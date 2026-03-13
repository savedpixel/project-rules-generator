---
description: Mandatory agent observation logging — anomalies, recommendations, and critical findings must be disclosed before commit
applyTo: "*"
---

# Agent Observation Logs — Mandatory Disclosure System

## Purpose

During any task — planning, implementation, auditing, reporting, or debugging — you will encounter information the user needs to know: anomalies in data, inconsistencies between code and documentation, improvement opportunities, potential risks, or broken assumptions. **You MUST proactively disclose these findings.** Silence is a violation.

This system exists because agents have historically buried important findings, only surfacing them when explicitly asked for a report. That is unacceptable. The observation logs are the user's primary way to stay informed about the health of their system without having to ask.

---

## Directory Structure

```
docs/agent-observations/
├── critical.md          ← Open critical issues only
├── recommendations.md   ← Open recommendations only
├── anomalies.md         ← Open anomalies only
└── closed/
    ├── critical.md      ← Resolved critical issues (archive)
    ├── recommendations.md ← Resolved recommendations (archive)
    └── anomalies.md     ← Resolved anomalies (archive)
```

- **Root files** (`docs/agent-observations/*.md`) contain **only open/in-progress items**. This gives the user a clean view of what actually needs attention.
- **Closed files** (`docs/agent-observations/closed/*.md`) are append-only archives of resolved items. Never delete entries from closed files.

---

## The Three Observation Logs

### 1. `critical.md` — Blocking Issues

Things that are **broken, dangerous, or require immediate attention**. If you find something critical and do NOT log it, you are hiding a problem from the user.

Examples:
- Data inconsistencies that affect production output
- Security misconfigurations (exposed credentials, missing auth checks, unsafe defaults)
- Broken assumptions that code relies on
- Regression risks from the current task
- Data corruption or loss
- Webhook signature verification bypass
- Unescaped user input in payment forms

### 2. `recommendations.md` — Improvement Suggestions

Things that **aren't broken but should be addressed**. These are follow-up tasks, optimization opportunities, or architectural improvements.

Examples:
- "Consider adding retry logic for transient Glitch API failures"
- "The admin settings page could benefit from credential validation on save"
- "Webhook handler should implement idempotency keys to prevent duplicate processing"
- "Consider adding WooCommerce Subscriptions compatibility"

### 3. `anomalies.md` — Inconsistencies & Drift

Things that are **odd, inconsistent, or divergent** but not necessarily broken.

Examples:
- Documentation says X but code does Y
- A payment method is registered but never rendered at checkout
- Two config values that should match but don't
- A template file that appears to be unused but hasn't been verified

---

## Entry Format

Every entry in any of the three logs MUST use this exact table format:

```markdown
| {date} | {source} | {observation} | {impact} | {action} | Open |
```

**Columns:**
- **Date** — `YYYY-MM-DD` when the observation was made
- **Source** — What task/file/action led you to discover this
- **Observation** — Concise description of what you found. Be specific — include file paths, URLs, values, or data points.
- **Impact** — Why this matters. What could go wrong, or what's being missed.
- **Action** — What should be done about it. Be prescriptive.
- **Status** — `Open` or `In Progress`. Items in the root files are always open/active. Closed items live in `closed/`.

---

## When to Write Observations

### During Implementation (MANDATORY)

As you work through any task, you MUST actively watch for:

1. **Data you're touching** — Is it consistent? Are formats normalized? Are there orphaned references?
2. **Code you're reading** — Does it match what the docs say? Are there edge cases not handled? Are there deprecated patterns?
3. **Assumptions you're making** — Are they validated? What if they're wrong?
4. **Side effects of your changes** — Could your change break something else? Did you change a shared resource?
5. **Things outside your task scope** — If you notice something wrong while working on something else, log it.

### During Planning

When reading code/data to plan a task, log anything unexpected you find — even if it's unrelated to the planned work.

### During Reporting

When generating a report, your Findings/Recommendations sections MUST be cross-posted to the relevant observation log. Do not put recommendations only in a report file — they must also appear in the observation logs.

---

## The Observation Gate — BLOCKING

This is a **hard gate** in the task completion workflow. It runs BEFORE documentation updates, task logging, and git commits.

### Before you commit, you MUST:

1. **Review your work** — Scan everything you touched, read, or discovered during this task.
2. **Classify observations** — For each finding, determine: critical, recommendation, or anomaly.
3. **Append entries** to the appropriate log file(s) in `docs/agent-observations/`.
4. **If you have NO observations**, you MUST still acknowledge this explicitly in your response:
   > `Observations: none — no anomalies, recommendations, or critical issues detected.`

**Silence is a violation.** You must either log observations or explicitly state "none." Proceeding to commit without doing either is a rule breach.

### What counts as "no observations"

You may legitimately have no observations when:
- The task was purely mechanical (e.g., renaming a variable, fixing a typo)
- You touched very little code and the code you touched was straightforward
- Everything you encountered was consistent and well-documented

But if you read 50+ files, touched schemas across many pages, or made architectural changes — it is extremely unlikely you have zero observations. Be honest with yourself.

---

## Closing Observations — Move to `closed/`

When an observation is resolved, you MUST **move it** from the root file to the corresponding `closed/` file. Do NOT leave closed items in the root files.

### Closing procedure:

1. **Remove the row** from `docs/agent-observations/{type}.md`
2. **Append the row** to `docs/agent-observations/closed/{type}.md`, changing the Status column to `Closed` and adding the closure date and reason
3. The root file must only ever contain `Open` or `In Progress` items

### Who can close:

- **Critical observations:** Only the user can authorize closing. You may move to `closed/` only after user confirmation.
- **Recommendations & anomalies:** You may close if you fix the issue in a subsequent task, or if the user says "ignore it" or "not relevant."

---

## Cross-References

- **Reports:** When generating reports (`report.prompt.md`), findings and recommendations from the report MUST also be appended to the relevant observation log.
- **Plans:** When creating plans (`plan-task.prompt.md`), read the observation logs first. Unresolved observations may affect your plan scope.
- **Execution:** When executing plans (`execute-plan.prompt.md`), check observation logs for items that may be relevant to the current work.
- **Task Documentation:** When documenting tasks (`document-task.prompt.md`), the observation gate runs BEFORE documentation — observations first, then docs.

---

## File Location Exception

The observation log files (`critical.md`, `recommendations.md`, `anomalies.md` — both root and `closed/`) are **persistent files** and are exceptions to the date-prefix naming rule. They are NOT date-prefixed because they are living documents, not point-in-time snapshots.

---

## Enforcement Summary

| Situation | Required Action |
|---|---|
| You find something broken/dangerous | Append to `critical.md` immediately |
| You have an improvement suggestion | Append to `recommendations.md` before commit |
| You notice an inconsistency | Append to `anomalies.md` before commit |
| You have no observations | State "Observations: none" explicitly |
| You're generating a report | Cross-post findings to observation logs |
| You're creating a plan | Read observation logs first |
| You're about to commit | Verify observations are logged or explicitly "none" |
| An observation is resolved | Move row from root file → `closed/` file |
