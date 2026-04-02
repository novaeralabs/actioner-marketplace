---
description: Get a daily briefing of customer activity
allowed-tools: ["mcp__actioner__get_daily_briefing", "mcp__actioner__search_entities", "mcp__actioner__query_data", "mcp__actioner__get_deal", "mcp__actioner__get_company", "mcp__actioner__get_ticket"]
---

Generate a daily customer activity briefing. Use the `get_daily_briefing` tool first to get the overview.

> **MANDATORY — Read view schemas before generating SQL:** Before writing ANY SQL query, you MUST first read **`skills/actioner-mcp-server/references/view-schemas.md`** to verify every table name, column name, and column type you intend to use. Never generate SQL from memory or assumption — always consult the schema file first.

Then enrich it by querying for:

1. **Urgent tickets** — Use `search_entities` to find open tickets with priority `HIGH` or `URGENT` created or updated in the last 24 hours. Include ticket subject, company, priority, status, and sentiment.

2. **Deals at risk** — Use `search_entities` to find deals where `health_status` indicates concern or `health_score < 50`. Include deal name, company, amount, stage, and health score.

3. **Upcoming closes** — Use `search_entities` to find deals with `expected_close_date` in the next 7 days. Include deal name, company, amount, and stage.

4. **Customer health alerts** — Use `query_data` to summarize companies with `overall_status = 'at_risk'` or `'needs_attention'` from `company_health`.

Present the briefing in a clear, scannable format with the most urgent items first. Highlight anything that needs immediate attention.
