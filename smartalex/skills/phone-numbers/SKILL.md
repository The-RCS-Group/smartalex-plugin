---
name: phone-numbers
description: Manage the business's SmartAlex phone numbers. See which agent answers each number, put a number on an agent or move it to another one, buy a number, or release one for good. Use when the user says "move my number to the new agent", "put my new number on the Sales agent", "which agent answers my number", "take the number off this agent", "buy a London number", "release this number", "we don't need this number any more", or asks why they cannot get a number in their country.
---

# Phone numbers

This mirrors the web app's **Calls → Numbers** page. For every tool here with a `confirm_token`,
follow "The confirmation step" in the `smartalex` skill. Show the preview and ask even when the request already says to do it ("delete it", "just send it"): only a yes given after the preview counts.

## How numbers and agents fit together

- Each number is on one agent at most. That agent answers calls to the number and can call out from
  it.
- Putting a number on an agent takes it off the agent it was on. Nothing else about the old agent
  changes, but if that was its only number it can no longer answer or make calls until it gets
  another.
- One agent can have several numbers. When it calls out it takes turns between the ones in
  rotation, unless a campaign is set to call from one number. An agent's last number in rotation
  cannot be taken out of it.

`smartalex_list_phone_numbers` lists the numbers and which agent each one is on.

## Example: "move my number to the new agent"

1. `smartalex_list_phone_numbers` and `smartalex_list_agents`.
2. Decide, do not ask, when there is one sensible answer:
   - "my number": their only number. If they have several and one is on the old agent, that one.
   - "the new agent": the agent created most recently.
   Ask only when it really could be either, for example two numbers on two different agents.
3. `smartalex_assign_phone_number` with the number and the agent, exactly as the page's **Assign
   agent** does. The user asked for this, so do not ask again.
4. The tool returns the number with the agent it is now on. Check it is the agent you asked for,
   then say what changed:

```
+27 21 555 0199 now rings Alex. Sarah no longer has a number, so she cannot take or make calls
until she gets one.
```

To take a number off every agent, assign it to no agent (`agent_id: null`), like the page's **No
agent** choice. To stop an agent calling out from a number but keep it answering, set
`in_rotation: false`, like **Remove from rotation**.

## Buy a number

`smartalex_search_available_numbers` with the country, then `smartalex_buy_phone_number`. Buying is
a monthly charge and has its own confirmation: give the price exactly as the preview says, in the
plan's currency. Pick the country from what you know (the business's other numbers, or the people
it calls) and say which you chose; ask only if there is no clue. Put the new number on an agent
afterwards with `smartalex_assign_phone_number`.

Buying numbers needs the **Outbound Calling** add-on. The web app: **Calls → Numbers → Buy
number**.

## Release a number

`smartalex_release_phone_number` gives the number up for good. It cannot be undone, the number may
go to another business, and its monthly charge stops. Any agent on it stops answering it straight
away. The preview says all this; show it, name the number and the agent on it, and wait for a clear
yes.

Never release a number to move it. To move it, assign it to the other agent.

## Verification

Some countries need proof of who will answer the calls before a number can be issued. That means
uploading documents, which only works in the web app: **Calls → Verification**
(<https://getsmartalex.com/phone/verification>). Reviews take 1 to 3 business days.

## If these tools are not available

The user can do all of this in the web app: **Calls → Numbers**
(<https://getsmartalex.com/phone-numbers>), then the menu on the number's row: **Assign agent** or
**Release number**. Phone numbers are not on the free plan.
