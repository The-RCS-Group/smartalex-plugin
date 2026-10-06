<p align="center"><img src="smartalex/assets/logo.png" alt="SmartAlex" width="96"></p>

**Run your SmartAlex AI phone team from your coding agent.** One plugin for Claude Code, Codex
and Cursor: a connection to your SmartAlex account plus skills that show the agent how to use it.

[Getting started](GETTING_STARTED.md) · [Docs](https://docs.getsmartalex.com/mcp/overview) · [SmartAlex](https://getsmartalex.com)

SmartAlex gives a business voice agents that answer the phone and call lists of people, with every
call recorded, transcribed and logged in a built-in CRM. This repository is the plugin marketplace
for `smartalex`: one plugin, three coding agents, one set of skills, and the SmartAlex MCP server.

## Example

```
> Add the SmartAlex voice widget to this site, then tell me who needs a call back from today.

⏺ smartalex_get_widget_install_code
⏺ Edit app/layout.tsx  (+ next/script, loads on every page)

⏺ smartalex_list_calls → 18 calls today, 14 answered
⏺ smartalex_get_call ×14

Widget added to app/layout.tsx.
Needs a call back:
- Thandi Mokoena: wants a quote for 3 units, after 2pm.
- James Carter: delivery complaint, asked for a manager.
```

## Set up

Paste this into your coding agent:

```
Set up the SmartAlex plugin by following the steps in https://github.com/The-RCS-Group/smartalex-plugin
```

**If you're an agent reading this:** read [`GETTING_STARTED.md`](GETTING_STARTED.md) and follow it.
If you cannot open that link, fetch the raw file instead:

```bash
curl -fsSL https://raw.githubusercontent.com/The-RCS-Group/smartalex-plugin/main/GETTING_STARTED.md
```

## What's inside

| Path | What it is |
|---|---|
| `.claude-plugin/`, `.cursor-plugin/`, `.agents/plugins/` | Marketplace files for Claude Code, Cursor and Codex |
| `smartalex/.mcp.json`, `smartalex/mcp.json` | The connection to `https://mcp.getsmartalex.com` (sign-in by OAuth, no keys in files) |
| `smartalex/skills/setup` | Connect, sign in and verify |
| `smartalex/skills/smartalex` | The map: every tool, which skill does what, and the safety rules |
| `smartalex/skills/add-voice-widget` | Put the voice and chat widget on the site in this codebase |
| `smartalex/skills/webhooks` | Receive signed call events in your own backend |
| `smartalex/skills/build-an-agent` | Create or change a voice agent |
| `smartalex/skills/call-a-list` | Phone a list as a campaign, only after you confirm |
| `smartalex/skills/todays-calls` | What happened on today's calls, and who to follow up |

No server code lives here. The plugin only points your agent at SmartAlex and teaches it the
workflows.

## Support

Questions or problems: [support@getsmartalex.com](mailto:support@getsmartalex.com).
Security reports: [security@getsmartalex.com](mailto:security@getsmartalex.com).
