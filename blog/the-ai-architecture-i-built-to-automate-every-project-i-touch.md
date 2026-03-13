# The AI Architecture I Built to Automate Every Project I Touch

Most developers treat AI agents like disposable interns. I built a system that turns them into disciplined engineers who plan, track, document, and verify every step.

> This article accompanies the [`project-rules-generator`](https://github.com/savedpixel/project-rules-generator) repository and explains the thinking behind the generator, the workflow design, and the generated project structure.

## The cycle every developer is stuck in

Open a session. Re-explain the project. Set the rules. Get good work. Close the session. Open a new one. Start from zero.

If you have used AI agents on anything bigger than a weekend project, you know this loop. Everyone does. It is not some niche complaint. It is the default experience for every developer building with AI right now, and they talk about it constantly — on Reddit, on Twitter, in every "AI tools" thread that exists. The agents forget. The documentation drifts. The codebase grows and nobody can trace what changed or why.

I lived it for months. I would start a session with Copilot, walk it through the project structure, set the conventions, and the agent would do solid work. Next session, all of that context was gone. Same conventions, same file structure, same coding standards — re-explained from scratch, every single time. On a small project, that is annoying. On a real production codebase with dozens of files and active feature work, it is a disaster.

Most people blame the wrong thing. They assume the model is the bottleneck — not smart enough, context window too small, tool not ready yet. That is wrong. The bottleneck is not the AI. It is the absence of structure. No persistent rules. No documentation that auto-updates. No task tracking that carries forward. You give the agent a task, it does the task, and everything it learned about your project evaporates. You restart from nothing every time and hope the agent figures it out.

I stopped hoping. I built a system that fixes this entirely. One markdown file that I feed to my agent at the start of any project, new or existing. The agent reads it, analyzes my project, and generates a complete set of instruction files, reusable prompts, task tracking structures, and documentation scaffolding. From that point forward, every agent session follows the same rules, tracks every task, updates documentation automatically, and verifies its own work before calling anything done.

This is how I actually control my agents. And it changed everything about how I ship.

## Why Your AI Agent Keeps Forgetting Everything

Think of it this way. You hire a new developer and drop them into your codebase on day one with no onboarding. No README. No code conventions document. No explanation of how deployments work or where the important files live. You just say "build this feature" and walk away.

That developer might be talented, but they are going to make mistakes that anyone with context would avoid. They will name files wrong. They will put code in the wrong directory. They will skip your testing requirements because nobody told them testing was required. Every decision they make is a guess because nobody gave them the rules.

That is exactly what happens when you use an AI agent without project-level instructions.

When you open a Copilot chat and start giving instructions with no rules in place, the agent is that new hire with no onboarding. It does not know your naming conventions. It does not know where documentation lives. It does not know that you require verification before marking a task done. It does not know that every completed task needs a log entry. It just executes whatever you ask, in whatever way it sees fit, and hopes that matches what you wanted.

On a small project, this works well enough. You hold the entire codebase in your head and catch inconsistencies manually. But the moment your project hits real complexity — multiple feature domains, shared components, a real deployment pipeline, documentation that matters — the lack of structure starts compounding. The agent creates files in random places. It does not update docs after making changes. It does not track what it modified or why. Three months later, you have a codebase full of inconsistencies that nobody can trace back to their origin because nothing was ever logged.

The fix is not waiting for a smarter model. The fix is rules.

## The System: One File That Generates Everything

I wrote a single markdown file that I call my **project instructions generator**. It is roughly 1500 lines of structured instructions that tell my agent exactly how to analyze any project and produce a complete rule system for it. I give the agent this file alongside a description of whatever project I am working on, and it builds the entire scaffolding automatically.

The core idea is what I call a **pointer-based instructions system**. Instead of cramming everything into one massive instructions file that the AI has to parse every session, the main instructions file stays slim — around 150 to 250 lines — and points to dedicated documentation files for each feature domain. The main file says "when working on authentication, read `docs/logic/auth.md` first." The domain file contains the actual architecture decisions, implementation details, and patterns for that feature.

Why does this matter? AI agents have context windows. Every token you feed them costs capacity. A 2000-line instructions file burns through context that your actual code needs. A 200-line pointer file with references to domain docs means the agent only loads the details relevant to the current task. Everything else stays out of the way until it is needed.

Here is what the generator produces when it runs:

**Five instruction files** — the persistent rules:

- **`copilot.instructions.md`** — the main project instructions. Contains the stack, folder structure, coding conventions, documentation routing table, and pointers to every domain doc and prompt file. This is what the agent reads on every session.
- **`task.instructions.md`** — a blocking gate-checked pipeline that governs how every single task gets executed. Plan it, implement it, verify it, document it, log it. No shortcuts allowed.
- **`agent-observations.instructions.md`** — a mandatory disclosure system that forces the agent to log anomalies, critical issues, and recommendations before it can commit any work. No burying findings. No silent omissions.
- **`browsetools.instructions.md`** — rules for browser-based verification using Chrome DevTools MCP tools. When the agent changes UI, it has to actually open a browser and check its own work.
- **`ratelimitting.instructions.md`** — rate limit prevention rules that keep the agent from burning through tool calls unnecessarily.

**Four prompt files** — reusable agent commands:

- **`plan-task.prompt.md`** — forces the agent to create a detailed implementation plan before writing a single line of code. Every step must be specific, numbered, and verifiable.
- **`execute-plan.prompt.md`** — forces the agent to follow a previously created plan step by step. No deviations. No improvising. No adding scope.
- **`document-task.prompt.md`** — triggers a full documentation pass. The agent scans the conversation, identifies what was done, updates the daily task log, and syncs affected domain docs.
- **`report.prompt.md`** — generates structured reports on completed work or a specific topic.

**A complete task tracking system:**

- **`todo.md`** — the active task checklist. Every task starts here before implementation begins.
- **`lessons.md`** — every correction I make gets logged with a rule preventing the same mistake. The agent improves over the project's lifetime.
- **Daily task logs** — every completed task gets a dated entry. After two weeks, I have a complete record of everything that changed.
- **Planning files** — multi-step tasks get their own plan document with steps, risks, testing requirements, and rollback instructions.

**Documentation scaffolding:**

- **`docs/logic/`** — one file per feature domain. Authentication, database, API, components — each gets its own doc with architecture, key behaviors, and common patterns.

One generator file produces all of this. Customized for whatever project it analyzes.

The observation logs are new and worth calling out separately. The generator creates a `docs/agent-observations/` directory with three log files — `critical.md` for blocking issues and security concerns, `recommendations.md` for improvement suggestions, and `anomalies.md` for inconsistencies and drift. Open items live in the root files. Resolved items get moved to a `closed/` subdirectory. Every prompt file in the system cross-references these logs — the plan prompt reads them before planning, the execute prompt checks them during execution, the document prompt writes to them during completion, and the report prompt cross-posts findings into them. It is a closed feedback loop that ensures nothing gets lost between sessions.

## How It Works: From Zero to Fully Automated

The process takes two inputs and produces one output.

**Input one:** a markdown document where I describe my project. I write about what I am building, the technology stack, the folder structure, the key features, any conventions I care about. This does not need to be formal or perfectly structured. I write it the same way I would explain the project to a senior developer joining my team. A few solid paragraphs covering the important parts.

**Input two:** my generator file. This is the instruction set that tells the agent exactly how to analyze the project and what to produce.

**Output:** a complete, tailored instructions system ready to use immediately.

Here is what happens under the hood when the agent processes both documents:

### Phase 1: Discovery

The agent does not generate anything yet. It learns first.

It scans the project directory structure to identify the framework, language, and tooling. It checks for dependency files — `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, whatever exists. It reads the README. It identifies the dev server command, build command, test command, and deploy command. It captures the specific runtime versions, environment variables, and port configurations so the instructions reference real values, not placeholders. It maps out the feature domains by analyzing how the source code is organized.

This discovery phase is what separates my approach from a generic template. A Next.js app gets different conventions than a Django project. A WordPress theme gets different rules than a Rust CLI tool. The agent figures out what it is dealing with and configures the output to match.

The discovery also checks for existing work. If `docs/` already exists with domain files, it maps to them instead of overwriting. If `.github/instructions/` already has files, it updates them rather than creating duplicates. If task tracking already exists, it respects the current format.

This is critical. Most people thinking about AI instructions imagine a one-time setup. My system runs on existing projects with months of established conventions and does not destroy what is already there.

### Phase 2: Documentation Structure

Based on what it discovered, the agent sets up the documentation scaffolding. It creates the `docs/logic/` directory with one file per feature domain. It creates the task tracking structure. It sets up the `docs/agent-observations/` directory with the three observation log files and a `closed/` subdirectory for resolved items. It ensures the reporting and planning directories exist.

Each domain doc follows a consistent template: a brief description, architecture section listing key files and directories, key behaviors explaining how the feature works, and common patterns documenting code conventions specific to that domain. This consistency means any developer — human or AI — can find what they need in a predictable location.

### Phase 3: Generate the Instructions

This is where everything comes together. The agent populates each file with project-specific content:

The main instructions file gets the real stack with actual version numbers. The real folder structure as an ASCII tree. The real coding conventions discovered from existing code. A documentation routing table mapping every domain doc to its area of responsibility.

The task lifecycle file gets categories derived from the project's actual feature domains — not generic labels. If the project has an `auth/` directory, it gets an "Auth" category. If it has a `billing/` module, it gets "Billing." Every category maps to a real, observable concern in the codebase.

Everything references everything else. The main instructions point to every domain doc and prompt file. The task instructions reference the daily log format. The plan prompt references the task instructions. The execute prompt references the plan format. It is a closed, self-consistent system.

## What Gets Generated and Why Each Piece Matters

Let me walk through the key artifacts and explain why each one exists in the system.

### The Main Instructions File

I already covered what goes into this file — the stack, the folder structure, the conventions, the routing table. What matters more is what it deliberately leaves out.

It does not contain API endpoint lists, database schemas, component inventories, or step-by-step implementation guides. Those live in the domain docs. The instructions file is a routing layer that tells the agent where to find details, not a dump of every detail.

This is the critical design decision. Most people who try writing AI instructions end up with a massive file that covers everything — every endpoint, every schema, every convention, every exception. The pointer-based approach avoids this entirely. The main file stays under 250 lines. The agent loads only what it needs for the current task. Everything else stays out of the way.

### The Task Lifecycle Rules

This is the piece that transforms how agents work. Without it, an agent completes a task and says "done." With it, the agent cannot say "done" until it passes a multi-step verification gate.

Think of it like a pre-flight checklist for a pilot. The plane might be perfectly fine, but the pilot runs through every item on the checklist before takeoff anyway. Every time. No exceptions. That consistency is what prevents the rare catastrophic failure.

Here is the full pipeline:

**Before writing any code:**
1. Add the task to `todo.md`
2. If it is a multi-step task, create a plan file with numbered steps
3. Write a brief plan before touching any implementation

**During implementation:**
4. Keep progress updated in `todo.md` as sub-tasks complete
5. If new information invalidates the plan, stop and re-plan immediately

**After implementation, before marking complete:**
6. Verify the change actually works — build it, run it, test it, check it in a browser
7. For UI changes, take a screenshot to confirm the rendering is correct

**Log observations — this is a blocking gate:**
8. Review everything touched, read, or discovered during the task
9. Append entries to the observation logs — critical issues, recommendations, or anomalies
10. If there are no observations, explicitly state that. Silence is a violation.

**Completion — all five are required:**
11. Remove the finished item from `todo.md`
12. Add a row to today's daily task log with the title, description, date, and category
13. Update the relevant domain documentation if the change touched feature behavior
14. If the project uses git, commit the changes with a structured message
15. Verify that observations were logged or explicitly declared as none

Every single step is mandatory. If the agent skips step 12, the task is not complete. If it forgets step 13, the task is not complete. If it skips the commit in step 14, the task is not complete. If it never logged observations in step 9, the task is not complete. This is what makes it a **blocking gate-checked pipeline**. You cannot advance past a gate until every requirement at that gate is satisfied.

Is this rigid? Yes. That rigidity is the entire point.

The observation gate — steps 8 through 10 — is one I added after watching agents bury important findings for weeks. An agent would notice a data inconsistency or a security misconfiguration while working on something else and just move on. It would not tell me. It would not log it anywhere. The information disappeared into the context window and I never knew it existed. The observation gate forces disclosure. Before the agent can finish a task, it has to explicitly review everything it encountered and log anything noteworthy — or declare that it found nothing. Either way, the act of disclosure is mandatory. No silent omissions.

Step 14 is one that surprised me with how much it matters. The generator detects whether a project uses git during the discovery phase. If it does, every completed task ends with a commit. Not a manual "remember to commit" reminder — an automatic commit with a structured message format. The commit type is derived from the work: `feat` for new features, `fix` for bug fixes, `docs` for documentation, `refactor` for restructuring, `config` for configuration changes. The message includes the scope and a concise description. After a day of work, the git history reads like a changelog — not a mess of "fixed stuff" and "WIP" commits that tell you nothing.

For complex tasks — the kind with ten moving parts across multiple files — the system decomposes aggressively. Instead of one massive instruction where the agent juggles everything at once, the work gets split into focused sub-tasks. One task per concern. One concern per execution. The agent tackles them sequentially, each one passing through the same pipeline. This prevents the most common failure mode I see with AI agents: trying to do too much in one shot and producing a tangled mess where three different changes interfere with each other.

### The Prompt Files

These are reusable commands I invoke when I need a specific workflow.

When I say "plan this task," the plan prompt forces the agent to create a physical plan file with defined sections: context explaining why the task exists, current state of the affected files, target state describing the desired outcome, explicit acceptance criteria defining what "done" looks like, an out-of-scope list preventing scope creep, numbered implementation steps where each step has specific files, a specific change, and a specific rationale, a testing plan with browser verification steps, and a rollback plan explaining how to undo if something goes wrong. Before any of that, the agent reads the observation logs to check for unresolved issues that might affect the planned work. A critical observation from three tasks ago might completely change the approach.

Every step has to be verifiable. "Update the file" is not a valid step — that tells you nothing about what changed or how to confirm it. "Add rate limiting middleware to the `/api/orders` endpoint with a 100 requests/minute threshold and verify by sending 101 requests in 60 seconds" is a valid step. Specific. Measurable. Testable.

When I say "execute this plan," the execute prompt forces the agent to follow the plan exactly as written. No reordering steps. No skipping steps. No adding scope. No "improvements" that were not in the plan. If a step is blocked, it stops and tells me instead of improvising a workaround.

When I say "document what we just did," the document prompt scans the conversation, identifies every change made, updates the daily task log, and syncs any domain docs that were affected by the work.

When I say "generate a report," the report prompt produces a structured document saved to `docs/reports/` with a date-prefixed filename. It supports different report types — work summaries, feature reports, architecture overviews, audit reports, migration plans, incident postmortems, and status updates. Each type has a specific structure and purpose. A work summary covers everything completed in a given timeframe. A feature report documents the current state and recent changes to a specific domain. An architecture overview maps the system's structure and key decisions. I use these when I need to communicate progress to stakeholders, onboard someone to a feature, or get a clear snapshot of where things stand. The agent writes the report from the project's actual state — not from memory, not from assumptions, but from the documentation and logs that the system has been maintaining all along.

One design decision here matters more than you would expect. The report prompt requires the agent to write in my voice. First person. As if I did the work and I am presenting the findings. It never references Copilot, the agent, or AI anywhere in the report. No "Generated by Copilot" footers. No "the user requested" framing. The report reads like I wrote it, because I need to be able to send it directly to a colleague or a stakeholder without editing a single line. That sounds like a small detail, but it eliminates an entire round of cleanup that would otherwise happen every time I generate a status update or an audit.

### The Daily Task Logs

This is the accountability layer that most people never think to build. Every completed task gets a dated entry in a log file:

```markdown
# Task Log — 03/07/2026

> 3 tasks

| Title | Description | Start Date | End Date | Category | Type |
| --- | --- | --- | --- | --- | --- |
| Auth Session Fix | Fixed session expiry bug in JWT refresh flow | 03/07/2026 | 03/07/2026 | Auth | Task |
| Product API Refactor | Restructured product endpoints for v2 schema | 03/07/2026 | 03/07/2026 | API | Task |
| Dashboard Charts | Added revenue charts to admin dashboard | 03/07/2026 | 03/07/2026 | UI | Task |
```

After two weeks of development, I have a complete record of every change, when it happened, what feature domain it belonged to, and whether it was a single task or part of a larger sprint. Six months later, when something breaks, I can trace back through the logs and find exactly when a change was introduced. No guessing. No git-blaming through a hundred commits hoping to find the culprit.

This is the kind of documentation discipline that most teams cannot maintain manually. People forget. People get lazy. People say "I will update the log later" and then never do. The agent does it automatically because the rules require it and the rules are enforced by the gate-checked pipeline. No log entry, no completion. It is that simple.

### The Lessons File

Every time I correct the agent — fix a wrong assumption, point out a missed convention, catch a repeated mistake — the correction gets logged in `lessons.md` with a rule that prevents the same error from happening again.

This means the agent's behavior improves over the life of the project. A mistake made in week one never recurs in week ten because the rule is physically embedded in the project's knowledge base. The agent reads the lessons file, sees the rule, and avoids the trap. It is a self-improvement loop that compounds over time.

### The Agent Observations System

This is the feature that solved a problem I did not even realize I had until it was too late.

Agents notice things. While working on one task, they read through code and data that has nothing to do with the current work. They spot inconsistencies. They see security misconfigurations. They find documentation that contradicts the actual implementation. And then they do absolutely nothing about it. The information enters the context window, gets processed, and vanishes when the session ends. Nobody told the agent to report what it found, so it does not.

I lost weeks of potential fixes because of this. An agent would be refactoring a component and in the process it would read through a database schema that had orphaned references, or it would notice that two configuration values that should match did not. It never flagged any of it. It just completed its task and moved on. The problems stayed hidden until something actually broke and I had to chase it down manually.

The agent observations system is a mandatory disclosure protocol. The generator creates three log files: `critical.md` for things that are broken, dangerous, or require immediate attention. `recommendations.md` for things that are not broken but should be addressed — optimization opportunities, follow-up tasks, architectural improvements. `anomalies.md` for things that are odd, inconsistent, or divergent but not necessarily broken — documentation that says one thing while the code does another, components that are documented but never rendered, configuration values that should match but do not.

Each log uses a structured table format with the date, what triggered the observation, what was found, why it matters, and what should be done about it. Open items live in the root `docs/agent-observations/` directory. When an observation gets resolved, it moves to a `closed/` subdirectory. The root files only ever contain active items, so I can glance at them and immediately see what needs attention.

The observation gate in the task lifecycle is what makes this work. The agent cannot finish a task without reviewing everything it touched and explicitly logging observations or declaring that it found none. Silence is a rule violation. This is not a suggestion or a nice-to-have — it is a hard gate that blocks completion. The agent must disclose before it can document, log, or commit.

Every prompt file cross-references these logs. The plan prompt reads them before planning, so unresolved critical issues get factored into the next task's scope. The execute prompt checks them during execution. The document prompt writes to them. The report prompt cross-posts its findings into the appropriate log. It is a closed loop — observations flow in from every workflow and never get lost between sessions.

The result is that I now have a persistent, always-current inventory of everything the agent has noticed across every task it has ever worked on. Patterns emerge. When three different tasks produce anomaly entries about the same data model, that tells me something systemic needs attention — even if no individual task flagged it as critical.

## The Gate-Checked Pipeline: Why Every Step Is Tracked

Big projects fail in a predictable and frustrating way. Someone makes a change but does not document it. Someone else makes a conflicting change because they did not know about the first one. A bug gets introduced, and nobody can figure out when or why because there is no record. The codebase does not accumulate technical debt from bad code. It accumulates debt from bad process.

I have seen this pattern in teams of twenty developers and in solo projects where I was the only person working. The scale does not matter. If you are not tracking what changed, when it changed, and why it changed, you will eventually hit a wall where the project outgrows your ability to keep it in your head.

The gate-checked pipeline is my direct answer to this. Every task passes through the same checkpoints I described earlier — plan, implement, verify, disclose observations, document, log, commit. No exceptions. No batching documentation updates for later. Each task completes fully before the next one starts.

Underneath the pipeline, four engineering principles govern every decision the agent makes. Simplicity first — the minimum change needed, nothing more. No laziness — root-cause fixes only, no temporary patches or workarounds. Minimal impact — touch only what is required, avoid collateral damage to surrounding code. Stick to the stack — use existing tools and dependencies, no adding new libraries without explicit approval. These are not aspirational guidelines. They are enforced constraints baked into the rules the agent reads at the start of every session.

Two things happen when the agent follows these rules consistently:

First, every change becomes traceable. I can look at any daily log and see exactly what happened on that date. I can read a domain doc and trust that it reflects the current state of that feature, because the agent is required to update it after every change. The documentation never drifts from reality because the rules make drift impossible.

Second, the agent makes fewer mistakes over time. The lessons file captures every correction. The planning process catches edge cases before implementation starts. The verification gate catches bugs before they get marked as done. The system is self-correcting. It gets better the longer you use it.

## New Project or Existing — It Works Either Way

One constraint I set when building this system was that it had to work on day one of a brand new project and on day three hundred of an existing one. No special setup required either way.

For a new project, the discovery phase finds a mostly empty structure. Maybe a `package.json`, a README, a handful of source files. The generator creates everything from scratch — full documentation scaffolding, all instruction files, all prompt files, task tracking, the complete system.

For an existing project, the discovery phase maps to whatever already exists. It detects existing documentation, instruction files, and task tracking and integrates with them rather than overwriting — the same preservation behavior I described in the discovery phase. Nothing gets destroyed. The generator adapts to what is already there.

This adaptability matters because most developers do not start projects on a clean slate. They have codebases that are weeks or months old, with documentation that has evolved organically, conventions that emerged through practice, and structures that grew out of real decisions. The generator adapts to that reality instead of demanding you throw it away and start fresh.

The technology-specific configuration is equally adaptive. A React project gets component conventions and mandatory element ID rules — every rendered HTML element gets an `id` attribute following a `{page}-{section}-{element}` naming convention, which means the agent can target any element precisely during browser verification instead of relying on fragile CSS selectors or XPath queries. A PHP project gets template-specific patterns. A Python CLI with no user interface gets none of those rules — the irrelevant sections are simply omitted. Nothing generic. Nothing boilerplate. Everything maps to what actually exists.

## The Before and After

Before this system, my sessions started with setup and ended with amnesia. Every conversation was a blank slate. I was the memory, the documentation system, and the quality gate — all at once, every time.

Now I assign a task and walk away. The agent handles the rest — planning, implementation, verification, documentation, logging, committing. I come back to a completed task with a paper trail. If I correct something, the correction sticks permanently. If a bug surfaces, the agent traces it through the domain docs and logs, fixes it, and moves on without waiting for me to diagnose it first.

Most of my time now goes into thinking about how a feature should actually work. The logic. The integration points. The edge cases. I write that out as a plan and hand it to the agent. After ten years of building software, I know how systems should be structured — and over the last four years of working with AI, I have learned that when I describe the implementation clearly enough, the agent executes it exactly the way I would have built it myself. The bottleneck was never the coding. It was always the thinking. Now the thinking is all I do, and the system handles everything else.

The difference is not incremental. It is structural. I stopped being the project's memory and started being the person who decides what gets built and how.

## Start Here

Building this system was the moment AI agents stopped being something I managed and started being something that managed itself within my projects. The problem was never the model's intelligence. It was never the context window size. It was the absence of structure and rules.

Most developers do not realize that the exact same AI they are frustrated with can become a disciplined, self-documenting, self-verifying engineering partner. All it takes is giving it structure upfront. A set of persistent instructions. A task lifecycle with mandatory checkpoints. A documentation convention that auto-updates. A verification protocol that catches mistakes before they ship.

The AI does not resist rules. It thrives on them.

If you have been struggling with agents that lose context, skip documentation, or produce inconsistent work across sessions, try this approach on your next project. Write a description of what you are building. Create a generator prompt that tells the agent how to analyze your project and produce structured rules. Feed both documents to the agent and let the automation build the system that keeps everything on track from that point forward.

Once the rules are in place, you stop managing the AI and start building with it. That is the real shift.

I told you I would give it to you. Here it is. The full generator file — all 1500 lines — is on my GitHub. Fork it, adapt it to your stack, and let it build the system for your project.

**[Get the generator file on GitHub →](https://github.com/savedpixel/project-rules-generator)**

---

## Repository Contents

- [`generate-project-rules.prompt.md`](https://github.com/savedpixel/project-rules-generator/blob/main/generate-project-rules.prompt.md)
- [`examples/glitch-payment-gateway/project-description.md`](https://github.com/savedpixel/project-rules-generator/blob/main/examples/glitch-payment-gateway/project-description.md)
- [`examples/glitch-payment-gateway/generated/`](https://github.com/savedpixel/project-rules-generator/tree/main/examples/glitch-payment-gateway/generated)

For the Related Reading section, use this cleaner GitHub-friendly version:

## Related Reading

- [How I Set Up My AI Workflow and Use It to Ship Production Software](https://byronjacobs.com/how-i-ship)
- [Vibe Coding Without Getting Hacked: A Practical How to & Security Checklist](https://byronjacobs.com/vibe-code-like-a-pro)
