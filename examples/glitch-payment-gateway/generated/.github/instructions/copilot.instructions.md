---
description: Glitch WooCommerce Payment Gateway plugin coding guidelines — auto-doc sync for plugin, features, and infrastructure
applyTo: '*'
---

# Glitch WooCommerce Payment Gateway — Copilot Instructions

## Project Overview

A bespoke WooCommerce payment gateway plugin for Glitch, a major South African payment provider. Enables merchants to accept online payments (cards, instant EFT, digital wallets, bank transfers) through WooCommerce while routing transactions through Glitch's payment infrastructure.

- **Plugin Root:** `./` — Main plugin file, bootstrap, constants
- **Includes:** `includes/` — Gateway class, API client, webhook handler, admin settings, payment methods
- **Templates:** `templates/` — Checkout and admin template overrides
- **Assets:** `assets/` — CSS, JS, and image assets for checkout and admin
- **Docs:** `docs/` — Architecture, feature logic, and task documentation
- **Examples:** `examples/` — Project description and reference material

## Stack

- **Platform:** WordPress 6.x · WooCommerce 8.x+
- **Language:** PHP 7.4+ / 8.x
- **Database:** MySQL/MariaDB via WordPress `$wpdb`
- **Package Manager:** Composer (for autoloading and dev dependencies)
- **Payment API:** Glitch Payment Gateway REST API
- **Currency:** ZAR (South African Rand)
- **Admin UI:** WooCommerce Settings API
- **Checkout UI:** WooCommerce checkout hooks + PHP templates
- **HTTP Client:** WordPress HTTP API (`wp_remote_post`, `wp_remote_get`)
- **Logging:** WooCommerce Logger (`wc_get_logger()`)
- **Testing:** PHPUnit (WordPress test suite), WP-CLI for integration testing
- **Linting:** PHP_CodeSniffer with WordPress Coding Standards

## Folder Structure

```
glitch-woocommerce-gateway/
├── glitch-gateway.php              ← Main plugin file (bootstrap)
├── includes/
│   ├── class-glitch-gateway.php    ← WC_Payment_Gateway subclass
│   ├── class-glitch-api.php        ← Glitch API client
│   ├── class-glitch-webhooks.php   ← Webhook handler
│   ├── class-glitch-refunds.php    ← Refund processing
│   ├── class-glitch-methods.php    ← Payment method management
│   └── class-glitch-logger.php     ← Debug logging wrapper
├── templates/
│   ├── checkout/                   ← Checkout form templates
│   └── admin/                      ← Admin settings templates
├── assets/
│   ├── css/                        ← Stylesheets
│   ├── js/                         ← Scripts
│   └── images/                     ← Icons and logos
├── languages/                      ← i18n translation files
├── docs/                           ← Documentation
│   ├── logic/                      ← Feature logic docs
│   ├── reports/                    ← Generated reports
│   ├── server/                     ← Server/deployment docs
│   ├── agent-observations/         ← Agent observation logs
│   └── task/                       ← Task tracking
└── tests/                          ← PHPUnit tests
```

---

## MANDATORY: Auto-Update Documentation

Whenever you implement changes to the plugin, you **MUST** update the relevant doc(s) in `docs/` in the same response as the code change. After the change is tested and confirmed working, update the doc(s) before marking the task complete.

### Documentation Set — What Goes Where

The `docs/logic/` folder contains feature logic docs. Each owns a specific domain. **Always read the relevant file before working on that domain, and update it after making changes.**

| Doc file | Covers | Update when you change… |
|----------|--------|------------------------|
| `docs/logic/gateway.md` | Core gateway class, checkout flow, payment initiation | Gateway class, checkout hooks, redirect/return URL handlers |
| `docs/logic/transactions.md` | Transaction lifecycle, statuses, refunds, chargebacks | Order status transitions, refund logic, chargeback handling |
| `docs/logic/webhooks.md` | Webhook processing, signature verification, order sync | Webhook endpoint, callback handlers, verification logic |
| `docs/logic/admin-settings.md` | Admin settings page, credentials, sandbox/production toggle | Settings fields, option storage, environment switching |
| `docs/logic/security.md` | Signature verification, nonces, anti-spoofing, credential storage | Any security-related code, HMAC validation, sanitization |
| `docs/logic/api-integration.md` | Glitch API communication, requests, responses, error handling | API client class, endpoint URLs, request/response formats |
| `docs/logic/payment-methods.md` | Supported payment methods, enable/disable, checkout display | Payment method registration, icons, labels, conditional rendering |

### Quick Triage — Which Doc to Read First

| If this is broken… | Read first |
|---------------------|------------|
| Checkout not completing / payment not initiating | `docs/logic/gateway.md` |
| Order status wrong after payment | `docs/logic/transactions.md` |
| Webhooks not arriving or failing | `docs/logic/webhooks.md` |
| Admin settings not saving or displaying | `docs/logic/admin-settings.md` |
| Signature verification failing | `docs/logic/security.md` |
| API calls returning errors or timing out | `docs/logic/api-integration.md` |
| Payment methods not showing at checkout | `docs/logic/payment-methods.md` |
| Refund not processing | `docs/logic/transactions.md` then `docs/logic/api-integration.md` |

---

## Related Instruction Files

Read these **before** working in their domain. They contain mandatory rules:

| When working on… | Read… |
|-------------------|-------|
| **ALL tasks (always read first)** | **`.github/instructions/task.instructions.md`** |
| **Rate limit prevention (always active)** | **`.github/instructions/ratelimitting.instructions.md`** |
| **Pre-commit observations (always active)** | **`.github/instructions/agent-observations.instructions.md`** |
| Browser verification of UI changes | `.github/instructions/browsetools.instructions.md` |
| Documenting completed work | `.github/prompts/document-task.prompt.md` |
| Planning a task before implementation | `.github/prompts/plan-task.prompt.md` |
| Executing a planned task | `.github/prompts/execute-plan.prompt.md` |
| Generating a report on completed work or a topic | `.github/prompts/report.prompt.md` |

> **`.github/instructions/task.instructions.md` applies to EVERY task, not just "task tracking" work. Read it before starting ANY work.**

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
2. **Update affected documentation** — if the work touched plugin behavior, update the relevant docs per the auto-update rules in this file.

This applies to:

- User-requested tasks given in chat
- Sub-tasks you create for yourself during implementation
- Bug fixes, refactors, and feature additions
- Config changes, data migrations, and script creation

**Never defer documentation.** Never say "I'll update docs later." Never batch documentation updates across multiple tasks. Document each task as it completes.

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

- `Title` — Action-oriented, specific enough to be searchable (e.g., "Webhook Signature Verification", not "Fixed bug")
- `Description` — Concise, outcome-focused (what changed, not implementation minutiae)
- `Start Date` / `End Date` — `MM/DD/YYYY` format. For same-day work, both are today.
- `Category` — One of: `Gateway`, `Transactions`, `Webhooks`, `Admin`, `Security`, `API`, `PaymentMethods`, `Bug`, `Docs`
- `Type` — `Task` (single-scope) or `Sprint` (grouped multi-item work)

**Grouping guidelines:**

- Prefer feature-focused consolidation over file-by-file rows
- Group related same-scope sub-tasks into a single row when appropriate
- Use `Sprint` type for rows that consolidate multiple related changes
- Keep descriptions outcome-focused: describe the functional scope that changed
- Update the `> {n} tasks` counter in the header after adding rows
- For module/feature rows, describe what changed functionally (e.g., "refund processing, status sync, admin notifications"), not generic wording

---

## MANDATORY: Element IDs on Every Rendered Element

Every HTML-producing element you create **MUST** have a descriptive `id` attribute. This is a **project-wide standard** with **zero exceptions**.

### Naming Convention

`{page-or-feature}-{section}-{element}` in kebab-case.

- **Page/feature prefix:** the tool, page, or feature name (e.g., `glitch-checkout`, `glitch-settings`, `glitch-refund`).
- **Section:** logical group (e.g., `form`, `table`, `notice`, `methods`, `fields`).
- **Element:** specific control (e.g., `submit-btn`, `api-key-input`, `sandbox-toggle`, `method-card`).

### What Must Have an ID

- **All** container elements (`div`, `section`, `table`, `fieldset`, etc.)
- **All** interactive controls (`input`, `select`, `button`, `textarea`, `a`, etc.)
- **All** table parts (`table`, `thead`, `tr`, `td`, `th`)
- **All** form labels (`label`)
- **All** list items in loops (use dynamic IDs with the item's unique key)

### Hard Rules

1. **No element may be rendered without an `id`.** This includes admin settings pages, checkout templates, and all PHP template partials.
2. **Dynamic list items** use the item's unique key in the `id`.
3. **Nested elements** maintain parent hierarchy in their `id` prefix.
4. **If any element is missing an `id`, the code is incomplete and must not be submitted.**

### PHP Template Examples

```php
<div id="<?php echo esc_attr( $gateway_id ); ?>-checkout-form">
  <div id="<?php echo esc_attr( $gateway_id ); ?>-checkout-methods">
    <?php foreach ( $payment_methods as $method ) : ?>
      <div id="<?php echo esc_attr( $gateway_id . '-method-' . $method['id'] ); ?>">
        <label id="<?php echo esc_attr( $gateway_id . '-method-label-' . $method['id'] ); ?>"
               for="<?php echo esc_attr( $gateway_id . '-method-radio-' . $method['id'] ); ?>">
          <input type="radio"
                 id="<?php echo esc_attr( $gateway_id . '-method-radio-' . $method['id'] ); ?>"
                 name="payment_method"
                 value="<?php echo esc_attr( $method['id'] ); ?>" />
          <?php echo esc_html( $method['label'] ); ?>
        </label>
      </div>
    <?php endforeach; ?>
  </div>
  <button id="<?php echo esc_attr( $gateway_id ); ?>-checkout-submit-btn" type="submit">
    <?php esc_html_e( 'Pay Now', 'glitch-gateway' ); ?>
  </button>
</div>
```

---

## Coding Conventions

### PHP & WordPress

- Follow **WordPress Coding Standards** (WPCS); `snake_case` functions/variables, `UPPER_SNAKE_CASE` constants
- Prefix all functions, classes, and hooks with `glitch_` or `Glitch_`
- Sanitize input (`sanitize_text_field`, `absint`, `esc_attr`) and escape output (`esc_html`, `esc_attr`, `esc_url`)
- Use WordPress nonces for forms/AJAX; `wp_remote_post`/`wp_remote_get` for HTTP — never raw `curl`
- Extend `WC_Payment_Gateway`; use WooCommerce Settings API for admin config
- Use `WC_Order` methods for status transitions — never update `post_status` directly
- Support HPOS: use `$order->get_meta()` / `$order->update_meta_data()` instead of `get_post_meta`
- Log via `wc_get_logger()` with `glitch-gateway` source context

### Templates & Assets

- Templates in `templates/` using `wc_get_template()` for theme overrides; keep templates logic-light
- CSS classes prefixed with `glitch-`; enqueue assets only on relevant pages
- File naming: `class-glitch-{name}.php`, `glitch-{name}.{css|js}`, lowercase with hyphens

---

## Dev Server & Build

```bash
cd /path/to/glitch-woocommerce-gateway
wp-env start              # WordPress dev environment
composer test             # PHPUnit tests
composer phpcs            # Lint with WPCS
composer phpcbf           # Auto-fix lint issues
wp plugin activate glitch-woocommerce-gateway
```

### Environment Variables

`GLITCH_API_KEY`, `GLITCH_API_SECRET`, `GLITCH_MERCHANT_ID`, `GLITCH_SANDBOX_MODE`, `GLITCH_WEBHOOK_SECRET`, `GLITCH_API_URL`
