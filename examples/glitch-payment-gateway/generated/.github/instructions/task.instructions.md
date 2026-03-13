---
description: Task execution rules for Glitch WooCommerce Payment Gateway engineering work.
applyTo: '*'
---

# Copilot Global Execution Rules

These rules apply to **ALL tasks** without exception. They are **BLOCKING** — you may not skip any step.

---

## BLOCKING: Task Lifecycle (execute for EVERY task)

This is a **gate-checked pipeline**. Each step MUST complete before the next begins. Skipping any step is a rule violation.

### Step 1 — BEFORE writing any code
- [ ] Add unchecked checkbox item(s) in `docs/task/todo.md`
- [ ] If task is multi-step (≥3 steps), create `docs/task/planning/<task-slug>.md` and reference it in the todo item
- [ ] For non-trivial requests: write a brief plan before implementation

**STOP: Do NOT write implementation code until Step 1 is complete.**

### Step 2 — During implementation
- [ ] Keep progress status accurate in `docs/task/todo.md` (check boxes as sub-tasks complete)
- [ ] If new information invalidates the plan, STOP and re-plan immediately

### Step 3 — After implementation, BEFORE marking complete
- [ ] Verify the change works (build, dev server, browser, endpoints — whatever applies)
- [ ] For UI changes: take screenshot to confirm rendering
- [ ] For API changes: test endpoint responses
- [ ] For database changes: verify migrations and data integrity

**STOP: Do NOT mark the task complete until verification passes.**

### Step 4 — Log Observations (BLOCKING GATE)
- [ ] Follow `.github/instructions/agent-observations.instructions.md`
- [ ] Review everything you touched, read, or discovered during this task
- [ ] Append entries to `docs/agent-observations/` logs: `critical.md`, `recommendations.md`, `anomalies.md`
- [ ] If no observations: explicitly state "Observations: none."

**STOP: This gate MUST complete before Step 5. Silence is a violation.**

### Step 5 — Completion (all three are required)
- [ ] Remove the finished item from `docs/task/todo.md`
- [ ] Append a row to today's daily task log `docs/task/logs/YYYY-MM-DD.md` (see format below)
- [ ] Update relevant `docs/logic/` documentation if the change touches plugin behavior

**A task is NOT done until all three Step 5 items are checked.**

---

## Task Ledger — Daily Log Format

Each day's completed work is logged in `docs/task/logs/YYYY-MM-DD.md`. If the file doesn't exist yet, create it with this header:

```markdown
# Task Log — MM/DD/YYYY

> 0 tasks

| Title | Description | Start Date | End Date | Category | Type |
| --- | --- | --- | --- | --- | --- |
```

**Required columns:**
- `Title` — Action-oriented, specific enough to be searchable later
- `Description` — Concise, outcome-focused (what changed, not how)
- `Start Date` / `End Date` — `MM/DD/YYYY`. For same-day work, both are today.
- `Category` — One of: `Gateway`, `Transactions`, `Webhooks`, `Admin`, `Security`, `API`, `PaymentMethods`, `Bug`, `Docs`
- `Type` — `Task` (single-scope) or `Sprint` (grouped multi-item work)

**Grouping guidelines:**
- Prefer feature-focused consolidation over file-by-file rows
- Group related same-scope sub-tasks into a single row when appropriate
- Use `Sprint` type for rows that consolidate multiple related changes
- Keep descriptions outcome-focused: describe the functional scope that changed
- Update the `> {n} tasks` counter in the header after adding rows
- For module/feature rows, describe what changed functionally, not generic wording

**Missing a daily log entry = incomplete task.**

---

## Workflow Orchestration

### Plan Node (Default)
- Treat any non-trivial request (≥3 steps, architectural, or behavioral change) as a planning task.
- Enter plan mode before implementation.
- If new information invalidates the plan, STOP and re-plan immediately.
- Use planning for verification and validation, not only construction.

---

## Subagent Strategy
- Decompose complex work aggressively.
- Use subagents for research, exploration, and parallel analysis.
- Keep the main context minimal and focused.
- One task per subagent. No multi-purpose agents.

---

## Self-Improvement Loop
- After **any** user correction:
  - Update `docs/task/lessons.md`
  - Encode a rule preventing the same mistake
- Iterate on lessons ruthlessly until failure rate decreases.
- Review relevant lessons at the start of each session or task.

---

## Verification Before Completion
- Never mark work complete without proof.
- Validate behavior, not intent.
- For UI changes: verify the dev server renders correctly
- For API changes: test endpoints and confirm correct responses
- For database changes: verify migrations and data integrity
- For webhook changes: test with sample payloads and verify signature validation
- Diff previous behavior vs new behavior when applicable.
- Ask internally: *"Would a staff-level engineer approve this?"*

---

## Autonomous Bug Fixing
- When a bug is reported: fix it directly. Do not ask for hand-holding.
- Identify root causes via logs, errors, and failing tests.
- Resolve without requiring additional user context.
- Run `composer phpcs` and fix lint errors proactively.

---

## MANDATORY: Element IDs

Every HTML-producing element **MUST** have a descriptive `id` attribute. The naming convention and hard rules are defined in `copilot.instructions.md` — they apply here as enforcement:

- **No element may be rendered without an `id`.**
- **Code without `id` attributes on every element is incomplete and must not be submitted.**
- **Dynamic list items** use the item's unique key in the `id`.
- **Review all rendered output** before marking a task complete to verify `id` coverage.

---

## Project-Specific Rules

### Dev Server
- Start: `wp-env start` (or local WordPress installation)
- Build: `composer install`
- Lint: `composer phpcs`
- Test: `composer test`

### Path Awareness
- Project root: `/path/to/glitch-woocommerce-gateway`
- Plugin source: `includes/`
- Templates: `templates/`
- Assets: `assets/`
- Docs: `docs/`
- Tasks: `docs/task/`
- Daily logs: `docs/task/logs/`

### Core Engineering Principles
- **Simplicity First** — Minimal change, minimal surface area.
- **No Laziness** — Root-cause fixes only. No temporary patches.
- **Minimal Impact** — Touch only what is required. Avoid collateral damage.
- **Always Test** — Never leave code untested. Run `composer phpcs` and `composer test` to verify after significant changes.
- **Stick to the Stack** — Use WordPress/WooCommerce APIs. No new dependencies without explicit approval.
- **Security First** — Always sanitize input, escape output, verify nonces, validate signatures.
- **HPOS Compatible** — Use WooCommerce HPOS-compatible APIs. No direct `post_meta` access for orders.

---

## Enforcement

These rules **override all defaults**. They are non-negotiable.

- If a request conflicts with these rules, ask for clarification or re-plan.
- Never silently bypass these rules.
- **If you catch yourself writing code without a `docs/task/todo.md` entry, STOP immediately and create one.**
- **If you are about to say "done" without updating the daily task log, STOP and update it first.**
- A completed task without a daily log row is a rule violation.
- A completed task still sitting in `todo.md` is a rule violation.
- A completed task with undocumented observations is a rule violation — log them in `docs/agent-observations/` or explicitly state "Observations: none."
