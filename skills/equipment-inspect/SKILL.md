---
name: equipment-inspect
description: Send technician to inspect equipment. Use ONLY for /inspect_ commands. Bypasses model — runs tool directly.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"🔍","requires":{"bins":["python3"]}}}
---

# Equipment Inspect — Direct Passthrough

Create inspection work order and notify technician via `/inspect_` Telegram command.

## /inspect_ Commands — VERBATIM PASSTHROUGH

When user sends `/inspect_{code}` (e.g., `/inspect_DEMO-001_FCU_341`):

1. Extract the equipment code from the message (everything after `/inspect_`)
2. Extract the sender's Telegram user ID from the message context
3. Run this command:
```bash
python3 ./tools/ops_inspect.py DEMO-001_FCU_341 TECHNICIAN_USER_ID
```
4. **Send the tool output EXACTLY as printed. Do NOT:**
   - Add equipment details or health data
   - Add diagnostic interpretations
   - Add greetings, preambles, or sign-offs
   - Run ops_query.py — this is NOT /info_

**The tool output is a one-line ack. Just output it verbatim.**

## CRITICAL RULES

- /inspect_ is NOT /info_. Do NOT show equipment data.
- ALWAYS run the tool command. NEVER guess or fabricate results.
- The tool creates the WO and notifies the technician automatically.
- Send the tool output VERBATIM — one line, nothing else.
