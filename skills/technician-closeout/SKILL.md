---
name: technician-closeout
description: Guided inspection closeout when technician sends "done #WO-XXXX". Multi-step checklist, one item at a time, then AI diagnosis to FM.
disable-model-invocation: true
patterns:
  - "done #WO"
  - "done #WO-"
  - "WO- complete"
  - "work order complete"
  - "closeout"
metadata: {"clawdbot":{"emoji":"✅","requires":{"bins":["curl"]}}}
---

# Technician Closeout — Guided Inspection Checklist

Stateful, multi-turn inspection debrief triggered when a technician sends `done #WO-XXXX` after completing work on-site.

## Trigger

Matches any of:
- `done #WO-XXXX`
- `done WO-XXXX`
- `done #WO-2026-XXXX`
- `done WO-2026-XXXX`

Case-insensitive. The WO code is everything after `done` (strip `#` if present).

## Flow — Step by Step

### 1. Extract WO Code

Parse the WO code from the message. Normalise to `WO-2026-XXXX` format if needed.

### 2. Look Up Work Order

```bash
curl -s -H "X-Sentry-Secret: {SENTRY_WEBHOOK_SECRET}" \
  -H "X-Sentry-API-Key: {SENTRY_BOT_API_KEY}" \
  "${SENTRY_API_URL}/api/sentry/work-order/detail?code={WO_CODE}"
```

Extract `equipment_code` and derive the equipment type from it. If the response
contains a non-empty `work_order.checklist_items` array, use those persisted
items exactly; they are the assignment contract for this work order. For a
baseline work order (title starts with `Baseline:`), never call the generic
equipment checklist endpoint or use the legacy service-feedback template.

For baseline work orders, send the persisted item question directly and collect
free-text/structured evidence. Do not invoke shell/Python helpers to print the
prompt or reduce the item to an `ok`/`warning`/`critical` button. Preserve the
raw answer, measurements, units, operating context, observations, and
attachments against the current work-order item.

While processing an inspection answer, never run port discovery, localhost
probes, service-health diagnostics, or arbitrary shell commands. Evidence must
use the closeout API path.

**If WO not found:** Reply "Work order not found. Check the WO number and try again." — stop here.

### 3. Fetch Equipment-Specific Checklist

```bash
curl -s -H "X-Sentry-Secret: ${SENTRY_WEBHOOK_SECRET}" \
  ${SENTRY_API_URL}/api/sentry/inspection-checklist/{equipment_type}
```

Returns a list of checklist items with `item_id` and `question` fields.

**If equipment type not recognised:** The API returns a default checklist (visual, electrical, sensors, controls, parameters, findings).

**If API unavailable:** Reply "Couldn't reach SENTRY. Please try again." — stop here.

### 4. Prompt ONE Item at a Time

Send each checklist item as a separate message. Keep prompts short — the technician is on-site using a mobile phone.

**Format:** `"Item {n}/{total}: {short_question} ({options})"`

Examples:
- `Item 1/6: Filter? (Clean / Dirty / Blocked)`
- `Item 2/6: Fan? (Normal / Noisy / Not running)`
- `Item 3/6: Valve? (Operating / Stuck / Leaking)`
- `Item 4/6: Thermostat? (Accurate / Drifting / Unresponsive)`

**Rules:**
- ONE item at a time. Do NOT send the full list.
- Do NOT skip items. Every item must be answered before moving to the next.
- Do NOT fabricate checklist items. Use exactly what the API returned.
- Wait for the technician's response before sending the next item.
- Accept text, photos, or voice-to-text as valid responses.

### 5. Map Status from Answer

For baseline work orders, structured evidence is valid even when it cannot be
mapped to a status. Store the raw response and advance after persistence. Use
the status normalizer below only for non-baseline service-feedback checklists.

For each technician answer, assign a status:

| Status | When |
|--------|------|
| `ok` | Normal, clean, operating, accurate, within spec, good |
| `warning` | Dirty, noisy, drifting, worn, minor leak, degraded |
| `critical` | Blocked, not running, stuck, failed, major leak, tripped |

Use judgement — the technician may not use exact option words. Map their natural language to the closest status.

### 6. Save Results

After ALL items are collected, POST to:

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "X-Sentry-Secret: ${SENTRY_WEBHOOK_SECRET}" \
  ${SENTRY_API_URL}/api/sentry/inspection-result \
  -d '{
    "equipment_code": "DEMO-001-FCU-301",
    "work_order_code": "WO-2026-0030",
    "technician_name": "Synthetic Technician",
    "telegram_user_id": "DEMO_CHAT_ID",
    "items": [
      {"item_id": "filter_condition", "question": "Filter Condition", "answer": "Blocked", "status": "critical"},
      {"item_id": "fan_operation", "question": "Fan Operation", "answer": "Noisy", "status": "warning"}
    ],
    "ai_diagnosis": "Blocked filter causing restricted airflow, combined with noisy fan suggesting bearing wear.",
    "recommendations": "Replace filter immediately, schedule fan bearing inspection within 7 days."
  }'
```

**Required fields:**
- `equipment_code` — from the WO lookup
- `work_order_code` — as provided by the technician
- `technician_name` — from Telegram display name
- `telegram_user_id` — from Telegram message sender ID (for audit trail)
- `items` — array of all checklist responses with `item_id`, `question`, `answer`, `status`
- `ai_diagnosis` — your analysis (see step 7)
- `recommendations` — your recommended next actions (see step 7)

### 7. Generate AI Diagnosis

This is the ONE interpretive step in this skill. Before POSTing results, analyse the collected answers:

1. **Cross-reference answers.** Look for compound failures (e.g., blocked filter + noisy fan = restricted airflow causing motor strain).
2. **Consider equipment type context.** A chiller with low refrigerant and high compressor vibration is more urgent than either alone.
3. **Set severity.** If any item is `critical`, the overall diagnosis should flag urgency. Multiple `warning` items together may escalate to urgent.
4. **Be specific.** Reference the actual findings, not generic advice.

### 8. Notify FM

After saving results, send the FM a diagnosis summary using:

```bash
sentrybot message send --target DEMO_CHAT_ID
```

**FM notification format:**

```
Inspection Complete: {equipment_code}
WO: {work_order_code} | Tech: {technician_name}

Findings:
- {item}: {answer} [{status}]
- {item}: {answer} [{status}]
...

Diagnosis: {ai_diagnosis}

Recommended: {recommendations}

/info_{equipment_code_underscored} - View current status
/WO_{equipment_code_underscored} - Raise follow-up WO
```

**Rules for FM notification:**
- Highlight abnormalities — do not list items that are all OK unless everything is OK.
- If everything is OK: "All items passed. No follow-up required."
- Include equipment code and WO number for traceability.
- End with relevant slash commands for the FM to act on.
- Equipment codes in slash commands use underscores (e.g., `DEMO-001_FCU_301`).

## Equipment-Specific Checklists — Reference

The API returns these, but for context:

| Type | Items | Key Checks |
|------|-------|------------|
| **chiller** | 8 | Refrigerant, condenser coils, evaporator, compressor vibration, oil level, electrical, controls, performance |
| **ahu** | 8 | Filters, belt tension, bearing condition, damper operation, coil condition, VFD, drain pan, controls |
| **fcu** | 6 | Filter, fan motor, valve operation, thermostat, condensate, electrical |
| **generator** | 8 | Oil level, coolant, battery, fuel system, belt tension, exhaust, load test, controls |
| **pump** | 6 | Bearing noise, seal condition, vibration, pressure differential, coupling, electrical |
| **ups** | 6 | Battery bank, capacitor health, cooling, voltage readings, bypass operation, alarm history |
| **vav** | 6 | Damper operation, actuator, airflow sensor, controller, zone temp, duct condition |

**Default** (unknown type): Visual inspection, electrical connections, sensor readings, control response, operating parameters, document findings.

## Hard Rules

1. **ONE item at a time.** Never send the full checklist in one message.
2. **Keep prompts short.** Techs are on-site, on mobile. No essays.
3. **Do NOT skip items.** Every item must be answered.
4. **Do NOT fabricate checklist items.** Use the API response only.
5. **Always include `telegram_user_id`** in the POST body for audit trail.
6. **Status mapping is required for non-baseline closeouts.** Baseline items
retain structured evidence and may use `answered` rather than a severity
status; unanswered items remain `skipped`.
7. **AI diagnosis is required.** Analyse the full set of answers before POSTing.
8. **FM notification is required.** Always send the summary after saving.

## Error Handling

| Error | Response |
|-------|----------|
| WO not found | "Work order not found. Check the WO number and try again." |
| Equipment type not recognised | API returns default checklist — proceed normally. |
| API unavailable | "Couldn't reach SENTRY. Please try again." |
| Technician abandons mid-checklist | After 30 minutes of no response, remind once. After 60 minutes, save partial results with note "Checklist incomplete — {n}/{total} items answered." |
| POST fails | Retry once. If still failing: "Results saved locally. I'll sync when SENTRY is back." |

## Example Conversation

```
Tech: done #WO-2026-0030
Bot:  Starting closeout for WO-2026-0030 (DEMO-001-FCU-301, FCU).
      Item 1/6: Filter? (Clean / Dirty / Blocked)

Tech: blocked
Bot:  Item 2/6: Fan? (Normal / Noisy / Not running)

Tech: bit noisy but running
Bot:  Item 3/6: Valve? (Operating / Stuck / Leaking)

Tech: operating fine
Bot:  Item 4/6: Thermostat? (Accurate / Drifting / Unresponsive)

Tech: accurate
Bot:  Item 5/6: Condensate? (Clear / Slow / Blocked)

Tech: clear
Bot:  Item 6/6: Electrical? (Normal / Loose / Damaged)

Tech: normal
Bot:  All items recorded. Saving results and notifying FM.
```

FM then receives:
```
Inspection Complete: DEMO-001-FCU-301
WO: WO-2026-0030 | Tech: Synthetic Technician

Findings:
- Filter: Blocked [CRITICAL]
- Fan: Noisy but running [WARNING]
- Valve, Thermostat, Condensate, Electrical: OK

Diagnosis: Blocked filter restricting airflow, causing fan motor strain
(noisy operation). Immediate filter replacement needed to prevent
motor burnout.

Recommended: Replace filter, monitor fan noise after replacement.
If noise persists, schedule bearing inspection.

/info_DEMO-001_FCU_301 - View current status
/WO_DEMO-001_FCU_301 - Raise follow-up WO
```
