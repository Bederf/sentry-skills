---
name: fault-codes
description: Look up BMS fault codes by manufacturer and code. 303 codes across 9 manufacturers.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"🔧","requires":{"bins":["python3"]}}}
---

# Fault Code Lookup

Search manufacturer fault code databases for BMS equipment.

## Commands

**Search a specific fault code:**
```bash
python3 ./tools/ops_fault_codes.py search <manufacturer> <code>
```

**List all codes for a manufacturer:**
```bash
python3 ./tools/ops_fault_codes.py list <manufacturer>
```

**Search codes by keyword:**
```bash
python3 ./tools/ops_fault_codes.py find <keyword>
```

## Supported Manufacturers

Carrier, York, Daikin, Trane, Mitsubishi Electric, Samsung, Bosch, LG, Lennox

## Output Handling

Present the tool output clearly for Telegram readability:
1. Fault code and meaning
2. Manufacturer and equipment type
3. Probable causes
4. Recommended actions
5. Safety warnings (if any)

Do NOT send raw output verbatim — format for readability.
Do NOT fabricate or embellish beyond what the tool returns.

## CRITICAL: Do Not Hallucinate

- ALWAYS run the tool command. NEVER guess fault codes, causes, or remedies.
- If the tool returns no results, say "Fault code not found in database."
- If the tool returns an error, report that error as-is.
- The tool queries the local fault code database — it is the single source of truth.
