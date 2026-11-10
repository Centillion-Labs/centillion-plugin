# Centillion plugin

Build and test [Centillion](https://cntillion.com) AI agents from your AI
assistant. The plugin connects the assistant to Centillion's MCP server
(`https://app.cntillion.com/mcp`) and adds a skill that walks it through
finding or creating an agent, changing its draft, adding tools, writing and
running tests, and asking for a publish. Publishing, traffic changes, batch
launches and archiving wait for a person's approval in the dashboard.

## Install

```bash
claude plugin marketplace add Centillion-Labs/centillion-plugin
claude plugin install centillion@centillion
```

Then run `/mcp`, choose `centillion` and sign in to choose a workspace.

Without the plugin, add the server by its address:

```bash
claude mcp add --transport http centillion https://app.cntillion.com/mcp
```

Setup for other assistants, the full tool list and the limits are in the
[MCP guide](https://cntillion.com/docs/developer-guides/mcp).
