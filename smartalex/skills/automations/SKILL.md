---
name: automations
description: See the SmartAlex workspace's automations and switch them on or off, and set up Instagram and Facebook comment-to-DM replies. Use when the user asks "what automations do we have running", "is the after-call email on", "pause the new-lead journey", "turn off AI replies", "stop the keyword reply for PRICE", "when someone comments PRICE on Instagram, DM them the price list", "change the giveaway DM", "who got the PRICE DM", or "stop the giveaway DMs".
---

# Automations

This mirrors the **Automations** page in the web app (<https://getsmartalex.com/automations>). For
every tool here with a `confirm_token`, follow "The confirmation step" in the `smartalex` skill. Show
the preview and ask even when the request already says to do it: only a yes given after the preview
counts.

## What there is

`smartalex_list_automations` shows every automation with its kind, id, name, when it runs, who or
what it reaches, its channels, and whether it is on:

- **journey**: messages and may phone each new contact (Outbound Campaigns add-on)
- **email**: emails people after calls, or on a schedule (Outbound Campaigns add-on)
- **rule**: a keyword reply to DMs (inbox add-on)
- **ai**: the workspace's AI replies to DMs and comments, id `replies` (inbox add-on)
- **comment**: an Instagram or Facebook comment-to-DM automation. These have no on/off switch.
- message sequences and WhatsApp broadcasts, shown with a status; they are managed in the web app.

`comment_accounts` lists the Instagram and Facebook accounts a new comment-to-DM can run on.

Answer from that list. Quote wording only as the list gives it; the full text of a message lives in
the web app.

## Switch one on or off

- **Off**: `smartalex_disable_automation` with its `kind` and `id`. It stops straight away (switching
  an email automation off also cancels the emails it has queued); its content is kept. It has no
  preview, so before calling it tell the user what stops, and wait for their yes.
- **On**: `smartalex_enable_automation` with its `kind` and `id`. From then on it runs by itself and
  can message or phone people, so it has a preview: say who it reaches, ask, then confirm.
- Only journey, email, rule and ai have a switch. A comment-to-DM automation is stopped by deleting
  it (below). Creating or editing journeys, emails, keyword replies and AI replies is done in the
  web app.

## Instagram and Facebook comment-to-DM

When someone comments (or replies to a story) with one of the keywords, the account sends them a
DM by itself, and can also reply publicly under their comment.

- **Set one up**: `smartalex_create_comment_automation` with an `account_id` from
  `comment_accounts`, a name, the keywords, the DM, and optionally a public reply. `trigger` is
  `comment` (a comment on a post) or `story_reply` (a reply to a story). Keywords are
  not case sensitive, and an empty keyword list means every comment sets it off. If
  `comment_accounts` is empty, no Instagram or Facebook account is connected: say so, and send the
  user to connect one (the `connectors` skill, or **Automations** in the web app). Never invent an
  account id. It starts running at once, so it has a preview: show the keywords and the DM, ask,
  then confirm.
- **Change one**: `smartalex_update_comment_automation` with its `id` and only what changes. The new
  wording is sent from its next match. It has a preview.
- **See what it did**: `smartalex_get_comment_automation_log` with its `id`: the latest 50 comments
  that set it off and the DMs it sent.
- **Stop one**: `smartalex_delete_comment_automation`. It stops sending at once and cannot be undone;
  DMs already sent stay sent. Name it, ask, then confirm.

Keep the DM short and in the business's voice. Do not promise anything in it (a price, a
discount, a time) that the user did not give you.

## When it is refused

Each kind needs its add-on and the matching permission. A viewer can look but change nothing. Say
plainly which it is (see "When a tool is refused" in the `smartalex` skill). A journey that phones
people cannot be switched on until the Outbound Calling Policy is accepted in the web app.
