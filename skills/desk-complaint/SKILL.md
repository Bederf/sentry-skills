---
name: desk-complaint
description: Handle desk comfort complaints (too hot/cold/stuffy/drafty). Run diagnosis and create WO when dispatch needed.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"❄️","requires":{"bins":["python3"]}}}
---

# Desk Comfort Complaint Handler

Handle desk-level HVAC complaints. When dispatch is needed, creates a work order AND notifies the technician.

## Trigger Phrases

- "desk [number] is too hot/cold/stuffy/drafty"
- "too cold at desk [number]"
- "freezing at desk [number]"
- "air is stuffy at desk [number]"
- "desk [number] complaint"

## Step 1: Diagnose

Always run the diagnosis tool FIRST. NEVER guess.

```bash
python3 ./tools/ops_desk_diagnosis.py <desk_id> <complaint_type>
```

Where `<complaint_type>` is one of: `too_hot`, `too_cold`, `stuffy`, `drafty`

**Examples:**
```bash
# "desk DEMO-DESK-203 is too cold"
python3 ./tools/ops_desk_diagnosis.py DEMO-DESK-203 too_cold

# "freezing at desk DEMO-DESK-045"
python3 ./tools/ops_desk_diagnosis.py DEMO-DESK-045 too_cold

# "desk DEMO-DESK-L12 is stuffy"
python3 ./tools/ops_desk_diagnosis.py DEMO-DESK-L12 stuffy
```

## Step 2: Report Diagnosis to FM

Present the diagnosis output clearly:
1. Desk location (floor, zone)
2. Current HVAC readings (temp vs setpoint, FCU status)
3. Root cause
4. Recommended actions
5. **Dispatch status** (YES/NO)

## Step 3: If Dispatch Required → Create Work Order

If the diagnosis shows `dispatch_required: True` (or technician needed):

```bash
python3 ./tools/ops_desk_wo.py <desk_id> <complaint_type> "<diagnosis_summary>"
```

Example:
```bash
python3 ./tools/ops_desk_wo.py DEMO-DESK-203 too_cold "FCU at 18C, setpoint 22C, deviation -4C"
```

This will:
1. Create work order for the equipment
2. Send notification to Synthetic Technician via the configured technician channel.
3. Return WO code and service record

## Complaint Type Mapping

| User says | complaint_type |
|-----------|---------------|
| too hot, warm, boiling | `too_hot` |
| too cold, freezing, chilly | `too_cold` |
| stuffy, stale air, no air | `stuffy` |
| drafty, windy, cold draft | `drafty` |

## Response Format

**Always start with the diagnosis tool output.** Then add:

```
[Diagnosis output from ops_desk_diagnosis.py]

---
❌ Dispatch needed → Work order created: WO-2026-XXXX
📨 Technician Synthetic Technician notified via the configured technician channel
```

If NO dispatch needed:
```
[Diagnosis output]

✅ No dispatch needed — auto-action taken or issue resolvable remotely.
```

## CRITICAL

- ALWAYS run the diagnosis tool first — NEVER fabricate readings
- If dispatch is needed, ALWAYS run the WO tool — the technician MUST be notified
- The WO tool sends to the Sentinel Tech bot (separate from FM bot)
- Report BOTH the diagnosis AND the WO result to the FM
