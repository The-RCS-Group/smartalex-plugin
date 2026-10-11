---
name: callbacks
description: See who asked to be called back, call them back now, reschedule or cancel a callback, and block or unblock numbers that should never reach the agents. Use when the user asks "who do I need to call back", "any callbacks today", "what's overdue", "call Thandi back now", "move that callback to Friday", "cancel the callback", "block this number", "stop this caller getting through", or "unblock a number".
---

# Callbacks and blocked callers

This mirrors two web app pages: **Calls → Callbacks** and **Calls → Blocked Callers**. For every
tool here with a `confirm_token`, follow "The confirmation step" in the `smartalex` skill.

## How callbacks work

When a caller asks to be called back, SmartAlex books a callback for that time. When it falls due,
SmartAlex calls the person itself, inside the campaign's calling hours, with the campaign's agent and
number. So a callback on the list is called automatically unless it is cancelled. "Call now" only
brings it forward.

A callback that could not be made is marked failed, with the reason (often that there is no number
to call from).

## Example: "who do I need to call back today?"

1. `smartalex_list_callbacks`. It returns the same list the page shows: each person's name, number,
   company, campaign, when the callback is due, and whether it is overdue, today, upcoming or
   failed, with the reason for a failure. Its `when` matches the page's tabs (all, overdue, today,
   week); leave it at all and sort the answer yourself, so nothing overdue is missed.
2. Report overdue first, then today, then anything failed. "Today" means overdue and due today; do
   not ask which period. With no period named, show overdue, today and this week.

```
3 callbacks need you today.

Overdue
- Thandi Mokoena, +27 82 555 0101, Spring follow-up. Was due 9:30 this morning.

Today
- James Carter, +44 20 7946 0102, Inbound. Due 2:00 pm.
- Priya Naidoo, +27 21 555 0103, Spring follow-up. Due 4:15 pm.

Could not be called
- Sam Lee: there is no number to call from. Add one in Calls → Numbers.
```

3. Offer, do not take, the next steps: call someone back now, move a callback, or cancel one. The
   ones still pending will be called automatically when due.

## Call someone back now

`smartalex_call_back_now`. It places a real call straight away, like the page's **Call** button.
The preview names the person and the number. Ask, and send the token only after a clear yes. It also
needs permission to run campaigns; if the role cannot, say so plainly.

## Reschedule or cancel

These do exactly what the page's menu offers, nothing more:

- **Reschedule**: `smartalex_update_callback` with the new `date` and `time` (24-hour), in the
  workspace's time zone, today or later. If the user gives a day but no time, keep the callback's
  current time of day and say so. Rescheduling does not revive a failed or cancelled callback; to
  phone that person, use call back now.
- **Cancel**: `smartalex_cancel_callback`. SmartAlex will not call that person. It only works on a
  callback still waiting, and cannot be undone; a callback that failed or finished cannot be
  cancelled.

There is no "mark as done": a callback is done when the call is made. If the user has called the
person themselves, cancel it so SmartAlex does not call them again.

Both are done when the user asks; do not ask again.

## Blocked callers

A blocked number cannot reach the agents: inbound calls from it get a busy tone. Blocking does not
stop SmartAlex calling that number. To stop calling a person, that is Do Not Call on the contact,
which is changed only in the web app (open the contact under **Contacts**).

- `smartalex_list_blocked_callers`: each number, the reason, how many calls it stopped and when it
  was blocked.
- `smartalex_block_caller`: the full number, with the country or area code, and an optional reason.
  If the user gave a reason, pass it; otherwise leave it out rather than asking. A number already on
  the list is refused; tell the user it is already blocked.
- `smartalex_unblock_caller`, with the id from the list: the number can reach the agents again
  from its next call. Say that in your reply.

Do each when the user asks for it; do not ask again.

## If these tools are not available

The user can do all of this in the web app: **Calls → Callbacks**
(<https://getsmartalex.com/callbacks>) and **Calls → Blocked Callers**
(<https://getsmartalex.com/phone/blocked>). Both pages are not on the free plan.
