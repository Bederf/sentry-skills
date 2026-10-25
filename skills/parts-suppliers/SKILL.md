---
name: parts-suppliers
description: Search for BMS equipment parts suppliers in South Africa with pricing and stock availability.
disable-model-invocation: true
metadata: {"clawdbot":{"emoji":"🔩","requires":{"bins":["python3"]}}}
---

# Parts Supplier Search

Search for BMS equipment parts suppliers across South Africa.

## Commands

**Search for parts suppliers:**
```bash
python3 ./tools/ops_parts_suppliers.py search <part> [location]
```

**Check stock availability:**
```bash
python3 ./tools/ops_parts_suppliers.py stock <part>
```

**Compare prices across suppliers:**
```bash
python3 ./tools/ops_parts_suppliers.py compare <part>
```

## Supported Locations

Johannesburg, Cape Town, Durban, Pretoria, Port Elizabeth, Bloemfontein

## Output Handling

Present the tool output clearly for Telegram readability:
1. Part identification
2. Supplier options (ranked by relevance)
3. Stock availability and pricing
4. Contact details
5. Lead times

Do NOT send raw output verbatim — format for readability.
Do NOT fabricate or embellish beyond what the tool returns.

## CRITICAL: Do Not Hallucinate

- ALWAYS run the tool command. NEVER guess supplier names, prices, or contact details.
- If no suppliers found, say "No suppliers found in database."
- If the tool returns an error, report that error as-is.
- The tool queries the local parts supplier database — it is the single source of truth.
