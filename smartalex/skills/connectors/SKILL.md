---
name: connectors
description: Connect other apps to SmartAlex and give its voice agents their tools. See which connectors and integrations are connected, get a sign-in link for Google, HubSpot and the rest, check a connection, add or remove the workspace's own MCP server, and switch a connector's tools on or off for an agent. Use when the user says "connect my Google calendar", "is HubSpot still connected", "which Google account is connected", "add our MCP server", "give the receptionist HubSpot's contact lookup", "turn off Notion's search for the sales agent", or "remove our old stock checker connector".
---

# Connectors

This mirrors **Agent Settings → Connectors** in the web app
(<https://getsmartalex.com/home?settings=connectors>). For every tool here with a `confirm_token`,
follow "The confirmation step" in the `smartalex` skill.

A connector is another service whose tools a voice agent can use during calls (Google Calendar and
Gmail, HubSpot, Notion, Stripe and others, or the workspace's own MCP server). The built-in
integrations (Outlook Calendar, email, WhatsApp, SMS, Telegram, social accounts, the CRMs, Shopify
and the rest) are listed alongside them.

## What is connected

`smartalex_list_connectors` lists each connector with its `connector_id`, whether it is connected,
how it connects (`sign_in`: none, a sign-in link, or the web app for one that needs an API key), its
tools and which agents have them on, and the built-in integrations with whether each is connected.
For one service, `smartalex_get_integration_status` says whether it is connected and as which
account. A sign-in that has lapsed shows `needs_sign_in_again`: offer a new sign-in link.

## Connect one

`smartalex_connect_integration` with the `connector_id` (`google` for Google Calendar and Gmail
sending, with `google_products` to pick one). It returns a one-time sign-in link that works for 10
minutes. Give the user the link to open themselves; you never open it or sign in for them. Nothing is
connected until they finish. Then check with `smartalex_get_integration_status`.

A connector that needs an API key gets no link: the reply gives the place in the web app where the
key is pasted. Never ask for the key in chat.

## Give an agent a connector's tools

`smartalex_set_agent_connector` with the `connector_id`, the agent, and the tools to switch on or
off (or `detach` to take the connector off the agent). Tools not named keep their setting. A tool
switched on lets the agent send that service what its calls need, from the next call, so before
calling it say which agent gets which tools and wait for a yes. The connector must be connected
first. The save is refused when the agent would have more than 20 tools; the reply names the
largest, so suggest one to switch off.

## The workspace's own MCP server

- **Add**: `smartalex_add_custom_connector` with a name, an `https` address, and `sign_in` (`none`,
  or `oauth` for a server that signs in, which then needs a sign-in link). SmartAlex reads its tool
  list straight away. Private-network addresses are refused. A server that needs an API key or a
  client secret is added in the web app. It has a preview: ask, then confirm. Its tools are on no
  agent until switched on as above.
- **Remove**: `smartalex_remove_custom_connector`. Its tools come off every agent and its saved
  sign-in is deleted. Curated connectors cannot be removed. Name it, ask, then confirm.

## Secrets

Never ask for, accept or repeat an API key, client secret or token. Keys are pasted in the web app
at the click path above.
