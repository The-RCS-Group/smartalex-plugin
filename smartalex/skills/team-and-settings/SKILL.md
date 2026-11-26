---
name: team-and-settings
description: Manage who is in the SmartAlex workspace and how it runs. Invite a colleague, change someone's role, remove someone, cancel an invitation, set the business hours, change your own notification settings, change the report emails, and pause or resume all email for the workspace. Use when the user says "who gets the campaign reports", "turn off the weekly recap", "send the daily report at 7", "add our sales manager to the campaign reports", "pause all email for our account", "why is nobody getting emails", "invite my colleague", "add Sarah to SmartAlex", "who's on my team", "make Tom an admin", "remove Jo from the workspace", "cancel that invite", "we're open 8 to 6 now", "change our business hours", "stop emailing me about every call", or "turn on WhatsApp alerts".
---

# Team and workspace settings

This mirrors the web app's **Settings → Workspace → Team**, **Settings → Workspace → General →
Business hours**, **Settings → Account → Notifications** and **Insights → Reports → Settings**. For every tool here with a `confirm_token`, follow "The
confirmation step" in the `smartalex` skill. Show the preview and ask even when the request already says to do it ("delete it", "just send it"): only a yes given after the preview counts.

## The team

Only workspace owners and admins manage the team. Roles, from most to least access:

| Role | Can |
|---|---|
| Owner | Everything. There is one owner, and nobody can be invited or changed to owner. |
| Admin | Manage the workspace and the team, except the owner and other admins. |
| Member | Use the workspace day to day. |
| Viewer | Look, not change. |

An owner can give admin, member or viewer. An admin can give member or viewer only.

`smartalex_list_team` lists the members and the invitations not yet accepted.

## Example: "invite my colleague Sarah as a member"

1. **The email address** is the one thing you cannot guess. If the user did not give it, ask for it,
   and only that.
2. **The role**: the one they named. If they named none, use **member** and say so in a few words;
   do not list the other roles or ask them to pick.
3. `smartalex_invite_team_member` with the email and role. It emails the person, so it has a
   confirmation step. The preview names the email address and the role; show it, ask, and send the
   token only after a clear yes. This holds when the user said "just send it" or "don't ask me to
   confirm": the request is not the yes, and an email cannot be unsent. Show the preview in a line or
   two, ask once, and stop. Never send the token in the same reply as the preview.
4. Tell the user the invitation is sent and the link in it works for 7 days.

**Trial limit.** On a trial, a workspace can invite up to 2 people (people already invited count,
the owner does not). When it is hit the tool says: "Your trial lets you invite up to 2 team
members." Pass that on: plans are managed in **Settings → Billing**
(<https://getsmartalex.com/account?tab=billing>). Do not retry, and do not suggest an upgrade.

**Already invited.** If the tool says the person was already invited, tell the user when, and offer
to send it again: the same call with `resend: true` emails the same link with 7 more days. That is
another email, so it previews again. Copying the link is only in the web app: **Settings → Team**.

## Change a role, remove someone, cancel an invitation

Members are found by the `user_id`, and invitations by the `invitation_id`, that
`smartalex_list_team` returns.

- `smartalex_change_member_role`: admin, member or viewer. Changing a role resets that person's
  custom page permissions to the new role's defaults; say so. Nobody can change their own role or
  the owner's, an admin cannot change another admin, and only an owner can make someone an admin.
- `smartalex_remove_team_member`: the person loses access to everything in the workspace. It has a
  confirmation step. Nobody can remove themselves or the owner, and an admin cannot remove another
  admin.
- `smartalex_revoke_invitation`: the invitation stops working. Do it when asked.

If the tool refuses because of one of these rules, pass the reason on in plain words.

## Business hours

`smartalex_get_business_hours` and `smartalex_update_business_hours`. Business hours are the days
and times the business is open, in the workspace's time zone. They decide when a live call can be
passed to a person: when the business is closed, the agent offers a callback or takes a message
instead of transferring the call. They can also limit when meetings can be booked.

They are not a campaign's calling hours; those belong to each campaign (the `call-a-list` skill).

- Read the current hours first and change only what the user asked for. "We're open 8 to 6 now"
  means the open days keep their days and get 08:00 to 18:00.
- The two switches, `transfers_only_in_hours` and `bookings_only_in_hours`, decide whether the
  hours apply to transfers and to bookings. Change them only when the user asks.
- Times are 24-hour, start before end on the same day. Hours that run past midnight are not
  supported; say so.
- **Time zone.** The hours are read in the workspace's time zone, which is UTC until someone sets
  it. When `smartalex_get_business_hours` warns it is not set, or the user says where they are,
  set `timezone` (a name like `Africa/Johannesburg` or `Europe/London`) in the same update. If you
  cannot tell the zone, ask once.
- Members and viewers can see the hours but not change them.

## Notification settings

`smartalex_get_notification_settings` and `smartalex_update_notification_settings`. These are the
signed-in person's own settings: which events reach them (a call ends, a transfer is missed, a
callback is due, a campaign finishes, money and account alerts) and on which channels (email,
WhatsApp, SMS, Slack, Telegram). Read them first for the event and channel names, and change only
what the user asked for. "Stop emailing me about every call" turns off email for that one event.

A channel that is not set up yet (a phone number for SMS or WhatsApp, a Slack or Telegram link) can
be switched on, but nothing arrives until it is set up in the web app. Say so when it applies.

These tools change only your own settings. To change someone else's, voicemail settings, quiet
hours, or to set up a channel, the user goes to the web app: **Settings → Account → Notifications**
(<https://getsmartalex.com/account?tab=notifications>) or **Settings → Workspace → Team**.

## Email for the whole workspace

`smartalex_get_workspace_notifications` shows whether the workspace is sending email at all (and
since when it is paused), the channels switched off for everyone, and whether the person can change
it. Check it first when the user asks why nobody is getting emails.

`smartalex_update_workspace_notifications` with `email_sending` pauses or resumes all email for the
workspace: receipts, notifications, reports, call reports and automation emails all stop for
everyone while it is paused. Owners and admins only. It has a preview: say exactly that, ask, then
confirm. It cannot switch other channels off for the workspace or connect
Slack; that is in the web app.

## Report emails

`smartalex_get_report_settings` shows the Sunday weekly recap to the owner, the campaign reports
(when a campaign finishes, a daily report while campaigns call, a weekly summary), who receives
them, the "Prepared for" line and the numbers left out, and for real-estate workspaces with Rex
connected the weekly broker email. With `campaign_id`, also that campaign's own list.

- **Change them**: `smartalex_update_report_settings`. Only what you pass changes. Times are 24-hour
  in the workspace's time zone; `weekday` is 1 Monday to 7 Sunday. `remove_recipients` takes
  addresses off. It has no preview: say what changes and wait for a yes.
- **Add people**: `smartalex_add_report_recipients` with the list (`campaign_reports`,
  `weekly_broker`, or `campaign` with a `campaign_id`) and the addresses. Every report on that list
  then emails them callers' names, numbers and what they said, so it has a preview: name the
  addresses and what they will receive, ask, then confirm. Use only addresses the user gave you.

No tool makes or sends a report now.

## If these tools are not available

Check the person's role first: `smartalex_get_notification_settings` shows `your_role`, and every
role has it. Usually a missing tool here means the role does not allow it, and the web app has the
same rule:

- **Viewer**: can look at the hours and the team, and change only their own notifications. Say their
  role is view-only, and that a workspace owner or admin can make the change or give them more
  access. Do not send them to the web app to do it themselves.
- **Member**: can change their own notifications, not the team or the business hours. Same answer.

Only when the role allows it, and the tool is still missing, send them to the web app: **Settings →
Workspace → Team** (<https://getsmartalex.com/account?tab=team>), **Settings → Workspace → General →
Business hours**, **Settings → Account → Notifications**, and **Insights → Reports → Settings**.
