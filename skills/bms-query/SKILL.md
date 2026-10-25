---
name: bms-query
description: Query SENTRY BMS building management data. Use for equipment status, alerts, temperatures, work orders, building overview, and any BMS-related query from DEMO-001 (Synthetic Office Building) or any other site. NEVER use web search for building-related queries — always use this skill first.
disable-model-invocation: true
patterns:
  - "DEMO-001"
  - "S001"
  - "site 002"
  - "site 001"
  - "Demo Building"
  - "site-002"
  - "site-001"
  - "building overview"
  - "/info_"
  - "/inspect_"
  - "/WO_"
  - "chiller"
  - "FCU"
  - "AHU"
  - "VAV"
  - "zone"
  - "equipment status"
  - " BMS "
  - "alerts"
  - "work orders"
  - "endpoints"
  - "what's broken"
  - "failures"
  - "faults"
  - "broken"
  - "/alarms"
  - "active alerts"
  - "health check"
metadata: {"clawdbot":{"emoji":"🏢","requires":{"bins":["python3","curl"]}}}
---

# BMS Intelligence Query

Query the SENTRY Building Management System for real-time equipment data.

## IMPORTANT: Command Routing

- `/info_{code}` → Use THIS skill (data lookup below)
- `/inspect_{code}` → Do NOT use this skill. Follow /inspect instructions in TOOLS.md (create work order immediately, no data lookup)
- `/WO_{code}` → Do NOT use this skill. Follow /WO instructions in TOOLS.md (create maintenance work order)
- **Any building/equipment query** ("Demo Building overview", "DEMO-001 status", "chiller alerts") → Use THIS skill. NEVER use web search.

## How to Use This Skill

When user asks about any building, site, equipment, alerts, or BMS data:
1. Run the appropriate ops_query.py command
2. Present the output — do NOT web search
3. If unsure which command to use, default to `summary` or `equipment`

## /info_ Commands — VERBATIM PASSTHROUGH

When user sends `/info_{code}` (e.g., `/info_DEMO-001_FCU_005`):

1. Extract the equipment code from the message (everything after `/info_`)
2. Run this command — pass the code AS-IS (the tool handles underscore→dash conversion):
```bash
python3 ./tools/ops_query.py info DEMO-001_FCU_005
```
3. **Send the tool output EXACTLY as printed. Do NOT:**
   - Reformat, rephrase, summarise, or restructure the output
   - Add "Likely causes" or diagnostic interpretations
   - Add cost estimates the tool didn't return
   - Query additional endpoints to "enrich" the response
   - Invent sensor readings, temperatures, or pressures
   - Add "Recommended checks" or action items not from the tool
   - Add greetings, preambles, or sign-offs around the output
   - Fetch the inspection checklist separately (it's already included)
   - Add action buttons separately (they're already included)

**The tool output is complete and ready to send. Just output it verbatim.**

## Other Commands

```bash
python3 ./tools/ops_query.py summary
python3 ./tools/ops_query.py equipment
python3 ./tools/ops_query.py details <asset_id>
python3 ./tools/ops_query.py search <site_name>
python3 ./tools/ops_query.py attention
python3 ./tools/ops_query.py temps [site_code]
python3 ./tools/ops_query.py work-orders
python3 ./tools/ops_query.py alarms
```

## Natural Language Patterns — How to Route

| User says | Command to run |
|-----------|----------------|
| `/alarms` or `active alerts` or `faults` or `what's broken` or `failures` | `ops_query.py alarms` — then filter severity if user asked `critical` or `warning` |
| `endpoints` or `what endpoints are available` | Run `ops_query.py summary` and `ops_query.py equipment` — present the combined list |
| `/health` | Say: "Health monitoring runs automatically every 2 hours during occupied hours. I can't trigger it manually — but you can ask me about active alerts, equipment needing attention, or current system status instead." |
| `health check` (non-command) | Route to `attention` — equipment needing attention is the closest interactive health indicator |
| `broken` or `faults` | `ops_query.py alarms` filtered to critical/high |

## CRITICAL RULES

- ALWAYS run the tool command. NEVER guess or fabricate BMS data.
- For /info_ commands: send the tool output VERBATIM. No additions, no reformatting.
- If the tool returns an error, report that error as-is.
- Do NOT add "Likely causes", cost estimates, or diagnostic conclusions.
- Supabase is the single source of truth. Present data clearly, do not interpret it.
