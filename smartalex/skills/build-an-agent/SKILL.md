---
name: build-an-agent
description: Create a SmartAlex voice agent for a business, change what an existing one says, does or knows, and try it with a test call. Use when the user says "make me an AI receptionist", "set up an agent from my website", "create a voice agent for this business", "change the agent's greeting", "make the agent ask for email", "the agent should know about our new pricing", "undo that prompt change", or "give the agent a test call".
---

# Build or change a SmartAlex voice agent

For every tool here with a `confirm_token`, follow "The confirmation step" in the `smartalex` skill.

## Example: "set up an agent from my website"

1. **Gather and create** (see "Creating a new agent" below). Pass the website as `website_url`.
2. **Check what it knows.** Creating an agent from a website already saves that site to the
   agent's knowledge, and reads more of its pages in the background over the next few minutes.
   `smartalex_list_knowledge` shows what is there. Do not add the same website again.
3. **Add what is missing.** Another site or a page the list does not cover:
   `smartalex_add_knowledge_from_website`, then `smartalex_get_knowledge_import_status` every
   20 to 30 seconds until it is done. A price list or FAQ the user pastes:
   `smartalex_add_knowledge_text`. Link anything new to the agent with
   `smartalex_link_knowledge_to_agent` if the import did not link it already.
4. **Test call** (see "Try it with a test call" below), to the user's own phone.
5. **Report** in a few lines: the agent's name, what it knows, how the test call went, and anything
   worth changing.

## Creating a new agent

`smartalex_create_agent` reads the business's website, writes the agent's instructions and puts the
agent live. It takes 30 to 60 seconds, so gather what you need first and call it once.

Gather, asking only for what you cannot find yourself:

1. **Website** (`website_url`). The strongest input. If this codebase is the business's site, find
   its production URL in the config, `package.json` homepage, or README.
2. **Company name** (`company_name`) and **what the company does** (`company_description`, one or
   two sentences, 300 characters max). Both are required. Read from the website when there is one;
   if the tool answers that it could not find them, ask the user and call it again with both.
3. **What the agent should do on calls** (`objective`), in one or two plain sentences. If the user
   did not say, use "Answer questions about the business and book a callback with the team" and say
   so; they can change it afterwards.
4. **Optional:** the name the agent uses on calls (`assistant_name`), the opening line
   (`first_message`), a voice (`voice`, for example Kore, Aoede, Puck, Charon) and an accent
   (`accent`: neutral, british, american, australian, south-african, indian).

Do not ask about the optional settings; leave them to SmartAlex's defaults. Show the user what you
are about to create in a short list, then call `smartalex_create_agent`.
Report the new agent's name and id.

**No free agent slot.** If the tool is missing, or refuses because every agent slot is in use:

- `smartalex_get_plan_and_usage` shows slots used and free.
- `smartalex_buy_agent_slot` adds one. It costs money, so it has its own confirmation; give the
  price exactly as the preview says, in the plan's currency.
- Or the user can delete an agent they no longer use, or add a slot in **Settings → Billing**.

## Changing what an existing agent says or does

1. `smartalex_list_agents` to find it, `smartalex_get_agent` to read its current settings.
2. Make the change one of two ways.

**The whole prompt, section by section.** If you have `smartalex_get_agent_prompt`, use it first: it
shows every section of the agent's instructions as the web app shows them. Change one section with
`smartalex_update_agent_prompt_section`, sending the section exactly as `smartalex_get_agent_prompt`
names it. Tell the user what changed, in one or two lines. Every change is kept as a version:
`smartalex_get_prompt_history` lists them and `smartalex_revert_prompt_section` puts an earlier
one back, so "undo that" is always possible.

**The quick fields**, with `smartalex_update_agent`, sending only the fields that change:

| The user wants... | Field |
|---|---|
| A different greeting | `first_message` |
| It to always or never do something | `custom_instructions` (a list of short rules; send the full list, including the rules to keep) |
| A new goal | `objective` |
| Up-to-date product or pricing facts | `product_description` |
| It to answer or make calls | `call_direction`: `inbound`, `outbound` or `both` |
| A phone number on it | `smartalex_assign_phone_number` (the `phone-numbers` skill); without it, `phone_number_id` from `smartalex_list_phone_numbers` |
| To switch it off | `is_active: false` |

`custom_instructions` and `knowledge_item_ids` replace the whole list. Always read the current
value first and send it back with your change, or existing rules and knowledge links are lost.

Changes go live on the next call. Read the agent back and confirm the change in one line.

## What the agent knows

The agent answers from the business's knowledge base. Longer facts (price lists, FAQs, policies)
belong there, not in the prompt.

- `smartalex_list_knowledge`: what is in it, and which agents use each item.
- `smartalex_add_knowledge_text`: text the user pastes.
- `smartalex_add_knowledge_from_website`: a web page or site. It runs in the background; check
  `smartalex_get_knowledge_import_status` until it is done before saying the agent knows it.
- `smartalex_link_knowledge_to_agent` and `smartalex_unlink_knowledge_from_agent`: choose which
  agents use an item. Prefer these over `knowledge_item_ids` on `smartalex_update_agent`.
- `smartalex_delete_knowledge_item`: deletes it for good, for every agent that uses it. If the user
  only wants one agent to stop using it, unlink it instead.

Never put passwords, API keys or customers' personal details into knowledge.

If these tools are not available, the user can manage knowledge in the web app: **Agent Settings →
Knowledge**.

## Try it with a test call

1. The number to call: the one the user gave, otherwise ask; it cannot be guessed. Only call the
   user's own phone, or a number they say belongs to someone expecting the call. Use E.164, for
   example `+27821234567`.
2. `smartalex_start_test_call` with the agent and the number. The preview names the agent, the
   number to be called and the cost estimate. Ask, and send the token only after a yes.
3. Only one test call runs at a time. If one is already running, wait for it to finish.
4. `smartalex_get_test_call_result` every 20 to 30 seconds until the call has finished. It gives the
   status, transcript and summary.
5. Tell the user in a few lines how the call went and anything the agent got wrong, and offer to fix
   it. Do not paste the whole transcript unless they ask.

## Deleting

`smartalex_delete_agent` cannot be undone and the agent stops answering calls immediately. Name the
agent and any phone number on it, and wait for a clear yes. Suggest `is_active: false` instead if
they might want it back.
