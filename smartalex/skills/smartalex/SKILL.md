---
name: smartalex
description: The map of SmartAlex, the AI phone team platform. Use whenever the user mentions SmartAlex, their voice agent, AI receptionist, call campaigns, call transcripts, or wants an AI to answer or make phone calls for their business. Explains what each smartalex_* tool does, which skill handles which job, and the safety rules for anything that calls real people, spends money or deletes data.
---

# SmartAlex

SmartAlex gives a business an AI phone team: voice agents that answer the business line, call
lists of contacts, take messages and book callbacks. Every call is recorded, transcribed and logged
against a contact in the built-in CRM, with deals in a sales pipeline.

You reach it through the `smartalex` MCP server. If no `smartalex_*` tools are available, run the
`setup` skill first.

## Pick the right skill

| The user wants to... | Skill |
|---|---|
| Put the voice and chat widget on their website | `add-voice-widget` |
| Get call or deal events into their own app or backend | `webhooks` |
| Create a voice agent, or change what an existing one says or does | `build-an-agent` |
| Have the AI phone a list of people | `call-a-list` |
| Know what happened on today's calls and what to follow up | `todays-calls` |
| Connect, reconnect or fix sign-in | `setup` |

Anything else, use the tools below directly.

## The tools

| Area | Tools |
|---|---|
| Account | `smartalex_get_platform_status` |
| Contacts | `smartalex_list_contacts`, `smartalex_get_contact`, `smartalex_create_contact`, `smartalex_update_contact` |
| Agents | `smartalex_list_agents`, `smartalex_get_agent`, `smartalex_create_agent`, `smartalex_update_agent`, `smartalex_delete_agent` |
| Phone numbers | `smartalex_list_phone_numbers`, `smartalex_search_available_numbers`, `smartalex_buy_phone_number` |
| Campaigns | `smartalex_list_campaigns`, `smartalex_get_campaign`, `smartalex_create_campaign`, `smartalex_add_contacts_to_campaign`, `smartalex_start_campaign` |
| Calls | `smartalex_list_calls`, `smartalex_get_call` (includes the transcript) |
| Deals | `smartalex_list_deals`, `smartalex_create_deal`, `smartalex_update_deal`, `smartalex_delete_deal` |
| Webhooks | `smartalex_list_webhooks`, `smartalex_create_webhook`, `smartalex_delete_webhook` |
| Website | `smartalex_get_widget_install_code` |
| Knowledge | `smartalex_search_company_knowledge` (searches SmartAlex's own help docs: plans, pricing, setup and policies. It does not search the user's business knowledge base, so never present its answers as the user's own prices or policies) |

Some tools only appear on plans that include them: campaigns and buying numbers need the
**Outbound** add-on, creating webhooks needs **Power Tools**. A missing tool means the plan does
not include it. Say which add-on unlocks it; do not report it as broken.

## Safety rules (always)

These actions reach real people or cost real money. Before each one, show the user exactly what
will happen and wait for a clear yes in this conversation. A yes for one action does not cover the
next.

1. **`smartalex_start_campaign`** phones real people. Show the campaign name, agent, phone number,
   number of contacts, objective and calling hours, then ask.
2. **`smartalex_buy_phone_number`** charges the account every month. Show the number and that it
   is a monthly charge, then ask.
3. **`smartalex_delete_agent`** and **`smartalex_delete_deal`** cannot be undone. A deleted agent
   stops answering calls immediately. Name what will be deleted, then ask.
4. **`smartalex_create_webhook`** returns a signing secret **once**. Put it straight into the
   project's secret store or an untracked `.env` file. Never print it in a commit, a log or a file
   that git tracks.

Also:

- Contacts: create a person once with `smartalex_create_contact`, then use
  `smartalex_update_contact` with the returned `id` for anything new. Calling create twice makes a
  duplicate.
- Transcripts and contact details are personal data. Show what the user asked for; do not paste
  whole transcripts into files, commits or issue trackers unless they ask.
- Every list tool is paginated (`page`, `per_page` up to 100). Page through when the user wants
  "all" of something.

## Where to send the user

- SmartAlex app: <https://getsmartalex.com>
- Docs: <https://docs.getsmartalex.com/mcp/overview>
- Support: <support@getsmartalex.com>
