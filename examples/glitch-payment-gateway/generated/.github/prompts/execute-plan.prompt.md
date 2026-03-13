---
description: 'Execute a previously planned task from a plan file'
agent: agent
---

# Execute Plan

You are about to execute a plan that was previously created. The user will reference a plan file using `#`. That file contains every detail you need.

## Step 1: Read the Plan

1. Open the referenced plan file and read it completely.
2. Confirm the plan has:
   - **Objective** — what is being achieved
   - **Implementation Steps** — numbered, specific steps
   - **Testing Plan** — browser verification steps
3. If the plan file is missing or incomplete, STOP and tell the user.

## Step 2: Review Recent Context

Before executing, check for recent changes that might affect the plan:

1. Read the latest daily task log in `docs/task/logs/` — look at the most recent file(s).
2. Read `docs/task/todo.md` — check for related in-progress or recently completed work.
3. Read `docs/task/lessons.md` — check for relevant lessons that apply to this plan.
4. Read agent observation logs in `docs/agent-observations/` (`critical.md`, `recommendations.md`, `anomalies.md`) — check for unresolved observations that may affect execution.
5. If ANY recent change conflicts with or overlaps the plan, STOP and tell the user before proceeding.

## Step 3: Execute Implementation Steps

Follow the plan's implementation steps **exactly as written**, in order.

- Do not skip steps.
- Do not reorder steps.
- Do not add steps that aren't in the plan.
- If a step is blocked or doesn't make sense given current code state, STOP and tell the user.

## Step 4: Execute Testing Plan — BLOCKING

**This is a blocking gate. Do NOT proceed to Step 5 until ALL tests pass.**

Execute every test listed in the plan's Testing Plan section using Chrome MCP tools.

Required tools (use the actual MCP tool names):

- `take_screenshot` — visual verification
- `click` / `fill` — interaction testing
- `evaluate_script` — DOM/state inspection
- `list_console_messages` — error checking

For each test:

1. Execute it with the appropriate MCP tool
2. Verify the expected result
3. If it fails, fix the issue and re-test

**Reading code is not testing. Confirming something "looks right" without a browser is not testing.**

> STOP — Do NOT proceed to documentation, task logging, or marking complete until ALL testing checks pass.

Use credentials and browser config from `browsetools.instructions.md`.

## Step 5: Documentation & Logging

Only after ALL tests pass:

1. Update any documentation files that the plan specifies.
2. Add an entry to today's daily task log (`docs/task/logs/YYYY-MM-DD.md`).
3. Log agent observations — follow `.github/instructions/agent-observations.instructions.md`. Append to `docs/agent-observations/` logs (`critical.md`, `recommendations.md`, `anomalies.md`). If none, state: "Observations: none."
4. Update `docs/task/todo.md` — check off the item if it exists.

## Step 6: Close Out

1. Confirm all implementation steps are done.
2. Confirm all tests passed.
3. Confirm docs and logs are updated.
4. Report to the user: what was done, what was tested, and the result.

## Rules

- **Follow the plan exactly.** Do not deviate, add scope, or "improve" things beyond what the plan specifies.
- **Testing is not optional.** Every test in the Testing Plan must be executed with Chrome MCP tools. If you skip testing, the task is incomplete.
- **Do not ask the user to test.** You are the tester. Use Chrome MCP tools.
- **If the plan is outdated** (references files that no longer exist, or code that has changed significantly), STOP and tell the user the plan needs updating.
- **Do not close a task with observations undocumented.** Log them in `docs/agent-observations/` or explicitly state "Observations: none."
- **Follow task.instructions.md** for the gate-checked completion pipeline.
