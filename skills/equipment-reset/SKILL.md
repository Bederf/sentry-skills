---
name: equipment-reset
description: Remote reset equipment fault status. Use ONLY for /reset_ commands. Bypasses model — runs tool directly.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"🔄","requires":{"bins":["python3"]}}}
---

# Equipment Reset — Direct Passthrough

Remote fault reset for equipment via `/reset_` Telegram command.

## /reset_ Commands — VERBATIM PASSTHROUGH

When user sends `/reset_{code}` (e.g., `/reset_DEMO-001_FCU_341`):

1. Extract the equipment code from the message (everything after `/reset_`)
2. Extract the sender's Telegram user ID from the message context
3. Run this command:
```bash
python3 ./tools/ops_reset.py DEMO-001_FCU_341 TECHNICIAN_USER_ID
```
4. **Send the tool output EXACTLY as printed. Do NOT:**
   - Reformat, rephrase, summarise, or restructure the output
   - Add diagnostic interpretations or likely causes
   - Add greetings, preambles, or sign-offs around the output

**The tool output is complete and ready to send. Just output it verbatim.**

## CRITICAL RULES

- ALWAYS run the tool command. NEVER guess or fabricate reset results.
- Send the tool output VERBATIM. No additions, no reformatting.
- If the tool returns an error, report that error as-is.
- FIRE and GEN equipment resets are blocked for safety — the tool handles this automatically.
