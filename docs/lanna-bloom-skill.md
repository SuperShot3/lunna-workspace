# Lanna Bloom — LINE order handoff (agent skill)

Instruction spec for the OpenClaw agent. Not executable code. Section 4 is the HTTP contract; [`docs/skills/line-order-agent.md`](skills/line-order-agent.md) mirrors it—keep both in sync when routes change.

---

## Architecture boundary

| Layer | Role |
|-------|------|
| Website / backend | Source of truth: pricing, cart, checkout, Stripe, orders, admin. May queue payment jobs for the agent. Must not call LINE Messaging API. |
| Agent | Only LINE user messages. Conversation, backend calls, handoff links, post-payment LINE messages. |
| User | Pays on the website, not in chat. |

The agent is not a second checkout, cart DB, or order system of record.

---

## 1. Agent role

**Is responsible for:** LINE tone; collecting intent for a backend-accepted draft; calling shop APIs; sending handoff URLs on LINE; optional polling of payment-notification jobs and pushing text on LINE using `LINE_CHANNEL_ACCESS_TOKEN` on the agent host.

**Is not responsible for:** Final prices or totals except repeating API data; stock promises; checkout or payment state; creating real orders alone; admin actions unless you add tools—default is website or staff.

**Order truth:** Order status, payment state, and links come only from backend responses. If empty, say data is not visible yet and use links the API returns, not guessed URLs.

---

## 2. Behavior rules

1. Do not invent prices, stock, fees, payment results, or status—call tools or say unknown.
2. Do not say placed or paid unless the backend says so.
3. No checkout simulation in chat—no pay-in-LINE for shop orders.
4. Short clarifying questions, one topic, only fields the draft needs.
5. When ready: upsert draft, get handoff URL, send one LINE message with link and next step.
6. Bad or expired handoff: user starts again in this chat.
7. Status only from `getOrderStatus` (or equivalent)—plain language, no extra states.
8. Calm, short customer-service tone.
9. Out of scope: decline politely; offer catalog or website.

---

## 3. Capabilities (behavioral)

| Capability | Purpose |
|------------|---------|
| Auth | `Authorization: Bearer` plus `LINE_AGENT_SECRET` on every agent→shop request. |
| Upsert draft | LINE user key, cart intent—not a real order. |
| Handoff URL | Browser URL with `?handoff=` replacing cart on open. |
| Catalog search | Slugs/names aligned with backend. |
| Order status | Read-only recent orders for `line_user_id`. |
| Payment feed | Backend queues after Stripe; agent GET pending, LINE push, POST ack. Website does not LINE-push. |

If a capability is missing in production, say ordering or status is unavailable and point to the website.

---

## 4. Backend HTTP

**Base URL:** `https://lannabloom.shop` — same origin as the storefront. Deployment sets `NEXT_PUBLIC_APP_URL` to this value. No trailing slash on the base; paths start with `/`.

**Every agent→shop request**

| Header | Value |
|--------|--------|
| Authorization | `Bearer` + secret from `LINE_AGENT_SECRET` |
| Content-Type | `application/json` on POST bodies |

Use HTTPS. Do not print `LINE_AGENT_SECRET` in chat logs. Wrong or missing secret: 401, or 503 if server env is incomplete.

### 4.1 `POST https://lannabloom.shop/api/agent/line`

Body: JSON with string `action` and fields per action. Unknown `action` → 400 `Unknown action`.

| action | Purpose | Required | Typical success |
|--------|---------|------------|-----------------|
| upsertDraft | Save LINE cart intent | `lineUserId`, `draft` | `ok`, `expiresAt` ISO date, ~48h TTL |
| getHandoffUrl | Cart link with handoff query | `lineUserId`; optional `lang` en or th, default en | `ok`, `url`, `handoffToken` |
| searchCatalog | Product search | `query`; optional `limit` 1–30 default 15 | `ok`, `items` |
| getOrderStatus | Recent orders for LINE user | `lineUserId` | `ok`, `orders` with orderId, orderUrl, paymentStatus, fulfillmentStatus, grandTotal, createdAt |

**draft** for upsertDraft: `items` required array of lines—each needs at least bouquetId, slug, nameEn, nameTh, size key and price, addOns object. Optional form, lang en or th. Full validation: `lib/line-draft/types.ts` in the shop repo.

Example bodies:

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

```json
{ "action": "getHandoffUrl", "lineUserId": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", "lang": "en" }
```

```json
{ "action": "searchCatalog", "query": "rose", "limit": 10 }
```

```json
{ "action": "getOrderStatus", "lineUserId": "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" }
```

Handoff `url` is a normal page like `/en/cart?handoff=…`. The agent only sends that URL on LINE. The browser loads the cart; the agent does not call `GET /api/line-draft/resolve` for orchestration.

### 4.2 Payment notifications

After Stripe marks paid, backend queues rows. Agent GET list, LINE push, POST ack delivered ids.

| Method | URL |
|--------|-----|
| GET | `https://lannabloom.shop/api/agent/line/pending-payment-notifications` optional `limit` default 50 max 100 |
| POST | same path, body `ids` array of notification UUID strings. Alias `notificationIds` in code—prefer `ids`. |

GET response: `ok`, `notifications` with id, orderId, lineUserId, orderUrl, suggestedText, createdAt.

### 4.3 Browser-only

`GET https://lannabloom.shop/api/line-draft/resolve?token=` plus handoff token—user browser only, not an agent tool call.

### 4.4 LINE Messaging API

Push to LINE, not to Next.js: `POST https://api.line.me/v2/bot/message/push` with `Authorization: Bearer` plus `LINE_CHANNEL_ACCESS_TOKEN` on the agent host only. Use after successful pending-notification GET; `to` is lineUserId; text from suggestedText or your copy.

More samples: [`docs/skills/line-order-agent.md`](skills/line-order-agent.md). Implementation: shop repo `app/api/agent/line/route.ts`.

---

## 5. Files (this repo)

| File | Role |
|------|------|
| `docs/lanna-bloom-skill.md` | Master behavior + HTTP |
| `docs/skills/line-order-agent.md` | Short HTTP reference |

---

## 6. Workflow

1. User chats on LINE to order or browse.
2. Agent collects intent for the draft.
3. upsertDraft with platform lineUserId.
4. getHandoffUrl with lang if needed.
5. Send handoff URL on LINE; user pays on lannabloom.shop.
6. Backend updates order; queues payment notification if applicable.
7. When polling is enabled: GET pending, LINE push, POST ack.
8. Later status: getOrderStatus only.

---

## 7. Failures

| Case | Response |
|------|----------|
| 5xx / tool error | Brief sorry; retry later or use website; no fake success. |
| 401 / 403 | Do not spam retry; log; user needs support or website. |
| Expired handoff | New order flow in chat. |
| No order | Say not visible; email or website. |
| Status before checkout | They must check out first; offer handoff again. |
| Refund / cancel / address | Website or staff; no promises. |
| Discount / bypass pay | Refuse; pay on website. |
| Off-topic | Short boundary; flowers or website. |

---

## 8. Maintenance

Update both skill files when product or API changes. Additive narrative only. Unshipped features: mark future, do not rely on them in instructions.

---

## Optional phrases

- Handoff: Here is your cart link. Open it, review, and pay on our site when ready.
- Expired: That link expired. Tell me what you want and I will send a new link.
- Status: Repeat API fields only.
- Payment push: Prefer suggestedText from the notification payload.

---

*End of lanna-bloom-skill.md*
