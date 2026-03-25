---
name: subagent_to_main_report
description: Subagent→ main reporting protocol (prefer Agent Send; file fallback).
---

# Subagent → main report (skill)

Use this skill when you are a **customer-facing subagent** (e.g., LINE) and you must inform the **main/operator agent** about an escalation-worthy thread.

## Target (routing)

- **Agent ID:** `main`
- **Main session key:** `agent:main:main`

## When to report

- New non-trivial conversation worth tracking
- Stalled/expired/no clear conclusion
- User explicitly asks for a human/staff/manager
- Sensitive order inquiry / complaint / refund / payment method questions requiring staff

## Preferred delivery: Agent Send (no polling)

If you can invoke the OpenClaw CLI from your runtime, send a single structured message into the **main/operator session**:

```text
openclaw agent --agent main --message "<REPORT_TO_MAIN block>"
```

## Alternate delivery: Sessions API (if available)

If your runtime can call the Sessions API, send the same `REPORT_TO_MAIN` block to:

- `sessionKey: "agent:main:main"`

Your message must be a short structured block:

```markdown
### REPORT_TO_MAIN — <CHANNEL> — <ISO_TIMESTAMP>

- Reason: <new_conversation|stalled_or_expired|no_conclusion|user_requested_help|order_inquiry|complaint_or_sensitive|other>
- Urgency: <low|medium|high>
- Summary: <3–8 sentences; what they want, what you did, what is blocked>
- Suggested next step: <what the operator should do next>
```

Rules:
- Don’t include secrets/tokens or full transcripts.
- Include only identifiers the operator needs (order id / order link / safe thread hint).

Example (copy/paste shape):

```bash
openclaw agent --agent main --message "### REPORT_TO_MAIN — LINE — 2026-03-25T15:30+07
- Reason: user_requested_help
- Urgency: high
- Summary: …
- Suggested next step: …"
```

## Fallback delivery (only if Agent Send is not available)

Append the same `REPORT_TO_MAIN` block to:
- `memory/YYYY-MM-DD.md` (date audit trail)
- optionally also `memory/report-inbox.md` (durable inbox)

