# LINE order agent — HTTP quick reference

Mirror of [`../lanna-bloom-skill.md`](../lanna-bloom-skill.md) section 4. Update both when the API changes.

---

## Cart and handoff — mandatory

- **Never** tell the customer you cannot show the cart or that a cart link is impossible. The website cart is the source of truth; your job is to **call the shop tools** and send the URL the API returns.
- When the user asks for their cart, checkout, or a link to continue: ensure draft intent is reflected with **`upsertDraft`** (if there are items or you just updated the draft), then call **`getHandoffUrl`** and **paste the `url` from the success response** on its own line in LINE.
- If there is no draft yet, briefly confirm what to add, **upsertDraft**, then **getHandoffUrl** — still send the link; do not stop at “I can’t show a cart.”
- **`lang` in `draft` and `getHandoffUrl` must match the language you are using in chat** (`en` or `th`). See `SOUL.md` / `MEMORY.md` for reply language rules.

---

## When a shop API call sends no reply or fails

After **any** agent→shop request (`POST /api/agent/line`, payment GET/POST) if the tool returns nothing, errors, times out, or you cannot parse JSON success:

1. **Append one line** to **`memory/shop-api-log.md`** using the format in that file’s header (timestamp, `action=…`, `result=` such as `no_response`, `timeout`, `http_503`, `parse_error`, short note). Do not log secrets.
2. In the customer message: apologize briefly, say the link could not be generated right now, and give the **manual fallback**: catalog or cart base on the site — e.g. `https://lannabloom.shop/en/cart` or `https://lannabloom.shop/th/cart` matching their language — and ask them to try again in chat in a moment if they prefer a handoff link.
3. Do **not** invent a `handoff` query or token; only send `url` from a successful **`getHandoffUrl`** response.

---

## Base

`https://lannabloom.shop` — matches `NEXT_PUBLIC_APP_URL` in production. No trailing slash.

Agent must call this host for all shop APIs below.

---

## Environment

| Variable | Where | Use |
|----------|--------|-----|
| `NEXT_PUBLIC_APP_URL` | Website deploy | Should be `https://lannabloom.shop` |
| `LINE_AGENT_SECRET` | Website + agent | Bearer secret for shop APIs |
| `LINE_CHANNEL_ACCESS_TOKEN` | Agent host only | LINE push |

---

## Headers — every agent → shop request

```http
Authorization: Bearer LINE_AGENT_SECRET
Content-Type: application/json
```

Use the real secret value from env, not the literal string `LINE_AGENT_SECRET`. HTTPS only. Never log the token.

---

## POST https://lannabloom.shop/api/agent/line

### upsertDraft

```json
{
  "action": "upsertDraft",
  "lineUserId": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "draft": {
    "items": [
      {
        "bouquetId": "bouquet-id",
        "slug": "product-slug",
        "nameEn": "Name EN",
        "nameTh": "Name TH",
        "size": { "key": "standard", "price": 0 },
        "addOns": { "cardType": null, "cardMessage": "", "wrappingPreference": null }
      }
    ],
    "lang": "en"
  }
}
```

Success shape: `ok`, `expiresAt` ISO string. Details: `lib/line-draft/types.ts` in the shop repo.

### getHandoffUrl

```json
{ "action": "getHandoffUrl", "lineUserId": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", "lang": "en" }
```

`lang`: `en` or `th`, default `en`. Success includes `url` like `https://lannabloom.shop/en/cart?handoff=…` and `handoffToken`.

### searchCatalog

```json
{ "action": "searchCatalog", "query": "rose", "limit": 10 }
```

`limit` 1–30, default 15. Success: `ok`, `items`.

### getOrderStatus

```json
{ "action": "getOrderStatus", "lineUserId": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" }
```

Success: `ok`, `orders` with orderId, orderUrl, paymentStatus, fulfillmentStatus, grandTotal, createdAt.

### Errors

400 unknown action. 401 or 503 auth / env issues.

---

## Payment notifications

**GET** `https://lannabloom.shop/api/agent/line/pending-payment-notifications`  
Query `limit` optional, default 50, max 100.

**POST** same URL:

```json
{ "ids": ["uuid-one", "uuid-two"] }
```

`notificationIds` works as alias; prefer `ids`.

GET response includes `notifications` with id, orderId, lineUserId, orderUrl, suggestedText, createdAt.

---

## Browser-only

```http
GET https://lannabloom.shop/api/line-draft/resolve?token=HANDOFF_TOKEN
```

User browser opens this. Agent does not use it as a tool.

---

## LINE push

```http
POST https://api.line.me/v2/bot/message/push
Authorization: Bearer LINE_CHANNEL_ACCESS_TOKEN
Content-Type: application/json
```

```json
{
  "to": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "messages": [{ "type": "text", "text": "Message from suggestedText or custom" }]
}
```

Use token from env on the agent host only.

---

## Shop code

`app/api/agent/line/route.ts` in the Lanna Bloom Next.js repo.

---

## Report to main agent (not HTTP)

Escalation and **operator awareness** (new conversation, stalled/expired/no conclusion, user asks for human help) are **not** `POST /api/agent/line` actions. They are defined in **[`subagent-to-main-report.md`](subagent-to-main-report.md)** — behavior only, no shop route.
