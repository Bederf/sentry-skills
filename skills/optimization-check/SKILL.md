---
name: optimization-check
description: Check and manage AI optimization recommendations from SENTRY. Approve or reject pending suggestions.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"⚡","requires":{"bins":["python3"]}}}
---

# Optimization Check

Check, approve, or reject AI optimization recommendations from SENTRY.

## Commands

**Check for pending recommendations:**
```bash
python3 ./tools/ops_optimization_check.py check [site]
```

**Show optimization status:**
```bash
python3 ./tools/ops_optimization_check.py status
```

**Approve pending recommendations:**
```bash
python3 ./tools/ops_optimization_check.py approve <site>
```

**Reject pending recommendations:**
```bash
python3 ./tools/ops_optimization_check.py reject <site>
```

**Trigger new AI analysis:**
```bash
python3 ./tools/ops_optimization_check.py analyze <site>
```

## Output Handling

Present the tool output clearly for Telegram readability:
1. Pending recommendations (if any)
2. Recommended changes and projected savings
3. Confidence level
4. Approval/rejection confirmation

Do NOT send raw output verbatim — format for readability.
Do NOT fabricate or embellish beyond what the tool returns.

## CRITICAL: Do Not Hallucinate

- ALWAYS run the tool command. NEVER guess recommendations or savings figures.
- If no recommendations pending, say "No pending recommendations."
- If the tool returns an error, report that error as-is.
- The tool queries the SENTRY BMS API — it is the single source of truth.
