# Actioner MCP Server

Customer intelligence powered by [Actioner](https://actioner.com). Query deals, contacts, tickets, health scores, and more — all through a unified data model connected to Claude via the Model Context Protocol.

## Description

Actioner is a customer intelligence platform that aggregates data from CRM tools (HubSpot, Salesforce), support systems (Zendesk, Freshdesk), email (Gmail, Outlook), meeting transcription services (Gong, Fathom, TLDV, Fireflies), and product usage sources into a single, queryable data model.

This MCP server gives Claude direct access to your organization's customer data so you can ask questions in natural language, browse interactive result tables, create records, compose emails, and manage calendar events — without switching between tools.

## Features

- **Natural-language data queries** — Ask questions like "which deals are closing this month?" and get structured, interactive tables with sorting, pagination, and drill-down links.
- **Entity detail views** — Get rich detail cards for deals, companies, contacts, tickets, tasks, and conversations with related data, stakeholders, and activity timelines.
- **Daily briefings & meeting prep** — One-command summaries of your day: upcoming meetings, action items, urgent tickets, and at-risk deals.
- **Record creation** — Add deals, tasks, contacts, companies, entitlements, and notes through interactive forms with prefilled context from your conversation.
- **Record updates** — Update existing deals, companies, contacts, tasks, entitlements, and notes through interactive forms.
- **Email composition** — Draft and send emails with recipient autocomplete, reply threading, and CC support — all reviewed before sending.
- **Calendar management** — Create, update, and cancel meetings with attendee management and description editing.
- **Document access** — Browse and read internal knowledge base documents stored in your Actioner workspace.
- **SQL analytics** — Run read-only SQL queries against pre-built, org-scoped views for custom aggregations, trend analysis, and distributions.

## Setup

1. Visit the [Anthropic MCP Directory](https://claude.com/connectors).
2. Find and connect to **Actioner**.
3. Complete OAuth authentication — sign in with your Actioner account when prompted.
4. Select your organization if you belong to multiple workspaces.

No API keys or environment variables are needed. The server uses OAuth 2.0 with PKCE for secure, token-based authentication.

## Authentication

This server requires OAuth 2.0 authentication.

- **Flow**: Authorization Code with PKCE (RFC 7636)
- **Transport**: Streamable HTTP (`POST https://app.actioner.com/mcp`)
- **Token format**: JWT with organization-scoped claims
- **Scopes**: `openid`, `profile`, `email`, `offline_access`

You need a valid Actioner account with at least one configured organization. After signing in, your active organization determines which data is accessible.

## Tools

### Data Querying

| Tool                | Type      | Description                                                                                   |
| ------------------- | --------- | --------------------------------------------------------------------------------------------- |
| **Query Data**      | Read-only | Execute read-only SQL for aggregated analytics — counts, averages, distributions, and trends  |
| **Search Entities** | Read-only | Display entity record lists in interactive tables with typed columns, sorting, and pagination |

### Entity Details

| Tool                 | Type      | Description                                                                                                   |
| -------------------- | --------- | ------------------------------------------------------------------------------------------------------------- |
| **Get Deal**         | Read-only | Full deal details including amount, stage, stakeholders, risks, qualifications, pipeline, and recent activity |
| **Get Company**      | Read-only | Company profile with contacts, deals, tickets, health assessment, and interaction timeline                    |
| **Get Contact**      | Read-only | Contact details with company association, communication history, and related deals                            |
| **Get Ticket**       | Read-only | Support ticket with full conversation thread, sentiment, priority, and resolution status                      |
| **Get Task**         | Read-only | Task details with assignees, status, priority, and related entities                                           |
| **Get Conversation** | Read-only | Conversation details with messages, participants, and metadata                                                |

### Insights

| Tool               | Type      | Description                                                                                          |
| ------------------ | --------- | ---------------------------------------------------------------------------------------------------- |
| **Daily Briefing** | Read-only | Today's meetings, suggested next actions, urgent items, and schedule overview                        |
| **Meeting Prep**   | Read-only | Pre-meeting context including attendee profiles, recent interactions, open deals, and talking points |

### Documents

| Tool                     | Type      | Description                                                             |
| ------------------------ | --------- | ----------------------------------------------------------------------- |
| **List Documents**       | Read-only | List all documents in the workspace with metadata                       |
| **Get Document Content** | Read-only | Retrieve markdown content of a document with pagination for large files |

### Record Creation (destructive)

| Tool                | Type  | Description                                                                                      |
| ------------------- | ----- | ------------------------------------------------------------------------------------------------ |
| **Add Deal**        | Write | Open an interactive form to add a new deal with company and pipeline selection                   |
| **Add Task**        | Write | Open an interactive form to add a task with assignee, priority, and due date                     |
| **Add Person**      | Write | Open an interactive form to add a new contact with company association                           |
| **Add Company**     | Write | Open an interactive form to add a new company/account with name, industry, and website           |
| **Add Entitlement** | Write | Open an interactive form to add a new entitlement with product, billing cycle, and usage details |
| **Add Note**        | Write | Open an interactive form to add a note linked to an entity                                       |

### Record Updates (destructive)

| Tool                   | Type  | Description                                                                         |
| ---------------------- | ----- | ----------------------------------------------------------------------------------- |
| **Update Deal**        | Write | Open a form to update an existing deal's stage, amount, close date, or other fields |
| **Update Company**     | Write | Open a form to update an existing company's details                                 |
| **Update Person**      | Write | Open a form to update an existing contact's details                                 |
| **Update Task**        | Write | Open a form to update a task's status, priority, assignee, or due date              |
| **Update Entitlement** | Write | Open a form to update an existing entitlement                                       |
| **Update Note**        | Write | Open a form to update an existing note                                              |

### Communication (destructive)

| Tool               | Type  | Description                                                                                |
| ------------------ | ----- | ------------------------------------------------------------------------------------------ |
| **Compose Email**  | Write | Open an email composition form with recipient autocomplete, subject, and rich body editing |
| **Create Meeting** | Write | Open a form to create a calendar event with attendees, time, and description               |
| **Update Meeting** | Write | Open a form to reschedule or modify an existing calendar event                             |

## Examples

### Example 1: Daily briefing and deal follow-up

**User prompt:** "Give me my daily briefing"

**What happens:**

1. The server calls the **Daily Briefing** tool, which fetches today's calendar events and pending action items for the logged-in user.
2. Claude presents a structured overview: upcoming meetings with summaries, suggested next actions ordered by priority, and any items requiring immediate attention.
3. The user can click on any meeting to get full details, or click an action item to drill into the related deal or ticket.

**Follow-up prompt:** "Tell me more about the Acme Corp deal"

**What happens:**

1. The server calls **Get Deal**, which runs parallel queries to fetch the deal record, stakeholders, risks, qualification criteria, pipeline stages, action items, and recent activity.
2. Claude presents a rich detail card with the deal amount, probability, stage progression, linked company, key stakeholders with sentiment scores, identified risks, and a timeline of recent interactions.

---

### Example 2: Searching and analyzing pipeline

**User prompt:** "Show me all deals closing this quarter over $50K"

**What happens:**

1. The server calls **Search Entities** with a SQL query against the `deals` view, filtering by `expected_close_date` within the current quarter and `amount > 50000`.
2. Claude presents an interactive table with columns: deal name (clickable link to deal details), company name (clickable link to company details), amount (formatted currency, sortable), stage (badge, sortable), and expected close date (sortable).
3. The user can sort by any column, paginate through results, and click any deal or company name to drill into details.

**Follow-up prompt:** "What's the total pipeline value by stage?"

**What happens:**

1. The server calls **Query Data** with an aggregation query grouping deals by `current_stage`, summing amounts.
2. Claude presents the stage-by-stage breakdown with deal counts and total values, making it easy to identify where the pipeline is concentrated.

---

### Example 3: Composing an email to a contact

**User prompt:** "Draft an email to Sarah at Acme Corp about the upcoming renewal"

**What happens:**

1. The server calls **Compose Email**, pre-filling the recipient by searching contacts matching "Sarah" at "Acme Corp". The email subject and body are drafted based on the conversation context.
2. An interactive email composition form appears with:
   - Pre-filled recipient (with autocomplete for adding more)
   - Draft subject line
   - Rich-text body editor with the AI-generated draft
   - CC field and reply-threading support
3. The user reviews the draft, makes any edits, and clicks **Send**. The server calls the submit tool to send the email through the connected email provider.

---

### Example 4: Adding a task from conversation context

**User prompt:** "Add a follow-up task to send the proposal to Acme Corp by Friday"

**What happens:**

1. The server calls **Add Task**, pre-filling the title ("Send proposal to Acme Corp"), linking to the relevant company, and setting the due date to the upcoming Friday.
2. An interactive task creation form appears with fields for title, description, assignee (with contact search), priority, status, due date, and associated company.
3. The user reviews the pre-filled form, selects an assignee from the contact dropdown, and submits. The task is created in Actioner and linked to the appropriate deal and company.

---

### Example 5: Checking customer health and preparing for a meeting

**User prompt:** "What's the health status of our top 5 accounts?"

**What happens:**

1. The server calls **Search Entities** joining `company_health` with `companies`, ordered by a relevant metric, limited to 5 results.
2. Claude presents a table with company name (clickable), overall health status (healthy / needs attention / at risk), health score, headline summary, and last assessment date.
3. The user clicks on an at-risk account to see full health dimensions, risk factors with severity, and stakeholder sentiment.

**Follow-up prompt:** "Prep me for my meeting with their team tomorrow"

**What happens:**

1. The server calls **Meeting Prep** for the selected company's upcoming meeting, fetching attendee profiles, recent interaction history, open deals, pending tickets, and AI-generated talking points.
2. Claude presents a structured prep document: attendee bios with roles and recent activity, key discussion topics, open items to address, and suggested agenda points.

## Available Data

The Actioner data model includes pre-built, organization-scoped views for:

| Category         | Views                                                                                                      |
| ---------------- | ---------------------------------------------------------------------------------------------------------- |
| **CRM**          | Companies, Contacts, Deals, Deal Stakeholders, Deal Risks, Deal Qualifications, Pipelines, Pipeline Stages |
| **Support**      | Tickets, Conversations, Conversation Messages, Participants                                                |
| **Health**       | Company Health, Health Assessment Dimensions, Health Risks, Health Stakeholders                            |
| **Productivity** | Tasks, Action Items, Work Items, Notes, Signals                                                            |
| **Product**      | Products, Product Usages, Entitlements                                                                     |
| **Knowledge**    | Documents                                                                                                  |

All views are automatically scoped to the authenticated user's organization. SQL queries run in a read-only context against PostgreSQL 17.

## Commands

| Command             | Description                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------- |
| `/briefing`         | Get a daily briefing — urgent tickets, at-risk deals, upcoming closes, and health alerts |
| `/search-customers` | Search for customers, contacts, or companies by name, industry, or criteria              |
| `/deal-pipeline`    | View and analyze the deal pipeline — by stage, rep, time period, or health               |
| `/customer-health`  | Check customer health scores, risk factors, and stakeholder sentiment                    |

## Privacy Policy

See our privacy policy: [https://actioner.com/privacy-policy](https://actioner.com/privacy-policy)

## Support

- **Email:** hello@actioner.com
- **Website:** [https://actioner.com](https://actioner.com)
- **Documentation:** [https://docs.actioner.com](https://docs.actioner.com)
