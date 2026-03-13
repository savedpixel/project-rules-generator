---
description: Generate a report on completed work, a feature area, or a specific topic — saved to docs/reports/
agent: agent
---

# Generate Report

You are generating a structured report based on this chat session or a user-specified topic. The report will be saved as a permanent markdown file in `docs/reports/`.

**CRITICAL — Voice & Authorship:** The report must be written **in the user's voice**, as if the user personally performed the work, made the decisions, and is presenting the findings. The user needs to be able to share these reports directly with colleagues or management without editing. Never reference yourself (the agent, Copilot, AI, etc.) anywhere in the report. Use first person ("I", "my") or neutral professional voice ("We audited...", "The team implemented..."). Never say "the user asked" or "the user requested" — instead say "I identified...", "after reviewing...", or describe the motivation directly.

## What You Receive

The user may:
- Ask you to report on the work just completed in this chat session
- Ask you to report on a specific feature, module, or area of the codebase
- Ask you to report on a specific topic (e.g., architecture decisions, audit results, performance analysis, migration status)
- Provide a custom focus or scope for the report

If the user doesn't specify a scope, default to **summarizing all work completed in this chat session**.

## Steps

### 1. Gather Context

- **If reporting on completed work:** Scan this conversation to identify all changes made — code changes, bug fixes, features, refactors, config changes, documentation updates. Note which files were touched and what the outcomes were.
- **If reporting on a specific topic:** Search the codebase, read relevant files, and gather all information needed to produce an accurate, thorough report on the requested topic.
- **Check recent task logs** in `docs/task/logs/` for additional context on recent work.
- **Read relevant `docs/logic/` files** to understand the documented state of affected areas.
- **Read agent observation logs** in `docs/agent-observations/` (`critical.md`, `recommendations.md`, `anomalies.md`) — check for unresolved observations that may be relevant.

### 2. Determine Report Type & Title

Based on what you're reporting on, choose the most appropriate report type:

| Type | When to use |
|------|------------|
| `work-summary` | Summarizing work completed in this session or a date range |
| `feature-report` | Deep dive on a specific feature or module |
| `audit` | Code quality, security, performance, or compliance analysis |
| `architecture` | Architecture decisions, patterns, or technical debt assessment |
| `migration` | Status of a data, code, or infrastructure migration |
| `incident` | Bug investigation, root cause analysis, resolution |
| `comparison` | Evaluating options, tools, or approaches |
| `status` | Project or sprint status update |
| `integration` | Payment provider integration status, API compatibility |
| `custom` | Anything that doesn't fit the above — describe the type in the report header |

Create a descriptive, searchable title (e.g., "Webhook Signature Verification Implementation", "Glitch API Integration Audit", "Refund Processing Architecture Decision").

### 3. Generate the Report File

**File path:** `docs/reports/{YYYY-MM-DD}-{slug}.md`

- `{YYYY-MM-DD}` = today's date
- `{slug}` = kebab-case summary of the report topic (e.g., `webhook-verification-implementation`, `api-integration-audit`, `refund-architecture-decision`)

If `docs/reports/` doesn't exist, create it.

### 4. Report Structure

Use this template, adapting sections based on report type:

```markdown
# {Report Title}

> **Type:** {report type from table above}
> **Date:** YYYY-MM-DD
> **Scope:** {Brief description of what this report covers}

---

## Summary

{2-4 sentence executive summary. What is this report about? What are the key findings or outcomes?}

## Context

{Why does this report exist? What prompted it? Link to relevant tasks, plans, or user requests.}

## Details

{The main body of the report. Structure depends on report type:}

{For work-summary: list changes made, organized by feature area or file group}
{For feature-report: architecture, current state, key behaviors, known issues}
{For audit: findings organized by severity or category, with file references}
{For architecture: decision context, options considered, decision made, rationale}
{For migration: what's done, what's remaining, blockers, timeline}
{For incident: timeline, root cause, fix applied, prevention measures}
{For comparison: criteria, options, analysis, recommendation}
{For status: progress summary, blockers, next steps}

### Files Involved

| File | Role |
| --- | --- |
| `path/to/file.ext` | {What this file does in the context of this report} |

## Findings / Outcomes

{Key takeaways, numbered or bulleted:}

1. {Finding or outcome 1}
2. {Finding or outcome 2}
3. {Finding or outcome 3}

## Recommendations / Next Steps

{If applicable — what should happen next based on this report:}

- {Recommendation or next step 1}
- {Recommendation or next step 2}

```

### 5. Capture Recommendations & Anomalies (MANDATORY)

Before final confirmation, scan the report's **Findings / Outcomes** and **Recommendations / Next Steps** sections.

- If there are unresolved anomalies or recommendations, append entries to the appropriate observation log(s) in `docs/agent-observations/`:
  - `critical.md` — blocking issues, security concerns, data corruption
  - `recommendations.md` — improvement suggestions, follow-up tasks
  - `anomalies.md` — inconsistencies, drift, oddities
  - Follow the entry format in `.github/instructions/agent-observations.instructions.md`
- If there are none, explicitly state: `Observations: none (no anomalies, recommendations, or critical issues).`

### 6. Save & Confirm

1. Write the report file to `docs/reports/{YYYY-MM-DD}-{slug}.md`
2. Confirm to the user:
   - Report title and type
   - Where it was saved
   - What observations were logged to `docs/agent-observations/` (or state "Observations: none")
   - Brief summary of key findings/outcomes

## Rules

- **Always save to `docs/reports/`.** Never write reports inline in chat only — they must be persisted as files.
- **Be thorough but concise.** Reports should be comprehensive enough to be useful weeks later, but not padded with filler.
- **Use specific file references.** When mentioning code, include file paths and line numbers where relevant.
- **Don't fabricate information.** If you don't have enough context to report on something, say so in the report.
- **Don't ask for confirmation before generating.** Just produce the report based on available context.
- **Date-prefix the filename.** This ensures reports sort chronologically in the directory.
- **If multiple reports are generated the same day on different topics**, each gets its own file with a distinct slug.
- **Adapt the template.** Not every section applies to every report type — omit sections that don't apply rather than leaving them empty.
- **Do not leave unresolved recommendations/anomalies only inside reports.** Persist them in `docs/agent-observations/` logs or explicitly state "Observations: none."
- **Write in the user's voice.** The report must read as if the user wrote it and did the work. Use first person ("I"/"we") or neutral professional voice. Never reference Copilot, the agent, AI, or "the user." Never include footers like "Generated by Copilot." The user must be able to share the report as-is with colleagues or management.
