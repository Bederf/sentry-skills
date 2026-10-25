---
name: health-monitor
description: >
  Proactive building health check triggered by cron schedule.
  Scans all equipment for anomalies, filters by severity, and
  pushes digest to FM via Telegram. Only fires during occupied
  hours. Only notifies when something needs attention.
disable-model-invocation: true
---

# Health Monitor — Proactive Building Check

## Purpose

This skill runs on a cron schedule (not user-triggered). Sentry wakes up,
runs health checks across all monitored equipment, and sends a digest to
the FM chat ONLY if there are items requiring attention.

**Principle:** Intervention-only. If everything is healthy, stay silent.

## Trigger

- Cron wake-up (every 2 hours, 06:00–18:00 SAST, Mon–Fri)
- NOT triggered by user messages
- Occupied hours gating is enforced by the cron schedule itself

## Execution

### Step 1: Check backend health

```bash
curl -sf http://localhost:8000/api/health > /dev/null
```

If down, send this to FM chat and stop:

```bash
sentry message send --channel telegram --target DEMO_CHAT_ID \
  --message "⚠️ Health check failed — unable to reach SENTRY API. Check system health dashboard."
```

### Step 2: Run the health monitor tool

```bash
python3 ./tools/ops_health_alert.py monitor
```

This handles everything:
- Queries all equipment health (Supabase → API fallback)
- Filters by severity: CRITICAL < 70%, WARNING 70–89%, HEALTHY ≥ 90%
- Applies throttling (critical: 30 min, warning: 60 min)
- Sends Telegram to FM chat (DEMO_CHAT_ID) only if action is needed
- Stays silent if everything is healthy

### Step 3: Pending work orders

```bash
curl -s http://localhost:8000/api/sentry/work-order/pending \
  -H "X-Sentry-Secret: ${SENTRY_WEBHOOK_SECRET}" \
  -H "X-Sentry-API-Key: ${SENTRY_BOT_API_KEY}"
```

If count > 0, include in digest. The monitor tool handles formatting and send.

## Severity reference

| Level    | Health Score | Action                          |
|----------|--------------|---------------------------------|
| CRITICAL | < 70%        | Notify FM immediately           |
| WARNING  | 70–89%       | Digest (throttled to 60 min)    |
| HEALTHY  | ≥ 90%        | Silent                          |

Active alerts with severity `critical` or `high` → treat as CRITICAL regardless of score.

## Anti-hallucination

- Only report what the tool returns — never fabricate scores or predictions
- If tool errors, send backend-down message and stop
- Never suppress or downgrade alerts
- If uncertain about severity, round UP

## Operational scope

- Do NOT attempt to fix issues — report only
- Do NOT debug Python errors from the tool
- FM_CHAT_ID and credentials come from TOOLS.md
