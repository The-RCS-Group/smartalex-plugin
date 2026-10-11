---
name: team-and-settings
description: Manage who is in the SmartAlex workspace and how it runs. Invite a colleague, change someone's role, remove someone, cancel an invitation, set the business hours, and change your own notification settings. Use when the user says "invite my colleague", "add Sarah to SmartAlex", "who's on my team", "make Tom an admin", "remove Jo from the workspace", "cancel that invite", "we're open 8 to 6 now", "change our business hours", "stop emailing me about every call", or "turn on WhatsApp alerts".
---

# Team and workspace settings

This mirrors the web app's **Settings → Team**, the **Business hours** section of **Settings →
Profile**, and **Settings → Notifications**. For every tool here with a `confirm_token`, follow "The
confirmation step" in the `smartalex` skill.

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
   token only after a clear yes.
4. Tell the user the invitation is sent and the link in it works for 7 days.

**Trial limit.** On a trial, a workspace can invite up to 2 people (people already invited count,
the owner does not). When it is hit the tool says: "Your trial lets you invite up to 2 team
members. Choose a plan to invite more." Pass that on, with **Settings → Billing**
(<https://getsmartalex.com/account?tab=billing>). Do not retry.

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

These tools change only your own settings. To change someone else's, the workspace-wide email and
voicemail settings, quiet hours, or to set up a channel, the user goes to the web app: **Settings →
Notifications** (<https://getsmartalex.com/account?tab=notifications>) or **Settings → Team**.

## If these tools are not available

Check the person's role first: `smartalex_get_notification_settings` shows `your_role`, and every
role has it. Usually a missing tool here means the role does not allow it, and the web app has the
same rule:

- **Viewer**: can look at the hours and the team, and change only their own notifications. Say their
  role is view-only, and that a workspace owner or admin can make the change or give them more
  access. Do not send them to the web app to do it themselves.
- **Member**: can change their own notifications, not the team or the business hours. Same answer.

Only when the role allows it, and the tool is still missing, send them to the web app: **Settings →
Team** (<https://getsmartalex.com/account?tab=team>), **Settings → Profile → Business hours**, and
**Settings → Notifications**.
