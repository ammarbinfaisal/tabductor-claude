---
name: tabductor
description: Use Tabductor's cloud browsers and workflows. Use when the user wants to browse, scrape, test or act on a website from Claude (open a page, run JavaScript, screenshot, log in with a saved profile), or wants a browser task to run durably, on a schedule, or with a JSON result. Covers the tabductor MCP tools (browser_* and workflow_*).
user-invocable: false
---

# Tabductor

Tabductor runs real browsers in the cloud and turns natural-language intent into durable browser
workflows. This plugin exposes it through the `tabductor` MCP server. Tool names are prefixed
`mcp__plugin_tabductor_tabductor__`; the short names below are what follows that prefix.

Full reference, generated from the live server: https://www.tabductor.com/docs/mcp

## Which surface to use

- **A browser you drive step by step** (`browser_*`): look at a page, run JavaScript, extract data,
  check how a site behaves, use a saved login. You see every step and decide the next one.
- **A workflow** (`workflow_*`): a task Tabductor's own agent performs end to end from a prompt,
  can repeat on a schedule, and returns a JSON result. Use it when the user wants something done
  reliably or repeatedly rather than watched live.

## Cloud browsers

Every browser belongs to the user's account and bills by the minute while it is live. Start one
only when needed, do the work, and stop it when done.

1. **Start**: `browser_start` (optional `profile_id` to reuse saved logins, `name`, `idle_timeout_ms`,
   `proxy_country`). It waits up to `wait_ms` (default 90 s) and returns the browser. Act only when
   `ready` is true: `status` is `ready` and `inputOwner` is `api`. Otherwise poll `browser_get`.
   The result also carries `liveViewUrl`, a page where the user can watch the browser, and
   `profileId`, which is how to reuse its logins later.
2. **Tabs**: `browser_tabs` lists tabs with `pageId`, `url`, `title`, `selected`. A new browser has one
   blank tab. `browser_new_tab` opens another (optionally at a `url`) and selects it;
   `browser_select_tab` switches; `browser_close_tab` closes. A browser holds at most 16 tabs.
   Tools that take `page_id` act on the selected tab when it is omitted.
3. **Navigate**: `browser_navigate` loads a `url` and waits for the document (default 15 s), then
   returns the final `url` and `title`.
4. **Read and act**: `browser_evaluate` runs a JavaScript expression in the page and returns its JSON
   value. Statements go in an IIFE; a returned promise is awaited. A thrown error comes back as
   `exception`. Typical uses:
   - text: `document.body.innerText.slice(0, 20000)`
   - structured data: `Array.from(document.querySelectorAll("tr")).map(r => r.innerText)`
   - clicks and forms: `(() => { document.querySelector("#login").click(); return location.href })()`
   - wait for something: `new Promise(r => setTimeout(() => r(document.readyState), 1500))`
5. **See**: `browser_screenshot` returns the viewport as an image (jpeg by default) for layout
   questions or to confirm a state before acting.
6. **Stop**: `browser_stop` closes the browser and saves its profile (cookies, logins). It also stops
   by itself after the idle timeout (default 10 min). `browser_keepalive` resets that timer during
   long pauses; tab commands already count as activity.
7. **Your own Playwright**: `browser_cdp_url` mints a URL for `chromium.connectOverCDP` when the user
   wants to script the browser directly. The `browser_*` tab tools need no CDP client.

Rules of the road:
- Never type credentials, payment details or personal data the user did not give you.
- Stop before irreversible steps (sending messages, placing orders, deleting things) unless the
  user asked for exactly that step.
- Pages change under you: after a click or navigation, re-read what you need instead of trusting a
  value read earlier.
- `profile_in_use` means a workflow run or the user in the app has that profile open; wait or use
  another profile. `browser_input_pending` means the browser is handing input over; retry shortly.
- Large pages: slice `innerText` or select what you need rather than returning the whole DOM.

## Workflows

- `workflow_publish` with a directing `prompt` (what to do, on which site, what to return) and an
  optional `result_schema` (JSON Schema draft-07) creates a workflow. Reference inputs as
  `$variable-name` in the prompt.
- `workflow_trigger` starts it (pass `inputs` keyed by variable name, and reuse `request_id` when
  retrying an uncertain response). It returns an `executionId`.
- `workflow_status` polls that execution. When `finished` is true read `result`; failed or cancelled
  executions carry `errors`. Runs take seconds to minutes, so back off between polls.
- `workflow_update` replaces the prompt and schema and republishes; `workflow_schedule` sets a cron
  schedule with a timezone.

Workflows run in their own browsers on the account's profiles. A profile open in a `browser_*`
browser is unavailable to workflows until that browser stops, and vice versa.

## Setup problems

- "MCP bearer token required" or "Invalid or revoked MCP token": `TABDUCTOR_API_KEY` is not set in
  the environment Claude Code started from, or the key was revoked. Keys are created at
  https://www.tabductor.com/app/mcp. Tell the user to export it and restart; never ask them to paste
  the key into the chat.
- No `tabductor` tools at all: the plugin's MCP server did not connect; `/mcp` shows why.
