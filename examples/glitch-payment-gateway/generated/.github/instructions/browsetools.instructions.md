---
description: Browser verification rules for Glitch WooCommerce Payment Gateway using Chrome DevTools MCP
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

Before browser verification, ensure the WordPress dev environment is running:

```bash
cd /path/to/glitch-woocommerce-gateway && wp-env start
```

The WordPress admin runs at `http://localhost:8888/wp-admin/`. The storefront runs at `http://localhost:8888/`. Navigate to the relevant page before taking screenshots or interacting.

Do not kill the server after you finished browsing, keep it alive so that the user can still work on it.

### Key Pages for Verification

- **Admin Settings:** `http://localhost:8888/wp-admin/admin.php?page=wc-settings&tab=checkout&section=glitch_gateway`
- **Checkout Page:** `http://localhost:8888/checkout/`
- **Order Admin:** `http://localhost:8888/wp-admin/edit.php?post_type=shop_order`

### Authentication Credentials

The WordPress dev environment requires login before accessing admin pages. Use these credentials:

- **Username:** `admin`
- **Password:** `password`

After navigating to any admin URL, if redirected to a login page, fill in the credentials above and submit before proceeding with verification.
