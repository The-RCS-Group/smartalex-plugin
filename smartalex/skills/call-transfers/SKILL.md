---
name: call-transfers
description: Set who a SmartAlex voice agent puts live callers through to, and when. See an agent's transfer list, add or change a person or line, remove one, turn transfers on or off, and change the "When to transfer" rule. Use when the user says "who does my agent transfer calls to", "put callers through to Sarah", "add reception as a transfer number", "only transfer people who want a manager", "stop transferring calls", "turn call transfer back on", or "take Mike off the transfer list".
---

# Call transfers

This mirrors **Home → agent menu → Agent settings → Tools → Transfer Call** in the web app. For
every tool here with a `confirm_token`, follow "The confirmation step" in the `smartalex` skill. Show
the preview and ask even when the request already says to do it: only a yes given after the preview
counts.

## How transfers work

Each agent has its own transfer list. Every entry (a destination) is a person or line with a name, a
number in international form, and aliases: other names callers use for them ("reception", "the
front desk"). One destination can be the default, for a caller who asks for no one by name. The
"When to transfer" rule is plain text the agent follows, such as "when a caller wants to speak to a
manager". While transfers are on, the agent can put a live caller through from its next call.

## Find the agent first

`smartalex_list_agents`. With one agent, use it and say so. With several, use the one the user named;
ask only when it cannot be told apart.

## See who the agent transfers to

`smartalex_get_agent_transfers` with the agent's id. Report whether transfers are on, the "When to
transfer" rule, and each destination with its number and aliases, marking the default. If the agent
also has an old single forwarding number, say so.

## Add a person or line, or change one

`smartalex_set_transfer_destination`. Without `destination_id` it adds one (name and phone needed);
with `destination_id` (from the list above) it changes only what you pass.

- The number must be in full international form with the country code. When the user gives a local
  number and the country is plain from the business, add the code and say what you used; otherwise
  ask for the country.
- Put the other names people use in `aliases`. Set `is_default` only when the user says this is who
  callers go to when they ask for no one by name, or when it is the agent's only destination.
- The preview says whether this turns transfers on (the agent's first destination does). Ask, and
  send the token only after a clear yes.
- A number can be refused (malformed, premium-rate, a UK personal or VoIP number, a blocked range).
  Pass the reason on in plain words and ask for another number. Never try it again unchanged.

## Turn transfers on or off, or change when to transfer

- **On**: `smartalex_turn_on_agent_transfers`, optionally with `when_to_transfer`. It checks every
  saved destination first and is refused when the agent has none: then add one first. It has a
  preview: ask, then confirm.
- **Off**, or a new rule: `smartalex_set_agent_transfers` with `enabled: false`, a new
  `when_to_transfer`, or both. It cannot turn transfers on. It has no preview, so before calling it
  tell the user what will change (from the next call the agent puts no one through; the list and the
  rule are kept) and wait for their yes.

## Remove a person or line

`smartalex_remove_transfer_destination` with the agent's id and the `destination_id`. Callers can no
longer be put through to it. Removing the last one turns transfers off for that agent. It has a
preview: name who is removed, ask, then confirm.

## What it cannot do

Transfers happen during the agent's calls; no tool here places a call. Anything else about
transfers is changed in the web app at the click path above.
