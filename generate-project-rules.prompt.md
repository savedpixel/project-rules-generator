---
description: Generate Project Instructions
agent: agent
---

# Generate Project Instructions — Agent Prompt

You are setting up a **pointer-based Copilot instructions system** for a project. This system keeps the main instructions file slim (~150-250 lines) by pointing to dedicated documentation files rather than containing inline feature logic.

Your job is to **discover** the project, then **generate or update** these artifacts:

## Instruction Files

1. `.github/instructions/copilot.instructions.md` — main project instructions (pointer-based)
2. `.github/instructions/task.instructions.md` — task tracking lifecycle rules (blocking gate-checked pipeline)
3. `.github/instructions/agent-observations.instructions.md` — mandatory pre-commit agent observation disclosure system
4. `.github/instructions/browsetools.instructions.md` — browser verification protocol
5. `.github/instructions/ratelimitting.instructions.md` — rate limit prevention rules (always active)

## Prompt Files

6. `.github/prompts/document-task.prompt.md` — reusable agent prompt for documenting completed work
7. `.github/prompts/plan-task.prompt.md` — reusable agent prompt for creating detailed implementation plans before coding
8. `.github/prompts/execute-plan.prompt.md` — reusable agent prompt for executing a previously created plan
9. `.github/prompts/report.prompt.md` — reusable agent prompt for generating a report on completed work or a specific topic and saving it to `docs/reports/`

## Observation Logs

10. `docs/agent-observations/` — `critical.md`, `recommendations.md`, `anomalies.md` (open items); `closed/` subdirectory for resolved items

## Task Tracking Files

11. `docs/task/todo.md`, `docs/task/lessons.md`, `docs/task/logs/`, `docs/task/planning/`

---

## Phase 1: Discovery

Before generating anything, you must thoroughly analyze the project. Run these discovery steps:

### 1.1 Project Structure

- List the root directory to identify framework, language, and tooling files
- List `src/` or equivalent source directory recursively (1-2 levels deep)
- List `docs/` if it exists — identify any existing documentation
- List `.github/` if it exists — identify any existing instruction files
- Check for `package.json`, `Cargo.toml`, `go.mod`, `Package.swift`, `requirements.txt`, etc. to determine the stack
- Read the project's README if it exists

**Capture:**

- Framework & language (e.g., Next.js 16, Swift 6, Django, etc.)
- Package manager & dependencies (read package.json/lockfile for key deps)
- Source directory layout (pages, components, lib, API routes, etc.)
- Database type (SQLite, Postgres, etc.) and ORM/driver
- UI library (shadcn/ui, Material UI, custom, etc.)
- State management (Zustand, Redux, Context, etc.)
- Testing framework (Vitest, Jest, XCTest, etc.)
- Styling approach (Tailwind, CSS modules, styled-components, etc.)
- Auth mechanism (JWT, NextAuth, Clerk, etc.)
- **Git repository** — check if `.git/` exists at the project root. If yes, auto-commit & push rules will be generated in `copilot.instructions.md` and `task.instructions.md`.
- **Rendering stack** (critical for Element ID rule — see Phase 3): identify ALL technologies that produce HTML output. Examples:
  - **JSX/TSX:** React, Next.js, Remix, Solid
  - **PHP templates:** WordPress themes, Laravel Blade, Symfony Twig
  - **Vue SFC:** Nuxt, Vue
  - **Server templates:** EJS, Handlebars, Pug, Jinja2, ERB
  - **Svelte/SvelteKit**
  - **Static HTML**
  - If the project has **no** HTML rendering (pure CLI, API-only, library), note this — the Element ID rule will be skipped.

### 1.2 Existing Documentation

- Scan `docs/` for any existing feature docs, architecture docs, or logic files
- Scan for any existing `.github/instructions/` files
- Scan for any existing `.github/skills/` files
- Check for inline documentation in README, CONTRIBUTING, etc.
- Look for any `tasks/`, `docs/task/`, or similar task tracking directories

**Capture:**

- List of existing doc files with their domain/topic
- Any existing instruction files (to update rather than overwrite)
- Any existing skills files (to reference)
- Current task tracking location and format (if any)
- **Task log format** — check for any existing daily logs, changelogs, or done.md files; if present, note the format to preserve it; if not, the agent will create the daily log system
- **Observation / notes logs** — check for any existing agent observation logs, anomaly trackers, or notes directories; if present, note their location and format to preserve and integrate

### 1.3 Dev Environment

- Identify the dev server command (e.g., `npm run dev`, `swift run`, `python manage.py runserver`)
- Identify the build command
- Identify the test command
- Identify the deploy command (if any scripts exist)
- Identify the dev server URL and port
- Check for `.env.example` or `.env` to identify required environment variables
- Identify any login/auth credentials needed for dev (check seeds, fixtures, or ask)

### 1.4 Feature Domains

Analyze the source code to identify distinct feature domains. For each domain, note:

- Key files/directories involved
- Whether existing documentation covers it
- Whether it needs a new doc file created

**Common domains to look for:**

- Auth system
- Data models / database schema
- API routes / endpoints
- UI component system
- State management patterns
- Email / notifications
- Background jobs / workers
- Deployment / infrastructure
- Third-party integrations

### 1.5 Identify Task Categories & Types

Analyze the project's feature domains, routes, data models, directory structure, and existing task/issue conventions to propose **project-relevant task categories and types**.

**Discover categories by examining:**

- Feature domains identified in 1.4 (each major domain may warrant a category)
- Directory structure (e.g., `api/`, `auth/`, `email/`, `workers/` → `API`, `Auth`, `Email`, `Infra`)
- Issue labels or existing task tags (if any)
- The project's primary domain (e.g., e-commerce → `Orders`, `Products`, `Payments`)

**Output two lists:**

1. **`TASK_CATEGORIES`** — 5-10 categories derived **entirely from this project's actual codebase**. Do NOT copy example categories from other projects or use generic labels. Instead:
   - Start from the feature domains discovered in 1.4 — each major domain is a candidate category.
   - Check the directory structure for top-level concerns (e.g., if there's an `auth/` dir, `Auth` is a category; if there's a `billing/` dir, `Billing` is a category).
   - Check existing issue labels, task tags, or changelog groupings for the project's own vocabulary.
   - Consider the project's primary business domain — a hosting company's categories differ from a CRM's categories.
   - Merge closely related domains into one category when they always change together.
   - Always include `Bug` and `Docs` unless the project has its own equivalent terms.
   - **Every category MUST map to a real, observable concern in THIS project.** If a category doesn't correspond to actual files, routes, or features you found during discovery, drop it.
   - **Never copy example category lists verbatim.** The categories below are illustrations of the *reasoning process*, not templates to reuse:
     - A project with `src/payments/`, `src/orders/`, `src/products/` → `Payments`, `Orders`, `Products` (derived from dirs)
     - A project with SEO meta output, blog system, pricing pages → `SEO`, `Content`, `Pricing` (derived from features)
     - A pure API project with no UI → no `UI` category (don't include what doesn't exist)

2. **`TASK_TYPES`** — Always include:
   - `Task` — single-scope unit of work
   - `Sprint` — grouped multi-item work (several related tasks consolidated into one row)

These lists will be injected into `copilot.instructions.md`, `task.instructions.md`, and `document-task.prompt.md`.

---

## Phase 2: Documentation Structure

### 2.1 Create Documentation Directory

Ensure these directories exist:

```text
docs/
├── logic/          ← Feature logic docs (one per domain)
├── reports/        ← Generated reports (saved by report.prompt.md)
├── server/         ← Server & deployment docs
├── agent-observations/ ← Agent observation logs (mandatory pre-commit disclosure)
│   ├── critical.md
│   ├── recommendations.md
│   ├── anomalies.md
│   └── closed/         ← Resolved observations archive
└── task/           ← Task tracking
    ├── todo.md
    ├── lessons.md
    ├── logs/       ← Daily task logs (YYYY-MM-DD.md)
    └── planning/   ← Multi-step task plans
```

### 2.2 Create or Identify Feature Doc Files

For each discovered feature domain, either:

- **Map** to an existing doc file, OR
- **Create** a new `docs/logic/<DOMAIN_NAME>.md` file

Each feature doc should follow this template:

````markdown
# <Feature Name>

<!-- Brief description of what this feature does. -->

<!-- Updated: YYYY-MM-DD -->

---

## Architecture

<!-- Key files, directories, database tables, API routes involved. -->

## Key Behaviors

<!-- How the feature works — flows, patterns, configuration. -->

## Common Patterns

<!-- Code patterns, conventions specific to this feature. -->
````

**Rules for what goes in feature docs vs. instructions:**

- **Feature docs** (`docs/logic/`): Implementation details, architecture tables, code examples, provider lists, component inventories, API endpoint lists, database schemas
- **Instructions file** (`.github/instructions/`): Rules, conventions, mandatory behaviors, routing tables, references to other files

---

## Phase 3: Generate `.github/instructions/copilot.instructions.md`

The generated file MUST follow this exact structure. Adapt content based on discovered project data.

````markdown
---
description: <Project_Name> project coding guidelines — auto-doc sync for app, features, and infrastructure
applyTo: '*'
---

# <Project_Name> — Copilot Instructions

## Project Overview

<!-- 1-2 sentences describing what the project is. -->

<!-- Bulleted list of key directories with brief descriptions, e.g.: -->

- **App:** `src/app/` — <description>
- **Components:** `src/components/` — <description>
- **Lib:** `src/lib/` — <description>
- **Docs:** `docs/` — Architecture, deployment, and task documentation
- **Data:** `data/` — <description>

## Stack

<!-- Bulleted list of ALL key technologies with versions, e.g.: -->

- **Framework:** Next.js 16 · App Router
- **Runtime:** React 19 · TypeScript 5
- **Database:** better-sqlite3 (SQLite)
  <!-- ... etc -->

## Folder Structure

<!-- ASCII tree of the project structure, 2-3 levels deep for key directories. -->
<!-- Mark important directories with ← comments. -->

---

## MANDATORY: Auto-Update Documentation

Whenever you implement changes to the app, you **MUST** update the relevant doc(s) in `docs/` in the same response as the code change. After the change is tested and confirmed working, update the doc(s) before marking the task complete.

### Documentation Set — What Goes Where

The `docs/logic/` folder contains feature logic docs. Each owns a specific domain. **Always read the relevant file before working on that domain, and update it after making changes.**

| Doc file | Covers | Update when you change… |
|----------|--------|------------------------|
<!-- One row per docs/logic/ file, plus server docs -->

### Quick Triage — Which Doc to Read First

| If this is broken… | Read first |
|---------------------|------------|
<!-- One row per common problem domain, pointing to the right doc -->

---

## Related Instruction Files

Read these **before** working in their domain. They contain mandatory rules:

| When working on… | Read… |
|-------------------|-------|
| **ALL tasks (always read first)** | **`.github/instructions/task.instructions.md`** |
| **Rate limit prevention (always active)** | **`.github/instructions/ratelimitting.instructions.md`** |
| **Pre-commit observations (always active)** | **`.github/instructions/agent-observations.instructions.md`** |
| Frontend layout, UI spacing, form controls | `.github/instructions/layout.instructions.md` |
| Browser verification of UI changes | `.github/instructions/browsetools.instructions.md` |
| Documenting completed work | `.github/prompts/document-task.prompt.md` |
| Planning a task before implementation | `.github/prompts/plan-task.prompt.md` |
| Executing a planned task | `.github/prompts/execute-plan.prompt.md` |
| Generating a report on completed work or a topic | `.github/prompts/report.prompt.md` |

> **`.github/instructions/task.instructions.md` applies to EVERY task, not just "task tracking" work. Read it before starting ANY work.**

---

## Skills Reference

<!-- Only include this section if .github/skills/ exists. -->

When working on **frontend components**, read the relevant skill file for patterns and best practices:

| Skill | File | Use when… |
|-------|------|-----------|
<!-- One row per skill file -->

---

## MANDATORY: Browser Verification Before Completion

After **every** new feature, bug fix, or change that touches the UI or frontend behavior, you **MUST** verify the change works using Chrome DevTools MCP tools before marking it as complete. See `.github/instructions/browsetools.instructions.md` for the full verification protocol.

**Do NOT mark any UI task as done until it has been tested and confirmed working.**

---

## CRITICAL: Always Follow task.instructions.md

Before starting **any** task, you **MUST** read and follow the rules in `.github/instructions/task.instructions.md`. This includes:

- Planning before implementation (Plan Node)
- Writing checklists to `docs/task/todo.md`
- Updating `docs/task/lessons.md` after any user correction
- Verification before marking work complete
- The full Task Management Protocol (blocking gate-checked pipeline)

These rules apply to **every** task — including ad-hoc chat requests, quick fixes, and multi-step implementations. **Never skip this.**

---

## CRITICAL: Document Every Task Before Proceeding

After completing **any** unit of work — even a single sub-task during a multi-step chat conversation — you **MUST** do ALL of the following **before** moving to the next task:

0. **Log agent observations (BLOCKING GATE)** — Before ANY of the steps below, you MUST follow the rules in `.github/instructions/agent-observations.instructions.md`. Review everything you touched, read, or discovered during this task. Append entries to the appropriate log(s) in `docs/agent-observations/` (`critical.md`, `recommendations.md`, `anomalies.md`). If you have zero observations, you MUST explicitly state: "Observations: none." **Silence is a violation — you cannot proceed to step 1 without completing this gate.**
1. **Update the daily task log** — add a row to today's file in `docs/task/logs/YYYY-MM-DD.md` (see Task Ledger format below). Create the file if it doesn't exist yet.
2. **Update affected documentation** — if the work touched app behavior, update the relevant docs per the auto-update rules in this file.

This applies to:

- User-requested tasks given in chat
- Sub-tasks you create for yourself during implementation
- Bug fixes, refactors, and feature additions
- Config changes, data migrations, and script creation

**Never defer documentation.** Never say "I'll update docs later." Never batch documentation updates across multiple tasks. Document each task as it completes.

---

<!-- If git repository detected in Phase 1.1, include this section. Otherwise skip entirely. -->

## CRITICAL: Auto-Commit & Push

After completing **any** unit of work — including documentation updates — you **MUST** commit all changes and push to the default branch. This is the final step of every task.

### Commit Rules

1. **Stage all changed files** — `git add -A`
2. **Write a detailed commit message** — The message must be specific enough that someone can understand exactly what changed without reading the diff. Include:
   - What was done (feature, fix, refactor, config change, data change)
   - Which files/modules were affected
   - Why it was done (if not obvious from the what)
   - Any notable side effects or dependencies
3. **Push to default branch** — `git push origin <default-branch>`
4. **Commit as often as possible** — Every completed task, sub-task, or logical unit of work gets its own commit. Do NOT batch multiple unrelated tasks into one commit.

### Commit Message Format

```
<type>: <concise summary>

<detailed description of what changed and why>

Files: <list key files touched>
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `data`, `config`, `tool`, `chore`

### When NOT to Push

- **Never push to production deployment branches** unless the default branch IS the working branch
- **Never force push** — always use regular `git push`
- If push fails due to remote changes, `git pull --rebase` first, then push

<!-- End conditional git section. -->

---

## MANDATORY: Task Tracking

Follow the full task lifecycle defined in `.github/instructions/task.instructions.md`:

1. Add task item(s) in `docs/task/todo.md` before implementation
2. Create `docs/task/planning/<task-slug>.md` for multi-step tasks
3. Track progress in `docs/task/todo.md`
4. Move completed items out of `docs/task/todo.md`
5. Append a row to today's daily task log `docs/task/logs/YYYY-MM-DD.md`

### Task Ledger — Daily Log Files (`docs/task/logs/YYYY-MM-DD.md`)

Each day's completed work is logged in a daily file. If the file doesn't exist yet, create it with this header:

```markdown
# Task Log — MM/DD/YYYY

> 0 tasks

| Title | Description | Start Date | End Date | Category | Type |
| --- | --- | --- | --- | --- | --- |
```

**Column definitions:**

- `Title` — Action-oriented, specific enough to be searchable (e.g., "Auth Session Fix", not "Fixed bug")
- `Description` — Concise, outcome-focused (what changed, not implementation minutiae)
- `Start Date` / `End Date` — `MM/DD/YYYY` format. For same-day work, both are today.
- `Category` — One of: <TASK_CATEGORIES from Phase 1.5>
- `Type` — `Task` (single-scope) or `Sprint` (grouped multi-item work)

**Grouping guidelines:**

- Prefer feature-focused consolidation over file-by-file rows
- Group related same-scope sub-tasks into a single row when appropriate
- Use `Sprint` type for rows that consolidate multiple related changes
- Keep descriptions outcome-focused: describe the functional scope that changed
- Update the `> {n} tasks` counter in the header after adding rows
- For module/feature rows, describe what changed functionally (e.g., "routing, validation, storage, UI actions"), not generic wording

---

<!-- If UI rendering stack detected in Phase 1.1, include this section. Otherwise skip entirely. -->

## MANDATORY: Element IDs on Every Rendered Element

Every HTML-producing element you create **MUST** have a descriptive `id` attribute. This is a **project-wide standard** with **zero exceptions**.

### Naming Convention

`{page-or-feature}-{section}-{element}` in kebab-case.

- **Page/feature prefix:** the tool, page, or feature name (e.g., `dashboard`, `settings`, `leads-table`, `product-detail`).
- **Section:** logical group (e.g., `header`, `form`, `dialog`, `sidebar`, `toolbar`, `table`).
- **Element:** specific control (e.g., `save-btn`, `name-input`, `row-3`, `select-all`, `title`).

### What Must Have an ID

- **All** container elements (`div`, `section`, `Card`, `CardContent`, `article`, etc.)
- **All** interactive controls (`Button`, `Input`, `Select`, `Checkbox`, `Switch`, `Textarea`, `a`, etc.)
- **All** table parts (`Table`, `TableHeader`, `TableHead`, `TableBody`, `TableRow`, `TableCell`, `<table>`, `<thead>`, `<tr>`, `<td>`)
- **All** dialog/modal parts (`Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogFooter`, modal wrappers)
- **All** form labels (`Label`, `<label>`)
- **All** list items and cards in mapped/looped arrays (use dynamic IDs with the item's unique key)

### Hard Rules

1. **No element may be rendered without an `id`.** This includes server-rendered HTML, client components, and template partials.
2. **Dynamic list items** use the item's unique key in the `id`.
3. **Nested elements** maintain parent hierarchy in their `id` prefix.
4. **Fragments** (`<>...</>` in JSX, or equivalent) that represent meaningful sections must be replaced with a `<div id="...">` wrapper.
5. **If any element is missing an `id`, the code is incomplete and must not be submitted.**

### Stack-Specific Examples

<!-- Generate examples matching the discovered rendering stack. Include ALL that apply: -->

<!-- If JSX/TSX (React, Next.js, Remix, Solid): -->

```tsx
<div id="leads-page">
  <div id="leads-header">
    <Button id="leads-header-add-btn">Add Lead</Button>
  </div>
  <Table id="leads-table">
    <TableHead id="leads-table-th-name">Name</TableHead>
    {leads.map(lead => (
      <TableRow id={`leads-table-row-${lead.id}`} key={lead.id}>
        <TableCell id={`leads-table-cell-name-${lead.id}`}>{lead.name}</TableCell>
      </TableRow>
    ))}
  </Table>
  <DialogContent id="leads-add-dialog">
    <Label id="leads-add-dialog-name-label" htmlFor="lead-name">Name *</Label>
    <Input id="lead-name" value={form.name} />
  </DialogContent>
</div>
```

<!-- If PHP templates (WordPress, vanilla PHP): -->

```php
<div id="<?php echo esc_attr($page_slug); ?>-page">
  <div id="<?php echo esc_attr($page_slug); ?>-header">
    <h1 id="<?php echo esc_attr($page_slug); ?>-header-title"><?php echo $title; ?></h1>
  </div>
  <?php foreach ($items as $item) : ?>
    <div id="<?php echo esc_attr($page_slug . '-item-' . $item['id']); ?>">
      <span id="<?php echo esc_attr($page_slug . '-item-name-' . $item['id']); ?>"><?php echo $item['name']; ?></span>
    </div>
  <?php endforeach; ?>
</div>
```

<!-- If Vue SFC: -->

```vue
<div :id="`${pagePrefix}-page`">
  <div :id="`${pagePrefix}-header`">
    <button :id="`${pagePrefix}-header-add-btn`">Add</button>
  </div>
  <div v-for="item in items" :key="item.id" :id="`${pagePrefix}-item-${item.id}`">
    <span :id="`${pagePrefix}-item-name-${item.id}`">{{ item.name }}</span>
  </div>
</div>
```

<!-- If Laravel Blade: -->

```blade
<div id="{{ $pageSlug }}-page">
  <div id="{{ $pageSlug }}-header">
    <h1 id="{{ $pageSlug }}-header-title">{{ $title }}</h1>
  </div>
  @foreach($items as $item)
    <div id="{{ $pageSlug }}-item-{{ $item->id }}">
      <span id="{{ $pageSlug }}-item-name-{{ $item->id }}">{{ $item->name }}</span>
    </div>
  @endforeach
</div>
```


<!-- Only include examples for stacks actually detected in the project. Do not include irrelevant stacks. -->

---

## Coding Conventions

<!-- Extracted from project analysis. Include sections for: -->

### <Language> (e.g., TypeScript, Swift, Python)

<!-- 3-6 bullet points on language-specific conventions discovered in the codebase. -->

### Components / Views

<!-- 3-6 bullet points on component patterns, directives, etc. -->

### Styling

<!-- 3-6 bullet points on styling approach. -->

### Icons

<!-- Only if relevant. Which icon library, how to import, standard sizes. -->

### File Naming

<!-- Naming conventions discovered in the project. -->

---

## Dev Server & Build

```bash
cd <project-path>
<dev command>        # Dev server on <url>
<build command>      # Production build
<lint command>       # Linter
<other commands>     # Other relevant scripts
```

### Environment Variables

<!-- List required env vars from .env.example or .env -->
````

### Content Rules for the Instructions File

**INCLUDE in instructions (rules & orientation):**

- Project overview, stack, folder structure
- Coding conventions (style rules, naming, patterns)
- Dev/build/deploy commands
- Documentation routing table (pointer to docs)
- Quick triage table
- References to other instruction files and skills
- Mandatory behaviors (doc sync, browser verification, task tracking, task logging)
- Auto-commit & push rules (if git repo detected) with commit message format
- Task ledger format (daily log template, categories, types, grouping guidelines)
- Element ID rules (if UI stack detected) with naming convention and hard rules
- Document-every-task mandate (never defer documentation)

**EXCLUDE from instructions (move to docs/logic/):**

- Feature architecture tables
- API endpoint inventories
- Database schema details
- Component inventories / lists
- Code examples longer than 5 lines
- Provider/integration lists
- Step-by-step "how to add X" guides

---

## Phase 4: Generate `.github/instructions/task.instructions.md`

Only create if it doesn't already exist. Use the blocking gate-checked pipeline template below, adapting project-specific values. Inject the `TASK_CATEGORIES` and `TASK_TYPES` discovered in Phase 1.5.

````markdown
---
description: Task execution rules for <Project Name> engineering work.
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
- [ ] Update relevant `docs/logic/` documentation if the change touches app behavior

**A task is NOT done until all three Step 5 items are checked.**

<!-- If git repository detected in Phase 1.1, include this step. Otherwise skip entirely. -->

### Step 6 — Commit & Push (if git repo exists)
- [ ] Stage all changes: `git add -A`
- [ ] Commit with a detailed message (see commit format in `copilot.instructions.md`)
- [ ] Push to default branch: `git push origin <default-branch>`
- [ ] Every completed task gets its own commit — do NOT batch unrelated work

**A task is NOT done until the commit is pushed.**

<!-- End conditional git step. -->

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
- `Category` — One of: <TASK_CATEGORIES from Phase 1.5>
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
- For UI changes: verify the dev server renders correctly (`<dev command>`)
- For API changes: test endpoints and confirm correct responses
- For database changes: verify migrations and data integrity
- Diff previous behavior vs new behavior when applicable.
- Ask internally: *"Would a staff-level engineer approve this?"*

---

## Autonomous Bug Fixing
- When a bug is reported: fix it directly. Do not ask for hand-holding.
- Identify root causes via logs, errors, and failing tests.
- Resolve without requiring additional user context.
- Run `<lint command>` and fix lint errors proactively.

---

<!-- If UI rendering stack detected in Phase 1.1, include this section: -->

## MANDATORY: Element IDs

Every HTML-producing element **MUST** have a descriptive `id` attribute. The naming convention and hard rules are defined in `copilot.instructions.md` — they apply here as enforcement:

- **No element may be rendered without an `id`.**
- **Code without `id` attributes on every element is incomplete and must not be submitted.**
- **Dynamic list items** use the item's unique key in the `id`.
- **Review all rendered output** before marking a task complete to verify `id` coverage.

<!-- End conditional section. -->

---

## Project-Specific Rules

### Dev Server
- Start: `<dev command>` (runs on `<url>`)
- Build: `<build command>`
- Deploy: `<deploy command>`
- Lint: `<lint command>`

### Path Awareness
- Project root: `<absolute path>`
- App source: `<source dir>`
- Docs: `docs/`
- Tasks: `docs/task/`
- Daily logs: `docs/task/logs/`

### Core Engineering Principles
- **Simplicity First** — Minimal change, minimal surface area.
- **No Laziness** — Root-cause fixes only. No temporary patches.
- **Minimal Impact** — Touch only what is required. Avoid collateral damage.
- **Always Build** — Never leave code unbuilt. Run `<build command>` to verify after significant changes.
- **Stick to the Stack** — Use existing libraries. No new dependencies without explicit approval.

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
<!-- If git repo detected: -->
- **If git repo exists: a completed task without a commit & push is a rule violation.**
<!-- End conditional. -->
````

---

## Phase 5: Generate `.github/prompts/document-task.prompt.md`

Generate this reusable agent prompt file for every project. It allows the user to invoke a documentation pass after completing work in a chat session. Adapt the template based on discovered project data.

````markdown
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

- **Category**: <TASK_CATEGORIES from Phase 1.5, formatted as backtick-separated list>
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

<!-- Generate a bulleted list mapping file areas to docs, based on the project's docs/logic/ files. Example: -->
   - Auth changes → `docs/logic/auth.md`
   - API route changes → `docs/logic/api.md`
   - UI component changes → `docs/logic/components.md`
   - Database schema changes → `docs/logic/database.md`
   <!-- ... one bullet per docs/logic/ file -->

   If no docs need updating (rare), explicitly state why.

5. **Confirm completion** — List what was logged, what observations were recorded (or state "none"), and which docs were updated.

## Rules

- Never defer documentation — do it all now in this response
- Never leave observations undocumented — log them in `docs/agent-observations/` or explicitly state "Observations: none."
- Don't ask for confirmation — just do it
- If unsure which category fits, pick the closest match
- Group same-scope work into single rows; don't create one row per file
````

---

## Phase 6: Generate `.github/prompts/plan-task.prompt.md`

Generate this reusable agent prompt file for every project. It allows the user to invoke a full planning pass before any implementation begins. The plan is written to a physical file and includes a testing plan with browser verification via `browsetools.instructions.md`. Adapt the template based on discovered project data.

````markdown
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
- If this task relates to a design system, admin UI, or layout standardization pass, also review the relevant baseline instruction files before planning.

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
  - `fix-magic-login-routing`
  - `add-backup-retention-ui`
  - `standardize-admin-card-spacing`
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

    1. Navigate to the affected page(s) on `http://127.0.0.1:3028`
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
- For backend-only or non-visual tasks, include an appropriate validation plan instead of forcing irrelevant browser steps.
- Do not write vague implementation steps. "Update the file" is not acceptable. "Add a guard clause in `class-deploy.php` before the rewrite flush call" is acceptable.
- Do not leave testing generic. Define the exact checks required for this task.
- Do not leave rollback generic. State how the change can be reversed.
- Follow `task.instructions.md` and any related project planning, documentation, or workflow rules.
- If baseline instruction files exist for the feature area, use them as the comparison standard when identifying risk, scope, and implementation requirements.
````

---

## Phase 7: Generate `.github/prompts/execute-plan.prompt.md`

Create the file with this template:

````markdown
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
````

---

## Phase 8: Generate `.github/prompts/report.prompt.md`

Generate this reusable agent prompt file for every project. It allows the user to invoke a report generation pass — either to summarize work just completed or to produce a report on a specific topic. Reports are saved to `docs/reports/`.

````markdown
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
| `seo` | SEO audit, meta tag coverage, schema markup analysis |
| `custom` | Anything that doesn't fit the above — describe the type in the report header |

Create a descriptive, searchable title (e.g., "Auth Module Refactor Summary", "SEO Audit — Missing Meta Tags", "API Rate Limiting Architecture Decision").

### 3. Generate the Report File

**File path:** `docs/reports/{YYYY-MM-DD}-{slug}.md`

- `{YYYY-MM-DD}` = today's date
- `{slug}` = kebab-case summary of the report topic (e.g., `auth-refactor-summary`, `seo-audit-missing-meta`, `api-rate-limiting-decision`)

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
````

---

## Phase 9: Generate `.github/instructions/agent-observations.instructions.md`

Always create this file. It applies globally to every chat interaction via `applyTo: '*'`. This file implements a mandatory pre-commit disclosure system for agent observations — anomalies, recommendations, and critical findings.

````markdown
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

### 2. `recommendations.md` — Improvement Suggestions

Things that **aren't broken but should be addressed**. These are follow-up tasks, optimization opportunities, or architectural improvements.

Examples:
- "Consider normalizing data formats across all schemas"
- "The deploy module could benefit from a dry-run mode"
- "3 pages have empty meta descriptions — should be filled"
- "The current caching strategy doesn't account for locale-specific pages"

### 3. `anomalies.md` — Inconsistencies & Drift

Things that are **odd, inconsistent, or divergent** but not necessarily broken.

Examples:
- Documentation says X but code does Y
- A component is documented but never rendered on any page
- Two config values that should match but don't
- A file that appears to be unused but hasn't been verified

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
````

---

## Phase 10: Generate `.github/instructions/browsetools.instructions.md`

Only create if it doesn't already exist. Use this template:

````markdown
---
description: Browser verification rules for <Project Name> using Chrome DevTools MCP
applyTo: "*"
---

# DevTools Verification Rules

## MANDATORY: Browser Verification Before Completion

After **every** new feature, bug fix, or change that touches the UI or frontend behavior, you **MUST** verify the change works using Chrome DevTools MCP tools before marking it as complete.

### Required Verification Steps

1. **Visual verification** — Use `take_screenshot` to confirm the page renders correctly.
2. **Console check** — Use `list_console_messages` to confirm there are no JS console errors.
3. **Functional test** — Use `click`, `fill`, and other interaction tools to test the changed behavior end-to-end.
4. **State check** — Use `evaluate_script` to inspect DOM values or framework state when needed.
5. **Regression check** — Take screenshots of adjacent/related views to ensure no regressions.

### Completion Gate

- Only after successful verification may you mark the task as completed.
- If verification fails, fix the issue and re-verify before completing.
- **Do NOT mark any UI task as done until it has been tested and confirmed working.**

### Dev Server Prerequisite

Before browser verification, ensure the dev server is running:

```bash
cd <project-path> && <dev-command>
```

The app runs at `<dev-url>`. Navigate to the relevant page before taking screenshots or interacting.

Do not kill the server after you finished browsing, keep it alive so that the user can still work on it.

### Authentication Credentials

<!-- If the app requires login, include credentials here. If not, remove this section. -->

The app requires login before accessing any page. Use these credentials:

- **Email:** `<dev-email>`
- **Password:** `<dev-password>`

<!-- If there's a password reset mechanism, include it: -->

If login fails, reset the password by running:

```bash
<password-reset-command>
```

After navigating to `<dev-url>`, if redirected to a login page, fill in the credentials above and submit before proceeding with verification.
````

---

## Phase 11: Generate `.github/instructions/ratelimitting.instructions.md`

Always create this file. It applies globally to every chat interaction via `applyTo: '*'`. This file prevents rate limit failures that would kill the chat session entirely.

````markdown
---
applyTo: '*'
---

# Rate Limit Prevention (CRITICAL)

**You cannot recover from rate limits - the chat session will stop completely. Prevention is essential. Do NOT ask for confirmation - work autonomously but cautiously.**

## Rules

- **Be concise**: Keep responses short and to the point. Avoid verbose explanations.
- **Don't repeat back**: Don't echo the user's request or repeat code they've already shared.
- **Plan first**: Before making ANY tool calls, create a complete mental plan. Identify the minimum number of calls needed.
- **Absolute minimum tool calls**: Only make tool calls that are strictly necessary.
- **Batch aggressively**: Combine multiple operations into single requests wherever possible.
- **Reuse information**: Never re-fetch data you already have from earlier in the conversation.
- **No exploratory calls**: No speculative searches or "just checking" requests.
- **Checkpoint silently**: After significant work, briefly note what's done so progress isn't lost if the session breaks.
- **If uncertain**: Make a reasonable decision and proceed - do not ask me to confirm.
- **Max 2 tool calls per response**: Never make more than 2 tool calls in a single response. Complete the response, then continue in the next one.

## Output Length Limits

**Responses that are too long will be cut off and fail. Stay within limits.**

- **Max ~300 lines per response**: If output would exceed this, split across multiple responses.
- **One file at a time**: When generating or editing multiple files, output one file per response, then continue.
- **Truncate examples**: Show only essential code snippets, not full files unless requested.
- **Use summaries**: For large changes, summarize what was done instead of showing everything.
- **Continue automatically**: If you need to split output, end with `... continuing` and proceed in the next response without asking.

## Visibility

When you take any rate-limit prevention action, output a short status line so I can monitor:

- `📦 BATCH: [what was combined]` - when combining multiple operations
- `♻️ REUSE: [what data]` - when reusing data instead of re-fetching
- `⏭️ SKIP: [what was skipped]` - when skipping an unnecessary call
- `💾 CHECKPOINT: [progress summary]` - when saving progress state
- `✂️ SPLIT: [what's being split]` - when breaking output into multiple responses
````

---

## Phase 12: Create Task Tracking & Observation Files (if missing)

Create these files and directories if they don't exist:

**`docs/task/todo.md`:**

```markdown
# <Project Name> — Active Tasks

<!-- Add task items here as checkboxes. Remove completed items and log them in docs/task/logs/. -->
```

**`docs/task/lessons.md`:**

```markdown
# <Project Name> — Lessons Learned

<!-- Add lessons after user corrections to prevent repeating mistakes. -->
```

Create `docs/task/logs/` directory (for daily task log files).

Create `docs/task/planning/` directory (for multi-step task plans).

Create `docs/agent-observations/` directory with these initial files:

**`docs/agent-observations/critical.md`:**

```markdown
# Critical Issues

| Date | Source | Observation | Impact | Action | Status |
| --- | --- | --- | --- | --- | --- |
```

**`docs/agent-observations/recommendations.md`:**

```markdown
# Recommendations

| Date | Source | Observation | Impact | Action | Status |
| --- | --- | --- | --- | --- | --- |
```

**`docs/agent-observations/anomalies.md`:**

```markdown
# Anomalies

| Date | Source | Observation | Impact | Action | Status |
| --- | --- | --- | --- | --- | --- |
```

Create `docs/agent-observations/closed/` directory (for resolved observation archives).

---

## Phase 13: Verification Checklist

After generating everything, verify:

### Instruction & Prompt Files

- [ ] `.github/instructions/copilot.instructions.md` exists and is < 300 lines
- [ ] `.github/instructions/task.instructions.md` exists with blocking gate-checked pipeline
- [ ] `.github/instructions/agent-observations.instructions.md` exists with observation gate, 3 log types, entry format, and enforcement table
- [ ] `.github/instructions/browsetools.instructions.md` exists with correct dev URL and credentials
- [ ] `.github/instructions/ratelimitting.instructions.md` exists with `applyTo: '*'`
- [ ] `.github/prompts/document-task.prompt.md` exists with correct categories, doc references, and observation step
- [ ] `.github/prompts/plan-task.prompt.md` exists with correct planning dir path, browsetools.instructions.md reference, observation log reading, and testing plan template
- [ ] `.github/prompts/execute-plan.prompt.md` exists with gate-checked execution flow, Chrome MCP testing enforcement, and observation logging
- [ ] `.github/prompts/report.prompt.md` exists with report types table, `docs/reports/` output path, date-prefixed filename convention, observation cross-posting, and user-voice authorship rule

### Documentation Structure

- [ ] `docs/logic/` has one file per identified feature domain
- [ ] `docs/reports/` directory exists
- [ ] `docs/task/todo.md` and `docs/task/lessons.md` exist
- [ ] `docs/task/logs/` directory exists
- [ ] `docs/task/planning/` directory exists
- [ ] `docs/agent-observations/` directory exists with `critical.md`, `recommendations.md`, `anomalies.md`
- [ ] `docs/agent-observations/closed/` directory exists
- [ ] No `done.md` file created (daily logs replace it)

### Content Quality

- [ ] Routing table in copilot instructions covers EVERY `docs/logic/` file
- [ ] Quick triage table covers common failure scenarios
- [ ] All instruction file cross-references are correct
- [ ] Skills references included (if `.github/skills/` exists)
- [ ] No feature logic is inline in the instructions file — all moved to docs/logic/
- [ ] Coding conventions are present (these stay in instructions, they're rules not logic)
- [ ] Dev commands are correct and include the absolute project path

### Task Logging System

- [ ] Task categories are project-specific (not generic placeholders) — at least 5 categories
- [ ] Daily log format matches the 6-column template (Title, Description, Start Date, End Date, Category, Type)
- [ ] `TASK_CATEGORIES` are consistent across copilot.instructions.md, task.instructions.md, and document-task.prompt.md
- [ ] Task lifecycle in task.instructions.md has 4 blocking steps with STOP gates
- [ ] Observation gate exists in task.instructions.md as a blocking step before completion
- [ ] Enforcement section exists in task.instructions.md with non-negotiable rules (including observation violation rule)

### Git & Auto-Commit (conditional)

- [ ] If `.git/` exists: auto-commit & push section is present in copilot.instructions.md
- [ ] If `.git/` exists: Step 5 (Commit & Push) is present in task.instructions.md task lifecycle
- [ ] If `.git/` exists: enforcement section includes commit violation rule
- [ ] If NO `.git/`: auto-commit sections are omitted entirely (not left as empty placeholders)

### Element IDs (conditional)

- [ ] If UI rendering stack detected: Element ID rule is present in copilot.instructions.md with stack-appropriate syntax examples
- [ ] If UI rendering stack detected: Element ID enforcement section exists in task.instructions.md
- [ ] If NO UI rendering stack: Element ID sections are omitted entirely (not left as empty placeholders)

### Agent Observations System

- [ ] `agent-observations.instructions.md` generated with 3 log types (critical, recommendations, anomalies)
- [ ] Observation gate is referenced in `copilot.instructions.md` Step 0 (Document Every Task)
- [ ] Observation gate is referenced in `task.instructions.md` lifecycle (as a blocking step)
- [ ] Observation step exists in `document-task.prompt.md`
- [ ] Observation reading exists in `plan-task.prompt.md` Step 1 (Review Recent Context)
- [ ] Observation reading + logging exists in `execute-plan.prompt.md`
- [ ] Observation cross-posting exists in `report.prompt.md` Step 5
- [ ] User-voice authorship rule exists in `report.prompt.md`
- [ ] `docs/agent-observations/` directory structure created with initial files

---

## Execution Notes

- **Always discover before generating.** Never assume project structure.
- **Preserve existing content.** If docs/logic/ files already exist, read them and update — don't overwrite.
- **If instructions files already exist**, read them first, identify what's inline logic vs. rules, and refactor accordingly.
- **Ask for credentials** if the app has auth and you can't find dev credentials in seeds/fixtures/env.
- **Run the dev server** and verify at least the homepage loads before finishing.
- **Log the setup task** in today's daily task log `docs/task/logs/YYYY-MM-DD.md` when complete.
- **Identify at least 5 project-relevant task categories** during discovery (Phase 1.5). Never use generic placeholders like "Misc" or "Other". Never copy example category lists from the prompt — derive every category from actual project directories, features, and domain concerns.
- **Generate `report.prompt.md` for every project** regardless of stack — report generation is universal. It must save reports to `docs/reports/` with date-prefixed filenames.
- **If the project renders HTML in any form** (JSX, PHP, Vue, Blade, Svelte, Jinja2, EJS, etc.), the Element ID rule **MUST** be generated with stack-appropriate syntax examples.
- **If the project has no HTML rendering** (pure CLI, API-only, library), omit the Element ID sections entirely.
- **Generate `document-task.prompt.md` for every project** regardless of stack — task documentation is universal.
- **Generate `plan-task.prompt.md` for every project** regardless of stack — planning is universal. It must reference `browsetools.instructions.md` for browser verification in the testing plan.
- **Generate `execute-plan.prompt.md` for every project** regardless of stack — plan execution is universal. It must reference `browsetools.instructions.md` for Chrome MCP testing enforcement.
- **Generate `ratelimitting.instructions.md` for every project** regardless of stack — rate limit prevention is universal. It must have `applyTo: '*'` so it applies to every chat interaction.
- **If a `.git/` directory exists at the project root**, generate auto-commit & push rules in `copilot.instructions.md` (CRITICAL section) and `task.instructions.md` (Step 6 in the lifecycle). Detect the default branch name from `git branch --show-current` or `git symbolic-ref refs/remotes/origin/HEAD`. If no `.git/` exists, omit these sections entirely.
- **Generate `agent-observations.instructions.md` for every project** regardless of stack — observation disclosure is universal. It must have `applyTo: '*'` so it applies to every chat interaction. The observation gate is a blocking pre-commit step that runs BEFORE documentation, task logging, and git commits.
- **The observation gate must be integrated into every workflow file.** The `copilot.instructions.md` must reference it as Step 0 in "Document Every Task." The `task.instructions.md` must include it as a blocking step in the task lifecycle. All four prompt files (`document-task`, `plan-task`, `execute-plan`, `report`) must reference the observation logs — either for reading, writing, or both.
- **Reports must use user-voice authorship.** The `report.prompt.md` must instruct the agent to write reports in the user's voice, never referencing Copilot, the agent, or AI. Reports must be shareable as-is with colleagues or management. No "Generated by Copilot" footers or `> **Author:** Copilot` headers.
