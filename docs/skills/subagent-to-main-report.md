# Subagent → main agent — report (behavior spec)

Instruction spec for OpenClaw. **Not** a shop HTTP API. There is no fixed “if X then always report” rule in code—the **model** reads this file and **decides** when a report is warranted, what to include, and how urgent it is.

Use this when a **customer-facing** session (for example LINE order support, “Lunna” on LINE) should surface something to the **main agent** (operator session: dashboard, direct chat, or any session that can act on behalf of the business).

---

## Roles

| Role | Typical session | Responsibility |
|------|-----------------|----------------|
| **Main agent** | Direct operator chat, heartbeat, tooling | Strategy, memory curation, joining tough threads, business decisions |
| **Customer subagent** | LINE (or similar) with the shopper | Orders, catalog, handoff links, tone; **reports** when the operator should know or step in |

The subagent does not replace the main agent for “whole business” memory; it **informs** the main agent when human attention may be needed.

---

## What “report” means

A **report** is a deliberate handoff of **structured context** from the customer subagent to the main agent so the operator can:

- know a new or important customer thread exists,
- know a thread stalled, expired, or ended without a clear outcome,
- know the customer asked for **help** / a **human** / **escalation** so someone can **join** the conversation or follow up.

The subagent **still** replies to the customer as usual (short acknowledgment, next step, or “someone will follow up”) unless this file says otherwise for a specific scenario.

---

## When to consider reporting (discretionary)

Use judgment. These are **signals**, not a checklist you must fire on every message.

### 1. New conversation started

**Consider reporting** when a **new** or **freshly reopened** customer conversation begins and there is a **non-trivial** intent: first-time buyer asking about products, starting an order path, or returning after a long gap with a new request.

**Do not** report on every automated “hello” or single emoji with no business intent—avoid noise.

### 2. Conversation expired, stalled, or no conclusion

**Consider reporting** when:

- a **handoff link** or draft is **expired** or clearly invalid and the customer has not successfully moved to checkout, **or**
- after a **reasonable** exchange there is **no clear outcome** (no handoff sent, no resolution, no polite close), **or**
- the customer **ghosts** after you asked for something critical (payment choice, delivery detail) and enough time has passed that follow-up from a human is valuable.

“Reasonable” and “enough time” are **contextual** (busy hours, urgency in their words, order cutoffs)—decide in the moment.

### 3. User asks for help (human / escalation)

**Report** when the customer **explicitly** asks to:

- speak to a **person** / **human** / **staff** / **manager**,
- get **help** beyond product selection (complaint, payment problem, delivery dispute),
- **escalate** or says they are **stuck** / **frustrated** and need someone else.

Pair the report with a **calm** customer-facing reply (per `SOUL.md` / `BUSINESS.md`): you are not abandoning them; you are alerting the operator so they can **join** or **reach out**.

### 4. Order or inquiry the operator should know about

**Consider reporting** for:

- a **new order-related** inquiry (status, change, problem) when `getOrderStatus` or policy leaves ambiguity,
- **sensitive** topics (complaints, refunds, legal threats) per `BUSINESS.md` escalation,
- anything you **cannot** honestly complete without a human decision.

---

## When **not** to report

- Trivial chit-chat with no business follow-up.
- You already reported the **same** thread for the **same** issue recently—**update** existing memory or add a short follow-up note instead of duplicating spam.
- Pure FAQ answered successfully with catalog + links.

---

## What to put in a report (content)

Write for a **busy operator**. Be concise and structured.

Suggested fields (include what applies):

- **When:** ISO or local time + timezone if known.
- **Channel:** e.g. LINE.
- **Thread / user key:** internal id the main agent can correlate (e.g. LINE user id **only** in operator-safe contexts—never paste secrets into public logs; follow workspace privacy rules).
- **Reason category:** one or more of `new_conversation`, `stalled_or_expired`, `no_conclusion`, `user_requested_help`, `order_inquiry`, `complaint_or_sensitive`, `other` (short gloss).
- **Summary:** 3–8 sentences: what the customer wants, what you did, what is blocked.
- **Urgency:** `low` | `medium` | `high` (customer language, deadlines, distress).
- **Suggested next step for main agent:** e.g. “Reply on LINE within 30m,” “Offer callback,” “Review order LB-… in admin.”

---

## How to deliver the report (no hardcoded channel)

Pick the **best available** mechanism your runtime supports; if none is explicit, use the **durable fallback** below.

1. **Native notify / parent session** — If OpenClaw (or your stack) exposes a way to message the main session or operator, use it and include the structured content above.
2. **Daily memory file (fallback)** — Append to `memory/YYYY-MM-DD.md` a clearly marked block so the main agent’s **heartbeat** or next session can ingest it:

```markdown
### REPORT_TO_MAIN — LINE — 2026-03-25T14:30+07

- **Reason:** user_requested_help
- **Urgency:** high
- **Summary:** …
- **Suggested next step:** …
```

Use a consistent heading pattern (`REPORT_TO_MAIN`) so searches stay easy.

---

## Main agent: what to do with a report

When you see a report (notify, memory, or dashboard):

1. **Read** the summary and urgency.
2. **Decide:** reply on LINE, schedule follow-up, update `MEMORY.md` / `BUSINESS.md` if policy-relevant, or take no action if already resolved.
3. **Join the conversation** when the customer asked for help: speak as the business, briefly introduce if needed, and continue the thread—do not duplicate a wall of internal notes on LINE.

---

## Files to keep aligned

| File | Role |
|------|------|
| This file | Full behavior for report / escalation to main |
| [`../lanna-bloom-skill.md`](../lanna-bloom-skill.md) | LINE order skill; architecture pointer |
| [`line-order-agent.md`](line-order-agent.md) | Shop HTTP only—**not** where report lives |
| `SOUL.md` | Tone and when to escalate to humans (customer-facing) |
| `BUSINESS.md` | Escalation and policy boundaries |

---

*End of subagent-to-main-report.md*
