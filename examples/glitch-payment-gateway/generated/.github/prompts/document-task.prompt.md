---
description: Document the task(s) just completed — update task log, sync docs
agent: agent
---

# Document & Log Completed Work

You just helped me complete one or more tasks in this chat session. Now document everything per the project rules.

## Steps

1. **Review what was done** — Scan this conversation to identify all completed work (code changes, bug fixes, features, refactors, config changes, documentation updates, etc). Note which files were touched.

2. **Update today's task log** — Append row(s) to `docs/task/logs/YYYY-MM-DD.md` (use today's date). Create the file if it doesn't exist. Follow this format exactly:

```markdown
# Task Log — MM/DD/YYYY

> {n} tasks

| Title | Description | Start Date | End Date | Category | Type |
| --- | --- | --- | --- | --- | --- |
| {Action-oriented title} | {Concise outcome-focused description} | MM/DD/YYYY | MM/DD/YYYY | {Category} | {Type} |
```

- **Category**: `Gateway` · `Transactions` · `Webhooks` · `Admin` · `Security` · `API` · `PaymentMethods` · `Bug` · `Docs`
- **Type**: `Task` · `Sprint`
- Consolidate related sub-tasks into one row when they share the same scope
- Keep descriptions outcome-focused (what changed), not implementation minutiae
- Update the `> {n} tasks` counter in the header

3. **Log agent observations (BLOCKING GATE)** — Follow the rules in `.github/instructions/agent-observations.instructions.md`. Review everything you touched, read, or discovered. Append entries to the appropriate log(s) in `docs/agent-observations/`:

   - `critical.md` — Blocking issues, data inconsistencies, security concerns, broken assumptions
   - `recommendations.md` — Improvement suggestions, optimization opportunities, follow-up tasks
   - `anomalies.md` — Inconsistencies, drift between docs and code, oddities

   If no observations, explicitly state: `Observations: none.` **Silence is a violation.**

4. **Update affected documentation** — Based on which files were changed, update the relevant docs in `docs/logic/`. Check each domain:

   - Gateway class / checkout flow changes → `docs/logic/gateway.md`
   - Transaction status / refund / chargeback changes → `docs/logic/transactions.md`
   - Webhook handler changes → `docs/logic/webhooks.md`
   - Admin settings / credential changes → `docs/logic/admin-settings.md`
   - Security / signature / nonce changes → `docs/logic/security.md`
   - API client / request/response changes → `docs/logic/api-integration.md`
   - Payment method registration / display changes → `docs/logic/payment-methods.md`

   If no docs need updating (rare), explicitly state why.

5. **Confirm completion** — List what was logged, what observations were recorded (or state "none"), and which docs were updated.

## Rules

- Never defer documentation — do it all now in this response
- Never leave observations undocumented — log them in `docs/agent-observations/` or explicitly state "Observations: none."
- Don't ask for confirmation — just do it
- If unsure which category fits, pick the closest match
- Group same-scope work into single rows; don't create one row per file
