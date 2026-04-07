---
name: actioner-mcp-server
description: >
  This document is the authoritative reference running Actioner MCP tools and
  for constructing SQL queries used by these tools. Any tool that accepts a SQL
  parameter — including `query_data`, `search_entities`, and any future tools —
  must follow the data model, view schemas, relationships, and query guidelines
  described below. All view names, column names, types, and join paths documented
  here apply to every SQL-accepting tool. All generated SQL must be PostgreSQL 17
  compliant.
version: 0.1.0
---

# Actioner MCP Server

## What is Actioner?

Actioner is a customer intelligence platform that aggregates data from support systems (Zendesk, Freshdesk), CRM tools (HubSpot), email (Gmail, Outlook), meeting transcription services (Gong), and product usage sources. It provides a unified view of customer relationships including support tickets, conversations, deals, entitlements, product usage, AI-generated work items and action items, and customer health assessments.

**This document is the authoritative reference running Actioner MCP tools and for constructing SQL queries used by these tools.** Any tool that accepts a SQL parameter — including `query_data`, `search_entities`, and any future tools — must follow the data model, view schemas, relationships, and query guidelines described below. All view names, column names, types, and join paths documented here apply to every SQL-accepting tool. All generated SQL must be **PostgreSQL 17** compliant.

## SQL Preflight Checklist

Before **ANY** call to `search_entities` or `query_data`, you MUST complete every step below in order:

1. **Load the schema** — Read **`references/view-schemas.md`**. Do NOT skip this step.
2. **Identify views** — Determine which views the query needs (e.g. `deals`, `companies`).
3. **Verify every column** — For each column you plan to SELECT, filter, join, or sort on, confirm it exists in the view schema and note its type.
4. **Check array columns** — Any `text[]` column requires `ANY()`, `array_length()`, `unnest()`, or `array_to_string()` — never use `=` or `LIKE` on arrays.
5. **Generate SQL** — Only now write the SQL query using verified names and types.
6. **If a column is not in the schema, do not use it** — there is no fallback.

## Column Name Pitfalls

These are the most frequent wrong guesses. The left column **does not exist**:

| Wrong (does NOT exist) | Correct column name    | View      | Notes                                  |
| ---------------------- | ---------------------- | --------- | -------------------------------------- |
| `website`              | `website_url`          | companies |                                        |
| `industry`             | `industries`           | companies | `text[]` — use `'X' = ANY(industries)` |
| `size`                 | `employees`            | companies |                                        |
| `employee_count`       | `employees`            | companies |                                        |
| `domain`               | `domains`              | companies | `text[]` — use `'X' = ANY(domains)`    |
| `company_name`         | `name`                 | companies | Use alias: `c.name AS company_name`    |
| `contact_name`         | `name`                 | contacts  | Use alias: `ct.name AS contact_name`   |
| `deal_name`            | `name`                 | deals     | Use alias: `d.name AS deal_name`       |
| `ticket_subject`       | `subject`              | tickets   |                                        |
| `close_date`           | `expected_close_date`  | deals     |                                        |
| `probability`          | `forecast_probability` | deals     |                                        |
| `is_deleted`           | _(pre-filtered)_       | all views | Views already exclude deleted records  |
| `deleted_at`           | _(pre-filtered)_       | all views | Views already exclude deleted records  |

## Choosing Between `search_entities` and `query_data`

Use **`search_entities`** when:

- The result is a **list of records** the user would browse and click into
- Each row represents a specific entity (deal, person, company, task, interaction)
- The user would benefit from clicking through to entity details
- The table supports **server-side sorting and pagination**
- Examples: "show me deals closing this month", "which companies are in healthcare", "find contacts at Acme"

Use **`query_data`** when:

- The result is **aggregated or analytical** data showing patterns/metrics
- Rows represent groups, counts, averages, distributions — not individual records
- The user wants to understand a trend, comparison, or summary
- Examples: "distribution of deals per close month", "average deal size by rep", "win rate by stage"

**When in doubt:** if the user would want to click on individual rows to see more detail, use `search_entities`. If they want to see a chart or summary number, use `query_data`.

## `search_entities` Tool Reference

`search_entities` displays a **list of entity records** in an interactive table with typed columns, formatting, clickable links, sorting, and pagination. Use it when each row represents a specific entity (deal, company, contact, task, interaction, etc.) the user would browse and click into. For aggregated/analytical data (counts, averages, distributions, trends), use `query_data` instead.

### Parameters

| Parameter     | Required | Description                                                             |
| ------------- | -------- | ----------------------------------------------------------------------- |
| `label`       | Yes      | Table title displayed above the table                                   |
| `sql`         | Yes      | Base SQL query (see SQL rules below)                                    |
| `columns`     | Yes      | Column definitions with display types, sortable flags, and link actions |
| `defaultSort` | No       | Initial sort: `{ column: "amount", direction: "DESC" }`                 |
| `pageSize`    | No       | Rows per page (10–100, default 50)                                      |

### SQL Rules

> **CRITICAL:** The `sql` parameter must only contain **SELECT, FROM, JOIN, WHERE, GROUP BY, HAVING** clauses. Do **NOT** add `ORDER BY` or `LIMIT` — sorting and pagination are managed automatically by the UI via the `defaultSort` and `pageSize` parameters.

### Column Rules

1. **Always include the entity's ID** as a hidden column (`visible: false`).
2. **Always include ID columns for any linked entities** — these are passed as `linkAction` args so the detail tool knows which record to load.
3. **Use `dataType: 'link'`** with a `linkAction` for entity name columns so users can click to view details.
4. **Set `sortable: true`** on columns where sorting is meaningful (names, dates, amounts, statuses).
5. **Do NOT set `sortable: true`** on ID columns, array columns, or long text fields.
6. **Keep total visible columns to 4–6** for readability.

### Required Columns by Entity Type

**DEAL:** deal name (`link` → `get_deal`, sortable), company name (`link` → `get_company`, sortable), amount (decimal, sortable), stage (enum, sortable), close date (date, sortable). Add others as relevant.

**COMPANY:** company name (`link` → `get_company`, sortable), industries (text[], use `array_to_string(industries, ', ')` to display), deal count or total pipeline (number, sortable), primary contact (`link` → `get_contact`).

**CONTACT:** name (`link` → `get_contact`, sortable), title (text), company name (`link` → `get_company`, sortable), email (text), last interaction date (date, sortable).

**TASK:** title (`link` → `get_task`, sortable), assigned to (`link` → `get_contact`), due date (date, sortable), priority (enum, sortable), status (enum, sortable), related deal (`link` → `get_deal`).

**INTERACTION:** date (date, sortable), type (enum), subject (`link` → `get_conversation`, sortable), person (`link` → `get_contact`), company (`link` → `get_company`).

## `query_data` Tool Reference

`query_data` executes a **read-only SQL query** for aggregated or analytical data — counts, averages, distributions, trends, and summaries. Use it when the result represents grouped metrics, not a list of individual records the user would click into. For lists of browsable entity records (deals, companies, contacts, tasks, interactions), use `search_entities` instead.

### SQL Rules

- The query runs in the **`agent_views`** schema which contains pre-built views for tickets, companies, contacts, conversations, deals, entitlements, health, products, work items, action items, conversation messages, documents, signals, tasks, notes, and more.
- All views are **automatically scoped** to the current organization.
- Generated SQL must be **PostgreSQL 17** compliant.
- Results are **capped at 100 rows** — use `LIMIT` and `ORDER BY` to control which rows are returned.

## Entity Detail Tools

| Tool               | Input            | Shows                                                                            |
| ------------------ | ---------------- | -------------------------------------------------------------------------------- |
| `get_deal`         | `dealId`         | Deal name, amount, probability, stage, close date, linked company, stakeholders  |
| `get_company`      | `companyId`      | Name, industries, employees, address; tabs: Deals, Contacts, Entitlements, Notes |
| `get_contact`      | `contactId`      | Name, email, title, company; tabs: Deals, Interactions, Tasks, Notes             |
| `get_task`         | `taskId`         | Title, status, priority, due date, assignees, related company, description       |
| `get_conversation` | `conversationId` | Title, type, date, participants, related deal, summary                           |
| `get_ticket`       | `ticketId`       | Subject, priority, status, due date; tabs: Participants, History, Tasks          |

## Data Model Overview

The data model is organized around **companies** (customer accounts) and their relationships:

- **Companies** – Customer accounts with metadata, domains, and tags.
- **Contacts** – People (end users or internal users) associated with companies.
- **Tickets** – Support tickets with status, priority, sentiment analysis, and AI-enriched signals.
- **Conversations** – Emails, meetings, and chats with participants and AI analysis.
- **Deals** – Sales opportunities with amount, stage, health, forecast, stakeholders, risks, and qualifications.
- **Entitlements** – Customer contracts/licenses with revenue, billing, and renewal information.
- **Company Health** – AI-assessed health status with stakeholders, risks, and assessment dimensions.
- **Products & Features** – Product catalog and feature definitions.
- **Product Usages** – Usage events per company/user/product/feature.
- **Work Items** – AI-generated actionable work items linked to contacts, conversations, tickets, deals, or health records.
- **Action Items** – Suggested actions for work items with execution tracking.
- **Documents** – Living knowledge base containing organization-specific definitions.
- **Signals** – Named emotions, request types, and account-level categories detected by AI.
- **Tasks** – Internal tasks assigned to team members, linked to companies.
- **Notes** – Free-form notes attached to companies or contacts.

## Query Guidelines

1. **All views are pre-filtered** to the current organization. No need to add organization filters.
2. **Use JOINs** to connect related entities.
3. **Array columns** (text[]) — use `'value' = ANY(column)`, `array_length(column, 1)`, or `unnest(column)`.
4. **Results are capped at 500 rows.** For `query_data`, use `LIMIT` and `ORDER BY`. For `search_entities`, do NOT add `ORDER BY` or `LIMIT`.
5. **Use `all_conversations`** instead of `conversations` if you need to include conversations without listed external contacts.
6. **Deleted records are already excluded.** Do NOT add `WHERE is_deleted = false` — the column does not exist on any view.

## Reference Files

- **`references/view-schemas.md`** — Complete column-level schemas for all available views (MUST be loaded before writing SQL)
- **`references/query-examples.md`** — Common JOIN patterns, date filtering, and aggregation examples
