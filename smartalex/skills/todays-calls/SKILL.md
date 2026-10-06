---
name: todays-calls
description: Summarise what happened on the SmartAlex calls today (or another period) and list the follow-ups. Read-only. Use when the user asks "what happened on today's calls", "any calls I need to follow up", "how did the campaign go", "who called us", or wants a daily call report.
---

# Today's calls

This skill only reads. It changes nothing unless the user asks for a follow-up action afterwards.

## 1. Collect the calls

`smartalex_list_calls` returns the newest first. Page through (`per_page: 100`) until the calls
are older than the period the user asked about. "Today" means since midnight in the user's time
zone; ask if you cannot tell it.

## 2. Read the ones that matter

Call `smartalex_get_call` for each answered call to get the transcript. Skip unanswered calls
(no answer, busy, failed) beyond counting them. For more than about 30 answered calls, read the
ones with the longest duration and lowest sentiment first, and say that you sampled.

## 3. Report

Lead with the numbers, then the people who need something:

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

## 4. Offer next steps, do not take them

You can offer to:

- log or move deals for positive calls (`smartalex_create_deal`, `smartalex_update_deal`; ask
  before changing a deal's stage),
- update contact details the caller gave (`smartalex_update_contact`),
- draft follow-up messages for the user to send.

Do each only after the user says which ones they want.
