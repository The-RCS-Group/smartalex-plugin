---
name: build-an-agent
description: Create a SmartAlex voice agent for a business, or change what an existing one says and does. Use when the user says "make me an AI receptionist", "create a voice agent for this business", "change the agent's greeting", "make the agent ask for email", or "the agent should know about our new pricing".
---

# Build or change a SmartAlex voice agent

## Creating a new agent

`smartalex_create_agent` reads the business's website, writes the agent's instructions and puts the
agent live. It takes 30 to 60 seconds, so gather what you need first and call it once.

Gather, asking only for what you cannot find yourself:

1. **Website** (`website_url`). The strongest input. If this codebase is the business's site, find
   its production URL in the config, `package.json` homepage, or README.
2. **What the agent should do on calls** (`objective`), in one or two plain sentences, for example
   "Answer questions about our services and book a callback with the team."
3. **Optional:** the name the agent uses on calls (`assistant_name`), the opening line
   (`first_message`), a voice (`voice`, for example Kore, Aoede, Puck, Charon) and an accent
   (`accent`: neutral, british, american, australian, south-african, indian).

Show the user what you are about to create in a short list, then call `smartalex_create_agent`.
Report the new agent's name and id.

If the tool is missing, every agent slot on the plan is in use. Say so; the user can free a slot or
add one in billing.

## Changing an existing agent

1. `smartalex_list_agents` to find it, `smartalex_get_agent` to read its current settings.
2. Call `smartalex_update_agent` with only the fields that change. Common ones:

| The user wants... | Field |
|---|---|
| A different greeting | `first_message` |
| It to always or never do something | `custom_instructions` (a list of short rules; send the full list, including the rules to keep) |
| A new goal | `objective` |
| Up-to-date product or pricing facts | `product_description` |
| It to answer or make calls | `call_direction`: `inbound`, `outbound` or `both` |
| A phone number on it | `phone_number_id` from `smartalex_list_phone_numbers` |
| To switch it off | `is_active: false` |

Changes go live on the next call. Read the agent back with `smartalex_get_agent` and confirm the
change in one line.

`custom_instructions` and `knowledge_item_ids` replace the whole list. Always read the current
value first and send it back with your change, or existing rules and knowledge links are lost.

## Deleting

`smartalex_delete_agent` cannot be undone and the agent stops answering calls immediately. Name the
agent and any phone number on it, and wait for a clear yes. Suggest `is_active: false` instead if
they might want it back.
