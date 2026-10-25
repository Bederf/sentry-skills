---
name: equipment-note
description: Add maintenance note to equipment. Use ONLY for /note_ commands. Bypasses model — runs tool directly.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"📝","requires":{"bins":["python3"]}}}
---

# Equipment Note — Direct Passthrough

Add a maintenance note to equipment via `/note_` Telegram command.

## /note_ Commands — VERBATIM PASSTHROUGH

When user sends `/note_{code}` (e.g., `/note_DEMO-001_FCU_341`):

1. Extract the equipment code from the message (everything after `/note_`)
2. Run this command (prompt mode — no note text yet):
```bash
python3 ./tools/ops_note.py DEMO-001_FCU_341
```
3. **Send the tool output EXACTLY as printed.** It will ask the user to reply with their note text.

## Saving the note (follow-up)

When the user replies with their note text after a `/note_` prompt, save it:
```bash
python3 ./tools/ops_note.py DEMO-001_FCU_341 TECHNICIAN_USER_ID "their note text here"
```
Send the tool output VERBATIM.

## CRITICAL RULES

- ALWAYS run the tool command. NEVER guess or fabricate results.
- Send the tool output VERBATIM. No additions, no reformatting.
- If the tool returns an error, report that error as-is.
