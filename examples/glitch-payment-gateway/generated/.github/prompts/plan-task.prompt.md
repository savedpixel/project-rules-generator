---
description: Create a detailed implementation plan file with testing checklist before starting work
agent: agent
---

# Create a Detailed Implementation Plan

Before making any implementation changes, create a thorough implementation plan as a file in the repository. This plan is the execution baseline for the task. Do not begin implementation until the plan has been written, reviewed against the current codebase, and saved.

## What You Receive

The user will describe a task, feature, bug fix, refactor, cleanup, or UI adjustment. Your job is to plan it completely before any implementation begins.

## Steps

### 1. Review Recent Context

Before doing anything else, check for recent changes that may affect this task:

- Read the latest daily task log(s) in `docs/task/logs/` and review the most recent 1-2 files to understand what was recently changed, reverted, or completed.
- Read `docs/task/todo.md` and check for in-progress or related tasks that might overlap.
- Read `docs/task/lessons.md` and check for any lessons that apply to this type of work.
- Read the relevant `docs/logic/` file(s) and understand the current documented architecture.
- Read agent observation logs in `docs/agent-observations/` (`critical.md`, `recommendations.md`, `anomalies.md`) — identify unresolved observations that may change the plan scope.
- If this task relates to gateway configuration, admin UI, or checkout flow, also review the relevant baseline instruction files before planning.

### 2. Understand the Request

- Read the user's description carefully.
- Search the codebase to understand the current state of the affected files.
- Identify all files, functions, templates, data structures, selectors, routes, and dependencies involved.
- Cross-check code against documentation. If they diverge, note it in the plan's Assumptions section.
- If the request is ambiguous, make the most reasonable implementation decision you can and document that assumption explicitly.
- Distinguish clearly between what is required, what is implied, and what is out of scope.

### 3. Create the Plan File

Write the plan to: `docs/task/planning/{slug}.md`

- `{slug}` must be a concise kebab-case summary of the task, for example:
  - `add-webhook-signature-verification`
  - `implement-refund-processing`
  - `configure-sandbox-mode-toggle`
- Create `docs/task/planning/` if it does not already exist.

### 4. Plan File Structure

Use this exact structure:

    # Plan: {Title}

    > **Status:** Draft | In Progress | Complete
    > **Created:** YYYY-MM-DD
    > **Estimated steps:** {n}
    > **Risk level:** Low | Medium | High

    ## Context

    {Why this task exists. What problem it solves.}

    ## Current State

    {Current behavior, code, architecture, or UI structure. Reference specific files and anchor points. Include line numbers when they are stable and useful.}

    ## Target State

    {Desired end state. What should be true when complete?}

    ## Acceptance Criteria

    - {Concrete pass condition 1}
    - {Concrete pass condition 2}
    - {Concrete pass condition 3}

    ## Out of Scope

    - {Explicitly excluded item 1}
    - {Explicitly excluded item 2}

    ## Assumptions

    {Any assumptions made. Flag anything decided without explicit user input.}

    ## Implementation Steps

    - [ ] **Step 1: {Action}**
      - Files: `path/to/file.ext`
      - What: {Specific change}
      - Why: {Rationale}

    - [ ] **Step 2: {Action}**
      - Files: `path/to/file.ext`
      - What: {Specific change}
      - Why: {Rationale}

    ## Files Affected

    | File | Action | Description |
    | --- | --- | --- |
    | `path/to/file.ext` | Modify / Create / Delete | {What changes} |

    ## Dependencies & Risks

    - {Anything that could go wrong}
    - {External dependencies, ordering constraints}
    - {Possible regressions or coupling concerns}

    ## Validation Notes

    {Environment requirements, credentials, fixtures, setup steps, feature flags, or prerequisites required to test this task.}

    ## Testing Plan — BLOCKING (execute BEFORE docs or completion)

    ### Browser Verification (you execute this — not the user)

    Open Chrome MCP tools. Follow credentials in `browsetools.instructions.md`.

    1. Navigate to the affected page(s) on the WordPress dev site
    2. Take an initial screenshot and confirm rendering
    3. {Specific interaction test 1}
    4. {Specific interaction test 2}
    5. Verify the expected result for each interaction
    6. Take a final screenshot after interactions

    ### Regression Checks

    - [ ] {Existing feature that must still work}
    - [ ] {Another existing feature to verify}

    ### Edge Cases

    - [ ] {Edge case 1 and expected behavior}
    - [ ] {Edge case 2 and expected behavior}

    **STOP: Do NOT proceed to documentation, logging, or completion until ALL testing checks pass.**

    ## Rollback Plan

    {How to undo this change if something goes wrong. Include specific files, reversions, toggles, or commands where relevant.}

    ## Post-Implementation Checklist

    **Gate 1 — Code complete:**
    - [ ] All implementation steps marked complete
    - [ ] Acceptance criteria satisfied

    **Gate 2 — Testing (BLOCKING — must happen before Gate 3):**
    - [ ] All required browser verification steps executed via Chrome MCP tools
    - [ ] All regression checks executed
    - [ ] All edge cases checked
    - [ ] Final result matches the target state

    **Gate 3 — Documentation & logging (only after Gate 2 passes):**
    - [ ] Documentation updated per project instruction files
    - [ ] Task logged in `docs/task/logs/YYYY-MM-DD.md`
    - [ ] `docs/task/todo.md` updated

    **Gate 4 — Close out:**
    - [ ] Plan status updated to **Complete**

### 5. Write to todo.md

After creating the plan file, update `docs/task/todo.md` with a checklist that mirrors the implementation steps.

Rules for the todo entry:

- Keep the wording concise but specific.
- The checklist items must map directly to the implementation steps in the plan.
- Do not create vague todo entries such as "update UI" or "fix issue."
- Use the same task title or a close short form so the plan file and todo entry are easy to correlate.

### 6. Present the Plan

After saving the plan file and updating `docs/task/todo.md`, present a concise summary to the user that includes:

- What the task is
- How many implementation steps the plan contains
- Which files are affected
- Key risks, assumptions, or dependencies
- Where the plan file lives

Then follow this decision rule:

- If the active workflow is planning-only, stop after presenting the summary.
- If the user explicitly asked for a plan before implementation, stop after presenting the summary.
- Otherwise, wait for the next execution instruction defined by the active workflow.

Use this exact closing line when stopping for approval:

Plan saved to `docs/task/planning/{slug}.md`. Review and say **go** to start implementation, or tell me what to adjust.

## Rules

- Never skip the plan. If this prompt is invoked, planning is mandatory.
- Do not implement anything in this step. This prompt is planning only.
- Be specific. Every implementation step must describe the exact change, affected files, and rationale.
- Every step must be verifiable.
- Reference specific files and anchor points. Include line numbers when they are stable and useful.
- Cross-check code against documentation and record any divergence in Assumptions.
- Include explicit acceptance criteria and out-of-scope items.
- The testing plan is a blocking gate. Execute all required checks before documentation, logging, or marking the task complete.
- Browser verification via Chrome MCP tools is mandatory for any task that affects UI, routing, rendered output, forms, interaction flows, or client-side behavior.
- For backend-only or non-visual tasks (API client, webhook handler, refund processing), include an appropriate validation plan instead of forcing irrelevant browser steps.
- Do not write vague implementation steps. "Update the file" is not acceptable. "Add HMAC-SHA256 signature verification in `class-glitch-webhooks.php` before processing the callback payload" is acceptable.
- Do not leave testing generic. Define the exact checks required for this task.
- Do not leave rollback generic. State how the change can be reversed.
- Follow `task.instructions.md` and any related project planning, documentation, or workflow rules.
- If baseline instruction files exist for the feature area, use them as the comparison standard when identifying risk, scope, and implementation requirements.
