# Centillion plugin

Build, test and run [Centillion](https://cntillion.com) AI agents for phone
calls and chats from your AI assistant. The plugin connects the assistant to
Centillion's MCP server (`https://app.cntillion.com/mcp`) and adds two skills:

- `setup-centillion` connects the assistant, signs you in or creates your
  account, and builds your first agent with you. The same guide is published
  at https://cntillion.com/SKILL.md.
- `build-centillion-agent` covers changing an agent in depth: drafts,
  workflows, tools, tests, branches and approvals.

Publishing, traffic changes, phone numbers, batch calls and campaigns wait for
a person's approval in the dashboard.

## Quick start from any coding agent

Paste this into Claude Code, Codex, Cursor or another assistant that can run
shell commands:

```text
I want an AI agent that answers my business's calls and chats. Fetch and follow https://cntillion.com/SKILL.md to set up Centillion || npx skills add Centillion-Labs/centillion-plugin
```

## Install

Claude Code:

```bash
claude plugin marketplace add Centillion-Labs/centillion-plugin
claude plugin install centillion@centillion
```

Third-party marketplaces don't auto-update by default: turn auto-update on
for `centillion` under **Marketplaces** in `/plugin`, or run
`/plugin marketplace update centillion` to get new commits.

Codex:

```bash
codex plugin marketplace add Centillion-Labs/centillion-plugin
codex plugin add centillion@centillion
codex mcp login centillion
```

Skills only, for any agent the [skills CLI](https://skills.sh) supports:

```bash
npx skills add Centillion-Labs/centillion-plugin
```

Then sign in: run `/mcp` in Claude Code and choose `centillion`, or
`codex mcp login centillion` in Codex. Sign in with your Centillion account,
or create one, and choose a workspace.

Without the plugin, add the server by its address:

```bash
claude mcp add --transport http centillion https://app.cntillion.com/mcp
codex mcp add centillion --url https://app.cntillion.com/mcp
```

Or use the CLI, which signs in through your browser and makes every MCP tool
a shell command:

```bash
npx -y centillion@latest onboard
```

Setup for Claude, ChatGPT, Cursor and VS Code is at
https://cntillion.com/mcp. The full tool list and limits are in the
[MCP guide](https://cntillion.com/docs/developer-guides/mcp).

## Layout

The plugin lives in `plugins/centillion/`: its skills, `.mcp.json`, and a
manifest for Claude Code (`.claude-plugin/plugin.json`) and Codex
(`.codex-plugin/plugin.json`). The repository root holds the marketplaces that
point at it (`.claude-plugin/marketplace.json`,
`.agents/plugins/marketplace.json`), the Cursor manifest, and the MCP Registry
entry (`server.json`).
