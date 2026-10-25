---
name: sentry-call-logging
description: Log facilities defects from non-technical staff. Deterministic flow: classify → confirm → log. No LLM decisions between steps.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"📋","requires":{"bins":["python3"]}}}
---

# Sentry Call Logging — Deterministric Flow

Run the correct command based on where the user is in the flow. NEVER compose text.

## Flow states

The user moves through 3 states. Run the matching command.

### STATE 1: NEW COMPLAINT

User reports an issue. Always run classify first:

```bash
python3 ./tools/call_log.py classify "<exact user message>"
```

Read the result:
- If `is_facilities_issue: false` or `no_match: true` → Tell user: "I wasn't able to match that to a known facilities issue. I've flagged it for the facilities supervisor."
- Otherwise → Show this exact format, replacing the values from classify:

```
Issue: [sub_category]
Location: [from user message — desk number or floor/area]
Priority: [priority, UPPERCASE]

Shall I log this?
```

Wait for yes/yep/sure/go. Do NOT proceed without confirmation.

---

### STATE 2: USER CONFIRMS

User said yes. Run the log command:

```bash
python3 ./tools/call_log.py log --complaint "<sub_category> at <location>" --desk "<desk_number>" --user "<display_name>" --user-id "<telegram_user_id>" --channel telegram
```

Read the JSON result. If `success: true` → Output the result EXACTLY as JSON:

```
{"success": true, "wo_code": "...", "issue": "...", "location": "...", "assigned_to": "...", "priority": "..."}
```

If `success: false` → Tell user the issue could not be logged and they should try again.

---

### STATE 3: LOCATION NEEDED

User didn't provide a location. Ask once:

"What's your desk number or where in the building are you?"

Then → STATE 1 with the updated message.

---

## HARD RULES

1. **ALWAYS run classify FIRST** on any new complaint message.
2. **NEVER skip the confirm step** — ask, wait for yes, then log.
3. **Output JSON verbatim** from the log command. Do NOT reformat, do not add text around it.
4. **Do NOT ask about priority** — it comes from classify.
5. **Never invent WO numbers** — only use what `call_log.py log` returns.
6. If classify fails or returns `no_match: true`, stop and escalate.