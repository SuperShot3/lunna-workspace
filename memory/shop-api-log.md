# Shop API tool log (agent-written)

Append-only log when agent→shop HTTP calls fail, time out, or return no usable body. **Never** paste `LINE_AGENT_SECRET`, tokens, or full request bodies.

## Format (one block per incident)

```
YYYY-MM-DDTHH:mm:ssZ | action=<upsertDraft|getHandoffUrl|searchCatalog|getOrderStatus|GET pending-payment-notifications> | result=<no_response|timeout|http_STATUS|parse_error> | note=<short plain text>
```

Example:

```
2026-03-25T12:00:00Z | action=getHandoffUrl | result=timeout | note=No response from tool after request
```

---

<!-- Entries below -->

