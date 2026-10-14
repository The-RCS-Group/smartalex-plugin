---
name: call-a-list
description: Have a SmartAlex voice agent phone a list of people as a campaign, and change, pause, resume or stop a campaign. Use when the user says "call these leads", "call my last 10 prospects and try each one 4 times", "phone everyone on this list", "run a follow-up campaign", "ring my contacts about X", "pause the campaign", or hands over a CSV of people to call. Always confirms before any call is placed.
---

# Call a list with SmartAlex

This phones real people. Nothing is dialled until the user says yes to a preview. Follow "The
confirmation step" in the `smartalex` skill for every tool here that has a `confirm_token`. Show the preview and ask even when the request already says to do it ("delete it", "just send it"): only a yes given after the preview counts.

Campaign tools need the **Outbound Calling** add-on. If `smartalex_create_campaign` is not
available, check `smartalex_get_plan_and_usage` (if you have it), tell the user what unlocks it,
and stop.

## Example: "call my last 10 prospects and try each one 4 times"

1. **Find them.** `smartalex_list_contacts` lists the newest first. Filter to prospects with
   `status` or `tag`, whichever their contacts use (look at a few first), and `per_page: 10`. If
   the user said how to tell (for example "my 10 newest contacts"), do that. If nothing marks a
   contact as a prospect and the user did not say, ask how to tell; that cannot be guessed.
2. **Agent and number.** Steps 2 and 3 below.
3. **Create the campaign** with `max_attempts: 4` and the defaults below for anything the user did
   not say (step 4). Do not stop to ask about them. Read it back and check the tries are 4.
4. **Add the 10.** `smartalex_add_contacts_to_campaign`. The preview says nobody is called until the
   campaign starts. Show it with the summary in step 6, including the defaults you chose, ask, and
   send the token only after a yes. The user can change any default in their answer.
   **In the same message, ask what the calls are for** (the purpose, step 4): "prospects" does not
   say whether it is selling, following up or booking appointments, and a campaign without a
   purpose cannot start.
5. **Set the purpose** from their answer with `smartalex_update_campaign`.
6. **Start.** `smartalex_start_campaign`. The preview says 10 people will be phoned. Show it with
   the summary in step 6, ask, and send the token only after a yes.
7. **Tell them it has started**, and how to check progress.

If the user also asks for a text after the last try, say that SmartAlex cannot do that (see "Text
messages" below) and carry on with the calls.

## 1. Who to call

- **Existing contacts:** `smartalex_list_contacts`. Use its filters rather than paging through
  everything: `search` (name, phone or email), `tag`, `status`, `created_after`, `never_called`,
  `last_called_before`. It returns a total count, so you can say how many match before going on.
- **A file or a list they give you:** `smartalex_import_contacts` takes up to 1000 rows per call.
  It matches existing contacts the same way creating one contact does, so nobody is added twice, and
  returns how many were created, updated and skipped, with a reason for each skip. Tell the user
  those numbers and the skip reasons. If the tool is not available, look each person up first and
  create only the missing ones with `smartalex_create_contact`. Never create the same person twice.
- Phone numbers go in E.164, for example `+27821234567`.

Drop anyone with no phone number and tell the user how many you dropped.

Only call people the user has a right to call. If the list looks bought or scraped, or the user
mentions cold calling consumers, remind them that calling rules (consent, do-not-call lists,
calling hours) are their responsibility and ask them to confirm they have consent.

## 2. Which agent

`smartalex_list_agents`. If only one agent can make calls (`call_direction` outbound or both), use
it without asking. If there is more than one and the user did not name one, ask which; that cannot
be guessed. If the agent's objective does not fit this campaign, set the campaign's own `objective`
in step 4 rather than changing the agent.

## 3. Which number

`smartalex_list_phone_numbers`. Pick the number yourself and say which: the only one; otherwise
one in the same country as the people being called; otherwise one on the chosen agent. Ask only if
that still leaves a real choice. If there are none, say so and stop; never buy one unasked. Buying
is a monthly charge with its own confirmation (the `phone-numbers` skill).

## 4. Create the campaign

`smartalex_create_campaign` with:

- `name`, `agent_id`, `phone_number_id`
- `purpose`: what the campaign is for (below)
- `objective`: what the agent should achieve on each call
- `timezone`: the people's time zone, IANA format
- `business_hours`: the days and times calls are allowed
- `max_attempts` and `retry_delay_minutes`, when the user wants more than one try

**Purpose.** What the campaign is for. If the user already said it in so many words ("run a
follow-up campaign", "remind everyone about tomorrow's appointment"), set it here. Otherwise never
guess: create the campaign without it, ask in the step 6 summary, and set it with
`smartalex_update_campaign` before starting. A campaign without a purpose cannot start. Use one
of `sales`, `follow_up`, `appointments`, `reminders`, `collections` (money owed), `renewals`,
`win_back` (past customers), `surveys`, or `other`.

**How tries work.** `max_attempts` is the total number of calls each person can get, the first one
included, so "try 4 times" is `max_attempts: 4` (10 at most). Only calls nobody answered (no
answer, busy, voicemail) are tried again; a person who answers is not called again.
`retry_delay_minutes` is the wait between tries, 10 minutes at least.

**Defaults.** For anything the user did not say, use these and list them in the summary (step 6).
Do not ask about them first; the user can change them when they answer the preview.

| Setting | Default |
|---|---|
| `name` | What the calls are for and the date, for example "Prospect follow-up, 10 Oct" |
| `objective` | What the user asked for, in one sentence; otherwise the agent's own objective |
| `timezone` | The people's time zone, from their numbers' country; the workspace's if they are mixed |
| `business_hours` | Monday to Friday, 09:00 to 17:00 in that time zone (the web app's default) |
| `retry_delay_minutes` | 60 minutes (the web app's default), when there is more than one try |

Read the campaign back with `smartalex_get_campaign`. If `max_attempts` is not what the user asked
for, the setting was not taken. Say so plainly and point them to **Pipeline → Campaigns**, open the
campaign, and edit it there. Never tell the user retries are set when they are not.

Creating a campaign places no calls.

## 5. Add the people

`smartalex_add_contacts_to_campaign` with the contact ids, at most 100 per call. Repeat for larger
lists. People already in the campaign are skipped.

On a campaign that has not started, the preview says nobody is called yet. On a running campaign
the people added start getting calls, so treat it like starting a campaign.

## 6. Confirm, then start or schedule

Show the preview together with this summary, then ask:

```
Campaign:      <name>
Purpose:       <purpose, or the question: what are these calls for?>
Agent:         <agent name>
Calling from:  <number>
People:        <count from the preview>
Tries:         <max_attempts> per person, <retry_delay_minutes> minutes apart
Objective:     <objective>
Hours:         <days and times, time zone>
Cost:          <as the preview says>
```

- To call now: `smartalex_start_campaign`.
- To start later: `smartalex_schedule_campaign` with the start time. Its preview gives the time in
  the workspace's time zone; check it is the time the user meant before asking.

Only after a clear yes, call it again with the `confirm_token`. If they want changes, make them and
start the preview again.

## 7. Afterwards

Tell the user the campaign has started (or when it will) and that they can ask for progress at any
time. For progress use `smartalex_get_campaign` (`pending_contacts` is how many are still waiting);
for results, the `todays-calls` skill.

## Changing a campaign

Read the campaign first with `smartalex_get_campaign`, and change only what the user asked for.
If a start is refused because the campaign has no purpose, ask what it is for and set `purpose`
with `smartalex_update_campaign`, then start it again.

| The user wants to... | Tool |
|---|---|
| Change the purpose, name, objective, agent, number, tries, wait between tries, calling hours, how many calls run at once, or the voicemail and text settings | `smartalex_update_campaign`. Some settings cannot change while the campaign is running; the tool says which. |
| Stop calling for now | `smartalex_pause_campaign`. No new calls are placed until it is resumed. |
| Carry on calling | `smartalex_resume_campaign`. The preview says how many people are still waiting. |
| Start at a set time | `smartalex_schedule_campaign` |
| Take people out of it | `smartalex_remove_contacts_from_campaign` |
| End it for good | `smartalex_stop_campaign`. It cannot be restarted. Offer pause instead unless they clearly want it over. |

If one of these tools is not available, they can do it in the web app: **Pipeline → Campaigns**,
then open the campaign.

## Text messages

SmartAlex cannot send a text after a campaign's last call attempt. Never offer it, promise it, or
build a workaround for it (no extra campaign, no other tool). If the user asks for it, say plainly
that it is not available today.

A campaign's own text options are different: one text per person after their first missed call,
and a text after an answered call. Neither waits for the last attempt. If the user asks about them,
describe them only that way.
