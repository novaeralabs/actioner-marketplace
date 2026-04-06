---
description: Search for customers, contacts, or companies
argument-hint: [search query]
allowed-tools:
  [
    'mcp__actioner__search_entities',
    'mcp__actioner__get_company',
    'mcp__actioner__get_contact',
    'mcp__actioner__query_data',
  ]
---

Search for customers based on the user's query: $ARGUMENTS

> **MANDATORY — Read view schemas before generating SQL:** Before writing ANY SQL query, you MUST first read **`skills/actioner-mcp-server/references/view-schemas.md`** to verify every table name, column name, and column type you intend to use. Never generate SQL from memory or assumption — always consult the schema file first.

Determine the best search strategy based on what the user is looking for:

**If searching for a company:** Use `search_entities` against the `companies` view. Match on `name` using `ILIKE '%term%'`. Include company name (linked), industries, customer state, employee count, and country. Also join with `deals` to show deal count and total pipeline value if relevant.

**If searching for a contact/person:** Use `search_entities` against the `contacts` view. Match on `name` using `ILIKE '%term%'` or on `email` using array containment. Include contact name (linked), title, company name (linked), email, and type.

**If the query is about a category** (e.g., "healthcare companies", "enterprise customers"): Use `search_entities` with appropriate filters on `industries`, `customer_type`, `employees`, `tags`, or `country`.

**If the query is analytical** (e.g., "how many customers in Europe", "top customers by deal value"): Use `query_data` with the appropriate aggregation.

Always follow the column metadata rules from the actioner-mcp-server skill when constructing `search_entities` calls — include proper ID fields, link actions, sortable flags, and keep visible columns to 5–8.
