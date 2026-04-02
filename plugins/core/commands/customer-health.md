---
description: Check customer health scores and risk factors
argument-hint: [optional: company name or filter]
allowed-tools: ["mcp__actioner__search_entities", "mcp__actioner__query_data", "mcp__actioner__get_company", "mcp__actioner__get_contact"]
---

Analyze customer health based on the user's request: $ARGUMENTS

> **MANDATORY — Read view schemas before generating SQL:** Before writing ANY SQL query, you MUST first read **`skills/actioner-mcp-server/references/view-schemas.md`** to verify every table name, column name, and column type you intend to use. Never generate SQL from memory or assumption — always consult the schema file first.

If no specific company is given, provide a health overview:

1. **Health distribution** — Use `query_data` to show count of companies by `overall_status` (`healthy`, `needs_attention`, `at_risk`) from the `company_health` view.

2. **At-risk accounts** — Use `search_entities` to list companies with `overall_status = 'at_risk'`. Join `company_health` with `companies` on `company_id`. Include company name (linked to `get_company`), overall status (badge), headline, and the last updated date. Sort by updated_at descending.

3. **Needs attention** — Same as above but for `overall_status = 'needs_attention'`.

If a specific company is mentioned:

1. Use `get_company` to get the full company details.
2. Use `search_entities` against `company_health` joined with `health_assessment_dimensions` to show all dimension scores for that company.
3. Use `search_entities` against `health_risks` to list identified risks with severity and mitigation.
4. Use `search_entities` against `health_stakeholders` to show key stakeholders with their sentiment and engagement.

Present findings with clear risk indicators. Highlight the most critical issues and any recommended mitigations.
