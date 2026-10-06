---
name: call-a-list
description: Have a SmartAlex voice agent phone a list of people as a campaign. Use when the user says "call these leads", "phone everyone on this list", "run a follow-up campaign", "ring my contacts about X", or hands over a CSV of people to call. Always confirms before any call is placed.
---

# Call a list with SmartAlex

This phones real people. Nothing is dialled until the user says yes to the exact summary in
step 5.

Campaign tools need the **Outbound** add-on. If `smartalex_create_campaign` is not available, tell
the user that, and stop.

## 1. Who to call

- From existing contacts: `smartalex_list_contacts`, paging through as needed, and filter by what
  the user asked for (tag, company, status).
- From a file or a list they give you: for each person, find them in the contacts first. Create
  only the ones that do not exist yet with `smartalex_create_contact` (phone numbers in E.164, for
  example `+27821234567`). Never create the same person twice.

Drop anyone with no phone number and tell the user how many you dropped.

Only call people the user has a right to call. If the list looks bought or scraped, or the user
mentions cold calling consumers, remind them that calling rules (consent, do-not-call lists,
calling hours) are their responsibility and ask them to confirm they have consent.

## 2. Which agent

`smartalex_list_agents`. If there is more than one outbound-capable agent, ask which one. If the
agent's objective does not fit this campaign, set the campaign's own `objective` in step 4 rather
than changing the agent.

## 3. Which number

`smartalex_list_phone_numbers` and ask which number to call from. Prefer a number in the same
country as the people being called. If there are none, say so; buying one is a monthly charge and
needs its own yes (`smartalex_search_available_numbers`, then `smartalex_buy_phone_number`).

## 4. Create the campaign and add people

1. `smartalex_create_campaign` with `name`, `agent_id`, `phone_number_id`, `objective` (what the
   agent should achieve on each call), `timezone` (the people's time zone, IANA format), and
   `business_hours` for the days and times calls are allowed.
2. `smartalex_add_contacts_to_campaign` with the contact ids, at most 100 per call. Repeat for
   larger lists. Duplicates are skipped automatically.

Creating and filling a campaign places no calls.

## 5. Confirm, then start

Show this and wait for a clear yes:

```
Campaign:   <name>
Agent:      <agent name>
Calling from: <number>
People:     <count>
Objective:  <objective>
Hours:      <days and times, time zone>
```

Only after a yes, call `smartalex_start_campaign`. If they want changes, make them and show the
summary again.

## 6. Afterwards

Tell the user the campaign has started and that they can ask for progress at any time. For
progress use `smartalex_get_campaign`; for results, the `todays-calls` skill.
