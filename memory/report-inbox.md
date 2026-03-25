# Report inbox — durable subagent → main handoff

Append-only file. Sub-agents use it to hand off structured context to the main/operator agent.

The main agent should search this file for `### REPORT_TO_MAIN — ...` blocks mainly on startup or during explicit catch-up (and it may also contain `### OPERATOR_ALERT_PENDING — ...` blocks), then decide whether to Telegram the operator, join the LINE conversation, or follow up.

---

## Block format (subagent writes)

Sub-agents must append one block per escalation/report event.

```markdown
### REPORT_TO_MAIN — <CHANNEL> — <ISO_TIMESTAMP>

- Reason: <new_conversation|stalled_or_expired|no_conclusion|user_requested_help|order_inquiry|complaint_or_sensitive|other>
- Urgency: <low|medium|high>
- Summary: <3–8 sentences; what the customer wants, what you did, what is blocked>
- Suggested next step: <what the operator should do next>
```

Rules:
- Do not include secrets, tokens, or full LINE transcripts.
- Use identifiers that help the operator join/reply (e.g., order id / order link, or a non-sensitive thread hint).

<!-- Entries below -->

