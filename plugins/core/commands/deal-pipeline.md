---
description: View and analyze the deal pipeline
argument-hint: [optional: stage, rep, or time filter]
allowed-tools: ["mcp__actioner__search_entities", "mcp__actioner__query_data", "mcp__actioner__get_deal", "mcp__actioner__get_company"]
---

Show the deal pipeline based on the user's request: $ARGUMENTS

> **MANDATORY — Read view schemas before generating SQL:** Before writing ANY SQL query, you MUST first read **`skills/actioner-mcp-server/references/view-schemas.md`** to verify every table name, column name, and column type you intend to use. Never generate SQL from memory or assumption — always consult the schema file first.

If no specific filter is given, provide a pipeline overview:

1. **Pipeline summary** — Use `query_data` to show deal count and total amount grouped by `current_stage`, ordered by pipeline progression.

2. **Top deals** — Use `search_entities` to list the top deals by amount. Include deal name (linked to `get_deal`), company name (linked to `get_company`), amount, stage, expected close date, and health score. Sort by amount descending.

3. **Closing soon** — Use `search_entities` to find deals with `expected_close_date` in the next 30 days.

If the user specifies filters, adapt accordingly:

- **By stage:** Filter on `current_stage`
- **By time period:** Filter on `expected_close_date` ranges
- **By rep/owner:** Join with `contacts` on `owner_contact_id` and filter by name
- **By health:** Filter on `health_score` or `health_status`
- **By forecast:** Filter on `forecast_category`, `forecast_fiscal_quarter`, or `forecast_probability`

Always follow the column metadata rules from the actioner-mcp-server skill. For deal entities, always include: deal name (linked), company name (linked), amount (currency, sortable), stage (badge, sortable), close date (date, sortable).
