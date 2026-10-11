---
name: todays-calls
description: Summarise what happened on SmartAlex calls (today, another period, or one campaign), list the follow-ups, give the workspace's numbers for a period, and export the call records. Read-only. Use when the user asks "what happened on today's calls", "any calls I need to follow up", "how did the campaign go", "review the 700 calls in last week's campaign", "who called us", "how did my agents do this month", "how are we doing", "export last week's calls", or wants a call report or a spreadsheet of calls.
---

# Review calls

This skill only reads. It changes nothing unless the user asks for a follow-up action afterwards.

## A whole campaign, or more than a day of calls

Example: "review the 700 calls in last week's campaign". Count on the server first. Never page
through hundreds of calls one by one.

1. **Find the campaign.** `smartalex_list_campaigns`. If more than one ran in that period, ask which.
2. **Get the totals.** `smartalex_get_campaign_call_summary` with the `campaign_id`, or with `since`
   and `until` when there is no single campaign. It returns totals by outcome, the answer rate, the
   average duration, voicemails, callbacks requested, the cost in the plan's currency, and a short
   list of calls worth reading, each with the reason.
3. **Read only the calls on that list.** `smartalex_get_call` for each gives the transcript and
   summary.
4. **Offer the export** (see "Export the calls" below).
5. **Report**, numbers first:

```
"Spring follow-up", last week: 700 calls, 412 answered (59%), 180 voicemails, 108 not reached.
Average call 1m 42s. Cost: <exactly as the summary gives it>.
14 people asked for a callback.

Worth reading
- Thandi Mokoena: wants a quote for 3 units, asked for a call after 2pm.
- James Carter: unhappy about a late delivery, asked for a manager.

Full list: <export link> (expires <time>)
```

Give the cost exactly as the summary gives it, in the plan's currency. Never convert it or work it
out yourself. If the summary has no cost, the user's role cannot see charges: leave the cost line
out.

If `smartalex_get_campaign_call_summary` is not available, do not try to add up hundreds of calls
yourself. Use `smartalex_list_calls` with its filters (`campaign_id`, `since`, `until`, outcome) to
read a sample, say clearly that it is a sample, and point the user to **Pipeline → Campaigns**, then
open the campaign, for the full numbers.

## Today, or a short period

1. **Collect the calls.** `smartalex_list_calls` with `since` set to the start of the period (today
   means since midnight in the workspace's time zone, or the user's own if you do not know it; say
   which you used rather than asking) and `per_page: 100`.
   It returns the newest first. If the filters are not accepted, page through until the calls are
   older than the period.
2. **Read the ones that matter.** `smartalex_get_call` for each answered call. Skip unanswered calls
   (no answer, busy, failed) beyond counting them. For more than about 30 answered calls, use
   `smartalex_get_campaign_call_summary` for the period if you have it; otherwise read the longest
   calls and the ones with the lowest sentiment first, and say that you sampled.
3. **Report.** Lead with the numbers, then the people who need something:

```
Today: 42 calls, 31 answered, 6 voicemails, 5 not reached.

Needs follow-up
- Thandi Mokoena: wants a quote for 3 units, asked for a call after 2pm.
- James Carter: unhappy about a late delivery, asked for a manager.

Good news
- Priya Naidoo: booked a viewing for Thursday.
```

For each person give one line: what they want and by when. Include the contact name so the user
can find them. Do not paste transcripts unless asked.

## How the workspace did over a period

Example: "how did my agents do this month?"

1. `smartalex_get_analytics` with the period. "This month", "last month", "the last 7 days" and
   the like map straight onto the periods the web app's **Analytics** page offers (`7d`, `30d`,
   `90d`, `all`, `this_month`, `last_month`, `last_60_days`, `last_6_months`, or `custom` with
   dates). Pick the one the user named; with no period named, use `30d`, as the page does, and say
   so.
2. Report the figures with the page's own labels and nothing added:

```
This month: 1,284 conversations, 96 leads captured, 31 bookings. Spent: R 9,410.20.
```

Give money exactly as the tool gives it, in the account's currency. Never convert it or work it out
yourself, and leave it out if the tool gives none.

"My agents" means the whole workspace: the page, and the tool, cannot split the figures by agent
or campaign. If the user wants that, say so and offer the per-campaign figures instead
(`smartalex_get_campaign_call_summary` for each campaign, or **Analytics → Campaigns** in the web
app).

The web app: **Analytics** in the sidebar (<https://getsmartalex.com/analytics>).

## Export the calls

Example: "export last week's calls".

`smartalex_export_calls` gives the same file the web app's call records download gives, as a link
that expires after an hour. Do not page through calls to build a file yourself.

- **The period.** One calendar month per file (`month`, as YYYY-MM), this month or any of the 12
  before it, as in the web app. For a week or a date range, export the month that holds it (both
  months if it spans two) and tell the user the file covers the whole month. Do not ask which
  format or columns: there is one spreadsheet, the same as the web app's.
- **What is in it:** a Summary sheet, every call with its date, time, numbers, call type, minutes
  and charge, in the account's currency, and a sheet of details. Recordings are never in it.
- **Who may export:** in the web app only the workspace owner can. If the tool refuses for the
  user's role, say so and suggest asking the owner.
- Give the link and say when it expires. If it has expired, get a fresh one.

The web app: **Settings → Billing → Records → Statement**, choose the month, then **Download → Call
records**.

## Recordings

Call recordings are not shared through these tools. Do not look for, guess or build a recording
link. If the user wants to listen, send them to **Calls → Call History** in the web app.

## Offer next steps, do not take them

You can offer to:

- log or move deals for positive calls (`smartalex_create_deal`, `smartalex_update_deal`; ask
  before changing a deal's stage),
- update contact details the caller gave (`smartalex_update_contact`),
- go through the callbacks people asked for (the `callbacks` skill); SmartAlex calls them itself
  when they fall due,
- phone a group of them again as a new campaign (the `call-a-list` skill),
- draft follow-up messages for the user to send.

Do each only after the user says which ones they want.
