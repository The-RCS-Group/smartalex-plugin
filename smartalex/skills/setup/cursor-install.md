# Cursor: getting the SmartAlex plugin loaded

Cursor can load a plugin three ways. Try them in this order and stop at the first that works.
Team and Enterprise workspaces can switch the first two off, and the plugin then never appears
however often Cursor restarts. Do not try to get around a blocked policy; tell the user an admin
has to allow it, and use path C in the meantime.

## A. Local folder (an agent can do this)

Resolve the plugin root as two levels above this file's directory (`.../smartalex/skills/setup`),
then copy it into Cursor's local plugin folder:

```bash
plugin_root="$(cd "<THIS_SKILL_DIR>/../.." && pwd -P)"
[ -f "$plugin_root/.cursor-plugin/plugin.json" ] || { echo "wrong plugin root" >&2; exit 1; }
target="$HOME/.cursor/plugins/local/smartalex"
if [ "$plugin_root" != "$(cd "$target" 2>/dev/null && pwd -P)" ]; then
  mkdir -p "$(dirname "$target")"
  rm -rf "$target"
  cp -R "$plugin_root" "$target"
fi
```

Ask the user to **fully quit Cursor (Cmd/Ctrl+Q) and reopen it**. Reloading the window is often
not enough. Then check **Settings → Plugins** for SmartAlex.

If it is not listed after a full restart, local plugins are blocked for this workspace. Remove the
dead copy (`rm -rf "$HOME/.cursor/plugins/local/smartalex"`) and use B.

## B. Import from GitHub (the user does this in Cursor)

Ask the user to open **Settings → Plugins → Add Marketplace → Import from GitHub**, enter
`The-RCS-Group/smartalex-plugin`, and install **SmartAlex**. Admins can choose team scope so
everyone in the workspace gets it. Then fully restart Cursor.

## C. Connect the server only (always works)

If both A and B are blocked, the user still gets every SmartAlex tool by adding the server to
`~/.cursor/mcp.json`. They lose the skills, which only guide the agent, not the tools.

```bash
mcp_json="$HOME/.cursor/mcp.json"
[ -f "$mcp_json" ] || echo '{"mcpServers":{}}' > "$mcp_json"
tmp="$(mktemp)"
jq '.mcpServers.smartalex = {"url": "https://mcp.getsmartalex.com"}' "$mcp_json" > "$tmp" && mv "$tmp" "$mcp_json"
```

If `jq` is missing, edit the file by hand and add the same `smartalex` entry under `mcpServers`.
Never remove or change the user's other servers.

If the plugin later installs through A or B, delete this manual entry so the server is not listed
twice:

```bash
tmp="$(mktemp)"; jq 'del(.mcpServers.smartalex)' "$HOME/.cursor/mcp.json" > "$tmp" && mv "$tmp" "$HOME/.cursor/mcp.json"
```

Return to `SKILL.md`, step 3, to sign in.
