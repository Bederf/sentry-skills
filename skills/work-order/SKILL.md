---
name: work-order
description: Create maintenance work order for equipment. Handles BOTH /WO_ commands AND natural language requests like "create a workorder for the AHU". Bypasses model — runs tool directly.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"📋","requires":{"bins":["python3"]}}}
---

# Work Order — Direct Passthrough

Create a maintenance work order via `/WO_` Telegram command OR natural language request.

## Natural Language — "create a workorder for [equipment]"

When user sends a free-text request like "create a workorder for the AHU" or "raise a WO for FCU 341":

1. Extract the equipment type/name from the message (e.g. "AHU", "FCU", "chiller")
2. Use ops_query.py to get the equipment list and find matching codes:
   ```bash
   python3 ./tools/ops_query.py search Demo Building
   ```
   (search by site name, then filter by type from results)
3. If no site search results, try direct Supabase equipment lookup by type:
   ```bash
   python3 -c "
   import urllib.request, json
   url = 'http://127.0.0.1:55321/rest/v1/equipment?type=eq.ahu&building_id=eq.site-002&select=code,name&limit=5'
   req = urllib.request.Request(url)
   req.add_header('apikey', '$(cat /path/to/consumer/.env | grep BACKEND_SERVICE_KEY | cut -d= -f2)')
   req.add_header('Authorization', 'Bearer $(cat /path/to/consumer/.env | grep BACKEND_SERVICE_KEY | cut -d= -f2)')
   with urllib.request.urlopen(req, timeout=10) as r:
       print(json.loads(r.read()))
   "
   ```
   Or use rapidfuzz for fuzzy matching the equipment type against the full equipment list.
4. Run ops_wo.py with the resolved code and the Telegram user ID
5. Send the tool output VERBATIM

**Resolution rules:**
- "AHU" → DEMO-001-AHU-L2-001 (Level 2 AHU at Demo Building)
- "FCU 341" → DEMO-001-FCU-341 (FCU at zone 341)
- "the chiller" → DEMO-001-CHILLER-B1-001
- "generator in basement" → DEMO-001-GEN-B1-001 or DEMO-001-GEN-B1-002

**If equipment is not found:** Ask the user to confirm the equipment code using `/info_{code}` format.

## /WO_ Commands — VERBATIM PASSTHROUGH

When user sends `/WO_{code}` (e.g., `/WO_DEMO-001_FCU_341`):

1. Extract the equipment code from the message (everything after `/WO_`)
2. Extract the sender's Telegram user ID from the message context
3. Run this command:
```bash
python3 ./tools/ops_wo.py DEMO-001_FCU_341 TECHNICIAN_USER_ID
```
4. **Send the tool output EXACTLY as printed. Do NOT:**
   - Ask the FM for title/description/priority — the tool auto-generates from alerts
   - Add diagnostic interpretations or likely causes
   - Add greetings, preambles, or sign-offs

**The tool output is complete with WO details and buttons. Just output it verbatim.**

## CRITICAL RULES

- ALWAYS run the tool command. NEVER guess or fabricate WO results.
- The tool auto-generates WO details from the equipment's active alerts.
- Send the tool output VERBATIM. No additions, no reformatting.
- If the tool returns an error, report that error as-is.
