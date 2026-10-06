# Getting started with SmartAlex

> This file is written to be handed to your coding agent. Paste the repository link and ask it
> to set SmartAlex up. The agent installs the plugin; you only click "Allow" once in your browser
> to sign in.

SmartAlex is an AI phone team for your business: voice agents that answer and make calls, plus
the contacts, campaigns, call transcripts and deals behind them. This plugin connects your coding
agent to your SmartAlex account and teaches it how to use it.

**What gets installed**

- A connection to `https://mcp.getsmartalex.com`, the SmartAlex MCP server. You sign in with
  your SmartAlex account in the browser. No API key is stored in any file.
- A set of skills that tell the agent how to do real jobs safely: add the voice widget to a site,
  wire call events into a backend, build a voice agent, call a list, and review today's calls.

**Before you start:** you need a SmartAlex account. Sign up at
<https://getsmartalex.com/signup> if you do not have one yet, then come back.

## Agent: install for the environment you are running in

Work out which agent you are (Claude Code, Codex or Cursor), then follow only that section.

### Claude Code

Requires Claude Code with plugin support (`claude plugin --help` lists `marketplace`).

```bash
claude plugin marketplace add The-RCS-Group/smartalex-plugin
claude plugin install smartalex@smartalex-plugins
```

The same works as slash commands inside a session:

```
/plugin marketplace add The-RCS-Group/smartalex-plugin
/plugin install smartalex@smartalex-plugins
```

Then go to [Run the `setup` skill](#run-the-setup-skill).

### Codex

```bash
codex plugin marketplace add The-RCS-Group/smartalex-plugin
```

Then ask the user to open **Plugins**, find **SmartAlex** and install it. Codex asks them to sign
in during install. Then go to [Run the `setup` skill](#run-the-setup-skill).

### Cursor

Cursor installs plugins from its marketplace or from a local folder. Team and Enterprise
workspaces can block local folders, so clone this repository and let the `setup` skill pick the
path that works:

```bash
git clone --depth 1 https://github.com/The-RCS-Group/smartalex-plugin.git /tmp/smartalex-plugin
```

Read `/tmp/smartalex-plugin/smartalex/skills/setup/SKILL.md` and follow it like a runbook. When it
finishes, delete the clone (`rm -rf /tmp/smartalex-plugin`).

## Run the `setup` skill

Run the bundled **`setup`** skill now, in this session, before anything else. It confirms the
SmartAlex connection is registered, gets the user signed in, and proves it works with one
read-only call.

- Invoke it by its full name, `smartalex:setup`, if your agent supports that.
- If the skill does not show up yet (some agents only register a new plugin after a restart),
  find the file and follow it directly:

  ```bash
  find ~/.claude ~/.codex ~/.cursor /tmp/smartalex-plugin -type f \
    -path '*smartalex/skills/setup/SKILL.md' 2>/dev/null | sort | tail -n1
  ```

## What's next

Run the bundled **`smartalex`** skill. It is the map: what SmartAlex can do, which skill handles
which job, and the safety rules for anything that calls real people or spends money.

## Support

Questions or problems: <support@getsmartalex.com>. Docs: <https://docs.getsmartalex.com/mcp/overview>.
