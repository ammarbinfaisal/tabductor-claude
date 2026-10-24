# Tabductor plugin for Claude Code

Cloud browsers and durable browser workflows from Claude. The plugin connects Claude Code to
Tabductor's MCP server and teaches Claude how to use it.

What Claude can do with it:

- **Browse**: start a cloud browser, open and switch tabs, navigate, run JavaScript in a page,
  take screenshots, reuse a profile's saved logins, and hand you a CDP URL for your own
  Playwright. (`browser_*` tools)
- **Automate**: publish a workflow from a prompt, run it, poll its JSON result, and schedule it.
  (`workflow_*` tools)

Tool reference, generated from the live server: https://www.tabductor.com/docs/mcp

## Install

1. Create an API key at https://www.tabductor.com/app/mcp and put it in your shell:

   ```bash
   export TABDUCTOR_API_KEY="td_mcp_..."
   ```

   The key must be in the environment Claude Code starts from (add the line to your shell
   profile). It is never written into the plugin or a config file.

2. Add the plugin:

   ```bash
   claude plugin marketplace add tabductor/tabductor-plugin
   claude plugin install tabductor@tabductor
   ```

   From a local checkout instead:

   ```bash
   claude plugin marketplace add /path/to/tabductor-plugin
   claude plugin install tabductor@tabductor
   ```

   Or, for one session without installing: `claude --plugin-dir /path/to/tabductor-plugin`.

3. Restart Claude Code (or run `/reload-plugins`). `/mcp` should list `tabductor` as connected.

To point the plugin at another deployment, set `TABDUCTOR_MCP_URL` to its `/api/mcp` endpoint.

## Use

Just ask: "open example.com in a browser and tell me what the pricing page says", "log in to
the portal with my saved profile and list my invoices", "make a workflow that checks this page
every morning and returns the price as JSON".

Two slash commands are included:

- `/tabductor:browse <url> [question]` opens a browser at the URL and reports what is there.
- `/tabductor:workflow <task>` publishes a workflow for the task, runs it once and reports the result.

Fewer permission prompts: allow the whole server in your settings.

```json
{ "permissions": { "allow": ["mcp__plugin_tabductor_tabductor"] } }
```

## What it costs

A browser bills by the minute while it is live. Claude stops browsers when it is done, and every
browser stops by itself after its idle timeout (10 minutes by default). Workflow runs bill their
own browser time and model usage. Balances and top-ups are at https://www.tabductor.com/app/billing.

## Layout

```
.claude-plugin/plugin.json       manifest
.claude-plugin/marketplace.json  lets this repository be added as a marketplace
.mcp.json                        the tabductor MCP server (HTTP, bearer key from the environment)
skills/tabductor/SKILL.md        how Claude uses the tools (loaded automatically)
skills/browse/SKILL.md           /tabductor:browse
skills/workflow/SKILL.md         /tabductor:workflow
```

Validate after editing: `claude plugin validate .`
