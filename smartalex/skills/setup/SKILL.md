---
name: setup
description: SmartAlex setup. Connects this agent to the user's SmartAlex account through the SmartAlex MCP server, gets them signed in, and proves it works. Use right after installing the plugin, when no smartalex_* tools are available, when a SmartAlex tool says the token is invalid or expired, when the Cursor plugin never appears, or when the user asks to connect or reconnect SmartAlex.
---

# SmartAlex setup

SmartAlex has no CLI. Everything runs through one remote MCP server:

```
https://mcp.getsmartalex.com
```

It signs the user in with OAuth in their browser. Never ask the user for a password, and never
write an API key into a file in their project.

## 1. Check whether the connection already works

If any `smartalex_*` tool is available to you, call `smartalex_get_platform_status`.

- It returns counts of contacts, campaigns, calls and deals: setup is done. Go to step 5.
- It fails with an auth error: go to step 3.
- No `smartalex_*` tools exist at all: go to step 2.

## 2. Make sure the server is registered

The plugin registers a server named `smartalex`. Check it is there for your agent.

**Claude Code**

```bash
claude mcp list
```

If `smartalex` is missing, the plugin is not installed or not enabled. Run:

```bash
claude plugin marketplace add The-RCS-Group/smartalex-plugin
claude plugin install smartalex@smartalex-plugins
```

**Codex**

```bash
codex mcp list
```

If `smartalex` is missing, run `codex plugin marketplace add The-RCS-Group/smartalex-plugin` and
ask the user to install **SmartAlex** from **Plugins**.

**Cursor**

Read `cursor-install.md` in this directory and follow it, then come back to step 3.

## 3. Sign in (you start it, the user finishes it in the browser)

Start the sign-in yourself, right now, in this session. Do not tell the user to go and find a
menu. The command opens the user's browser on a SmartAlex page; they sign in and press **Allow**,
and the command finishes on its own. Never try to fill in the page for them.

**Claude Code**

```bash
claude mcp login plugin:smartalex:smartalex
```

**Codex**

```bash
codex mcp login smartalex
```

(`codex mcp list` shows the exact server name if that one is not found.)

**Cursor**

If the `cursor-agent` command exists, run `cursor-agent mcp login smartalex`. Otherwise ask the
user to open **Settings → Tools & MCP**, find `smartalex`, and press **Connect**.

The command waits for the browser, up to about 5 minutes:

- If your shell tool lets you set a timeout, ask for at least 5 minutes and run it directly.
- If it does not, run it in the background so it survives your timeout, for example
  `nohup claude mcp login plugin:smartalex:smartalex > /tmp/smartalex-login.log 2>&1 &`, then
  poll `claude mcp list` (or `codex mcp list`) every few seconds until `smartalex` shows
  **Connected**.
- If no browser opened (a remote or headless machine), show the user the sign-in link from the
  command's output and ask them to open it.

Tell the user in one line what is happening, for example: "A SmartAlex sign-in page just opened
in your browser. Sign in and press Allow; I'll carry on as soon as you do."

When the command finishes, confirm with `claude mcp list` (or `codex mcp list`) that `smartalex`
shows **Connected**, then go to step 5. If the `smartalex_*` tools are still not available to
you in this session, they load in a new chat (Claude Code: `/mcp` → `smartalex` → **Reconnect**
also works; Cursor: fully quit and reopen). Say so in one line; the sign-in itself is done and
does not need repeating.

## 4. Fix the common failures

| What you see | What it means | What to do |
|---|---|---|
| "Invalid or expired token" straight after a successful sign-in | The account has no active workspace yet. This happens to brand-new sign-ups. | Ask the user to sign in once at <https://getsmartalex.com>, finish onboarding until they see their dashboard, then sign in again (step 3). |
| Sign-in page says the link expired | The sign-in took too long or was started twice. | Start step 3 again and use only the newest link. |
| A tool says the sign-in has expired (401), after it worked before | The connection was signed out. Signing in to the SmartAlex website with two-step login does this. | Start step 3 again. Nothing else needs fixing. |
| Campaign tools or `smartalex_buy_phone_number` are missing | They need the **Outbound Calling** add-on. API tools and creating or deleting webhooks need **Power Tools**. Tools the plan, add-ons or role do not allow are hidden, not broken. | Tell the user which add-on unlocks it. Add-ons are bought in **Settings → Billing** (<https://getsmartalex.com/account?tab=billing>). |
| `smartalex_create_agent` is missing | Every agent slot on the plan is in use. | The user can delete an unused agent, or add a slot (`smartalex_buy_agent_slot`, or **Settings → Billing**). |
| A tool is refused with "your role can't do this" (403) | Their role in this workspace does not allow it. | Suggest asking a workspace owner or admin. Signing in again will not change it. |
| Server unreachable | Network or proxy problem. | Check `curl -sI https://mcp.getsmartalex.com` returns a response. Do not restart sign-in for a network error. |

## 5. Verify and hand over

If the `smartalex_*` tools are available, call `smartalex_get_platform_status` and tell the user,
in one line, that SmartAlex is connected and how many contacts, campaigns and calls they have.
Keep account details to yourself unless they help fix a problem.

If they are not available in this session yet, `smartalex` showing **Connected** in
`claude mcp list` / `codex mcp list` is the proof. Tell the user setup is done and that the
SmartAlex tools are ready in a new chat.

Then run the `smartalex` skill, which explains what the user can do next.
