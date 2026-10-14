---
name: connect-an-api
description: Give a SmartAlex voice agent an API tool (an HTTP Tool), so during a call it can look something up in, or send something to, another system such as an order system, a booking system or a CRM. Use when the user says "add an API tool to my agent", "let the agent check order status in our system", "connect the agent to our API", "the agent should look up X during calls", or wants to change or remove an agent's API tool. Never handles API keys in chat.
---

# Connect an agent to an API

An API tool lets the agent call an HTTPS address during a call, sending details from the
conversation (an order number, say) and using the answer. The web app calls them **HTTP Tools**.

API tools need the **Power Tools** add-on. If `smartalex_create_http_tool` is not available, check
`smartalex_get_plan_and_usage` (if you have it), say what unlocks it (**Settings → Billing**), and
stop.

## Keys never go through chat

This rule covers the whole skill.

- Never ask for, accept, repeat or store an API key, token, password or auth header value: not in
  a tool argument, not in a file, not in your reply.
- If this codebase is the API being connected, never read a key out of `.env` or config to put into
  the tool.
- If the user pastes a key into the chat anyway, do not use it, and do not quote any of it back,
  not even to say which key to replace: call it "the key you pasted". Tell them it is safer to
  replace it with a new one, because it has now been shared in a chat, and to paste the new one in
  the web app.
- Create the tool without the secret. The user adds it in the web app: **Agent Settings → HTTP
  Tools**, open the tool, then **Authentication**, and paste it into **Token** (or pick a **Saved
  key**). Direct link: <https://getsmartalex.com/home?settings=http_tools>
- Stored header values come back masked. Never try to work out the hidden value.

## Example: "add an API tool to my agent"

Work out what you can from the request and go ahead. Ask only for what cannot be guessed, such as
the address when the user gave none, or which agent when there are several and they named none.

1. **What it is for.** Take it from the request, for example "When a caller gives an order number,
   look up its status." That becomes the tool's description.
2. **The API.** If this codebase is the API, read its routes for the address and parameters.
   Otherwise use these defaults and list them when you report back:
   - **Method:** GET for looking something up, POST for sending something. A GET goes in
     `body_template` as `{"$request": {"method": "GET", "query": {"order_id": "${arg.order_id}"}}}`;
     a header goes beside it, `"headers": {"X-Api-Key": "${vault.token}"}`. A GET has no other
     fields. Without `"$request"` the tool sends a JSON POST of the template.
   - **Parameters:** what the caller gives, as text (`order_id`), used as `${arg.order_id}` in the
     query or the body.
   - **No values inside the address.** The platform cannot put a value into the web address's path:
     `${...}` there is sent as typed. When the API takes the ID in the path
     (`/orders/{order_id}`), say so plainly, and ask whether it also accepts it as a query value
     (`/orders?order_id=`) or in the body; build it that way. If it accepts neither, it cannot be
     connected yet.
   - **Sign-in:** a bearer token if the user mentioned a key or token, none if they said it needs
     none. The web app's default is bearer too, and the user can switch it to a custom header or a
     username and password in **Authentication** when they paste the key. Never ask for the value.
3. **Check for an existing one.** `smartalex_list_http_tools`. If a similar tool exists, change it
   rather than adding a second.
4. **Create it.** `smartalex_create_http_tool` with:
   - a short lowercase name using letters, digits and underscores, for example `order_status`
     (names of built-in tools such as `transfer_call` or `end_call` are refused),
   - a description that tells the agent when to use it, 10 to 500 characters,
   - the address, which must be public HTTPS and must exist on the internet (an address that does
     not resolve, localhost, private network and cloud metadata addresses are refused),
   - the parameters, and the sign-in type with no secret.
5. **Give it to the agent.** `smartalex_attach_http_tool_to_agent`, to the agent the user named, or
   their only agent. Each agent has a tool budget,
   at most 20 tools and a size limit shared with its other tools. If it is full, the user turns
   another tool off first.
6. **Show the request.** `smartalex_preview_http_tool_request` with example values shows the exact
   request the agent would send, with secrets masked. It sends nothing. Show it to the user so they
   can check the address and fields.
7. **The key, if it needs one.** Give the click path from "Keys never go through chat". Until the
   key is added, the API will refuse the agent's requests.
8. **Try it.** Suggest a test call (the `build-an-agent` skill) where the caller asks for the thing
   the tool looks up.

## Changing or removing a tool

- `smartalex_update_http_tool`: change the name, description, address, parameters or sign-in type.
  The same rules apply, and the address is checked again. Never the secret.
- `smartalex_detach_http_tool_from_agent`: that agent stops using it; other agents keep it.
- `smartalex_delete_http_tool`: deletes it for good, for every agent. It has a confirmation step. Show the preview and ask even when the request already says to do it ("delete it", "just send it"): only a yes given after the preview counts.
  If the user only wants one agent to stop using it, detach it instead.

If these tools are not available, the user can do all of this in the web app: **Agent Settings →
HTTP Tools**.
