# LINE order agent — HTTP quick reference

Mirror of [`../lanna-bloom-skill.md`](../lanna-bloom-skill.md) section 4. Update both when the API changes.

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
