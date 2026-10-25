---
name: technician-notify
description: Use this skill when the FM asks you to contact, message, or follow up with a technician.
disable-model-invocation: true
patterns:
  - "follow up with"
  - "message the technician"
  - "tell John"
  - "send a message to the tech"
  - "contact technician"
  - "technician notify"
metadata: {"clawdbot":{"emoji":"📨","requires":{"bins":["curl"]}}}
---

# Technician Notify Skill

Use this skill when the FM asks you to contact, message, or follow up with a technician.

## Trigger

When the FM says things like:
- "follow up with John"
- "message the technician"
- "tell John I need an update"
- "send a message to the tech"
- Any request to communicate with a named technician

## HARD RULE

**NEVER send a Telegram message directly yourself.** You must always route through the backend API.

## Procedure

1. **Extract** the technician's Telegram ID and the message content
2. **Confirm** with the FM: "Send this to John? [message text]"
3. **After FM confirms**: Call the backend API using curl:

```bash
curl -s -X POST "http://localhost:8000/api/sentry/send-technician-message" \
  -H "Content-Type: application/json" \
  -H "X-Sentry-Secret: ${SENTRY_WEBHOOK_SECRET}" \
  -H "X-Sentry-API-Key: ${SENTRY_BOT_API_KEY}" \
  -d '{
    "technician_telegram_id": "<johns_telegram_id>",
    "message": "<exact message to send>",
    "wo_code": "<wo_code_if_applicable>",
    "source": "fm_agent"
  }'
```

4. **Report** the result to the FM: "Done — message sent to John."

## Technician Telegram IDs

- Synthetic Technician: `DEMO_CHAT_ID`

## Notes

- The backend routes via the `technician` account (`@Checksentinelcontabot`) so John sees it in his separate bot chat
- The FM's message and John's message are fully isolated — the FM does NOT see a copy
- Always confirm the exact message text with the FM before sending
