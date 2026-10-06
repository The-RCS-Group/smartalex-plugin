---
name: webhooks
description: Send SmartAlex events (a call finished, a contact or deal was created or changed, a campaign completed) into the user's own app. Builds a signed webhook endpoint in this codebase and subscribes it. Use when the user says "notify my app when a call ends", "sync SmartAlex calls into our database", "set up a SmartAlex webhook", or "post call transcripts to Slack".
---

# SmartAlex webhooks

SmartAlex POSTs a JSON event to an HTTPS address you choose. You build the receiving endpoint in
this codebase, deploy it, then subscribe it.

Creating webhooks needs the **Power Tools** add-on. If `smartalex_create_webhook` is not available,
tell the user that, and stop.

## Events

| Event | Fires when |
|---|---|
| `call.completed` | A call ends and has been processed |
| `campaign.completed` | A campaign has worked through its list |
| `contact.created` | A contact is added, from anywhere (app, API, voice agent, import) |
| `contact.updated` | A contact's name, email, phone, company, tags or status changes; `data.changed_fields` lists which |
| `deal.created` | A deal is added |
| `deal.stage_changed` | A deal moves stage; `data.previous_stage` and `data.previous_stage_label` say from where |
| `*` | All of the above |

Subscribe only to what the user needs. A big contact import sends one `contact.created` per
contact, so make that handler cheap.

## What arrives

```http
POST /your/path
Content-Type: application/json
X-SmartAlex-Event: call.completed
X-SmartAlex-Signature: <hex HMAC-SHA256 of the raw body, keyed with the signing secret>

{"event":"call.completed","tenant_id":"...","timestamp":"2026-10-06T09:14:03.120Z","data":{ ... }}
```

## 1. Build the endpoint

Write a handler in this project's existing server framework. It must:

1. Read the **raw** request body as text before any JSON parsing. Re-serialised JSON will not match
   the signature.
2. Compute HMAC-SHA256 of the raw body with the signing secret, as lowercase hex.
3. Compare it with `X-SmartAlex-Signature` using a constant-time compare
   (`crypto.timingSafeEqual` in Node, `hmac.compare_digest` in Python). Reject a mismatch with 401.
4. Parse the body, act on `event`, and return 2xx quickly. Do slow work in a queue or background
   job; SmartAlex waits 10 seconds at most.
5. Make the handler safe to run twice for the same event, and do not rely on a failed delivery being
   sent again: it is not retried. Repeated failures switch the webhook off.
6. Read the secret from an environment variable, for example `SMARTALEX_WEBHOOK_SECRET`.

Node example of the check:

```ts
import { createHmac, timingSafeEqual } from 'node:crypto';

export function isFromSmartAlex(rawBody: string, signature: string | null, secret: string): boolean {
  if (!signature) return false;
  const expected = createHmac('sha256', secret).update(rawBody).digest('hex');
  const a = Buffer.from(expected, 'hex');
  const b = Buffer.from(signature, 'hex');
  return a.length === b.length && timingSafeEqual(a, b);
}
```

Add a test that signs a sample body with a test secret and checks both a valid and a tampered
request.

## 2. Subscribe it

The address must be public HTTPS. A localhost address will not receive events; if the endpoint is
not deployed yet, use the user's tunnel (for example ngrok or Cloudflare Tunnel) or wait until it is
deployed.

Call `smartalex_create_webhook` with `url`, `event_types`, and a short `description`.

The response contains a **`signing_secret`, shown once only**. Immediately:

- put it in the environment the endpoint runs in, or in an untracked `.env` file that `.gitignore`
  already covers, as `SMARTALEX_WEBHOOK_SECRET`;
- never write it into a tracked file, a commit, a log line or your reply.

If the secret is lost, delete the webhook (`smartalex_delete_webhook`, after the user agrees) and
create a new one.

## 3. Check

Run `smartalex_list_webhooks` and confirm the subscription is active. Its failure count and last
delivery time show whether events are arriving. If it shows inactive after failures, fix the
endpoint, delete the webhook (with the user's yes) and create a new one. To see a real `call.completed`, the user can place
a test call to their agent.
