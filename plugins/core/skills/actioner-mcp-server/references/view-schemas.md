# View Schemas

All views live in the `agent_views` schema. Reference them directly by name (e.g. `SELECT * FROM tickets LIMIT 10`) in any SQL-accepting tool.

---

## tickets

Support tickets from all integrated data sources.

| Column              | Type      | Description                                                         |
| ------------------- | --------- | ------------------------------------------------------------------- |
| id                  | text      | Ticket ID (PK, e.g. `tck_123`)                                      |
| remote_id           | text      | ID in the source system                                             |
| end_user_id         | text      | Contact ID of the requester/submitter                               |
| agent_ids           | text[]    | Contact IDs of assigned agents                                      |
| subject             | text      | Ticket subject line                                                 |
| status              | text      | `OPEN`, `IN_PROGRESS`, `ON_HOLD`, `CLOSED`                          |
| priority            | text      | `LOW`, `NORMAL`, `HIGH`, `URGENT`                                   |
| type                | text      | `INCIDENT`, `PROBLEM`, `QUESTION`, `TASK`                           |
| submission_channel  | text      | Mapped submission channel name                                      |
| first_response_time | bigint    | First response time in milliseconds                                 |
| resolution_time     | bigint    | Resolution time in milliseconds                                     |
| created_at          | timestamp | When the ticket was created                                         |
| updated_at          | timestamp | Last update time                                                    |
| closed_at           | timestamp | When the ticket was closed                                          |
| due_at              | timestamp | Due date                                                            |
| ticket_url          | text      | URL to the ticket in the source system                              |
| tags                | text[]    | Tags attached to the ticket                                         |
| is_reopened         | boolean   | Whether the ticket was reopened                                     |
| summary             | text      | AI-generated summary                                                |
| sentiment_score     | integer   | Sentiment score (0-100)                                             |
| sentiment_enum      | text      | `VERY_NEGATIVE`, `NEGATIVE`, `NEUTRAL`, `POSITIVE`, `VERY_POSITIVE` |
| ticket_stage        | text      | `WAITING_FOR_CUSTOMER`, `WAITING_FOR_SUPPORT`, `TICKET_CLOSED`      |
| emotions            | text[]    | Detected emotion signal names                                       |
| products            | text[]    | Referenced product names                                            |
| product_features    | text[]    | Referenced product feature names                                    |
| account_signals     | text[]    | Detected account signal names                                       |
| source_table        | text      | Data source ID this ticket came from                                |

**Relationships:**
- `end_user_id` → `contacts.id`
- `agent_ids[n]` → `contacts.id`
- Join with `ticket_participants` on `ticket_id` for all participants
- Join with `ticket_histories` on `ticket_id` for history
- Join with `ticket_tasks` on `ticket_id` for linked tasks

---

## companies

Customer accounts from the unified contact mediation layer.

| Column                       | Type      | Description                                                                 |
| ---------------------------- | --------- | --------------------------------------------------------------------------- |
| id                           | text      | Company ID (PK, e.g. `cmp_123`)                                             |
| name                         | text      | Company name                                                                |
| domains                      | text[]    | Company domains                                                             |
| employees                    | integer   | Employee count                                                              |
| created_at                   | timestamp | Creation time                                                               |
| updated_at                   | timestamp | Last update time                                                            |
| linkedin_url                 | text      | LinkedIn profile URL                                                        |
| website_url                  | text      | Company website URL                                                         |
| light_logo                   | text      | Light theme logo URL                                                        |
| dark_logo                    | text      | Dark theme logo URL                                                         |
| light_color                  | text      | Light theme brand color                                                     |
| dark_color                   | text      | Dark theme brand color                                                      |
| is_from_sandbox              | text      | Whether this is sandbox/demo data                                           |
| deal_processing_triggered_at | text      | When deal processing last ran                                               |
| city                         | text      | City                                                                        |
| country                      | text      | Country                                                                     |
| details                      | text      | Company description                                                         |
| details_markdown             | text      | Detailed description in markdown                                            |
| subscription_ids             | text[]    | Linked subscription IDs                                                     |
| account_owner_id             | text      | Account owner user ID                                                       |
| tags                         | text[]    | Tags                                                                        |
| industries                   | text[]    | Industry classifications (use `'value' = ANY(industries)` to filter)        |
| kind                         | text      | Company classification                                                      |
| listed                       | text      | `LISTED`, `LISTED_BY_USER`, `NOT_LISTED`, `NOT_LISTED_BY_USER`              |
| customer_state               | text      | `prospect`, `past_prospect`, `current_customer`, `past_customer`, `unknown` |
| customer_type                | text      | Customer type from custom fields                                            |

**Relationships:**
- Join with `contacts` on `contacts.company_id = companies.id`
- Join with `deals` on `deals.company_id = companies.id`
- Join with `entitlements` on `entitlements.company_id = companies.id`
- Join with `company_health` on `company_health.company_id = companies.id`
- Join with `tasks` on `tasks.company_id = companies.id`
- Join with `notes` on `notes.company_id = companies.id`

---

## contacts

People (end users and internal users) from the unified contact mediation layer.

| Column           | Type   | Description                                                    |
| ---------------- | ------ | -------------------------------------------------------------- |
| id               | text   | Contact ID (PK, e.g. `end_123`)                                |
| name             | text   | Full name                                                      |
| email            | text[] | Email addresses                                                |
| details          | text   | Contact description                                            |
| avatar_url       | text   | Avatar image URL                                               |
| roles            | text[] | Roles assigned                                                 |
| type             | text   | `INTERNAL_USER` or `END_USER`                                  |
| tags             | text[] | Tags                                                           |
| teams            | text[] | Team memberships                                               |
| job              | text   | Job title from custom fields                                   |
| title            | text   | Title                                                          |
| phone            | text   | Phone number                                                   |
| details_markdown | text   | Description in markdown                                        |
| company_id       | text   | Associated company ID                                          |
| company_name     | text   | Associated company name (denormalized)                         |
| linkedin_url     | text   | LinkedIn URL                                                   |
| city             | text   | City                                                           |
| state            | text   | State/Province                                                 |
| country          | text   | Country                                                        |
| is_from_sandbox  | text   | Whether this is sandbox/demo data                              |
| listed           | text   | `LISTED`, `LISTED_BY_USER`, `NOT_LISTED`, `NOT_LISTED_BY_USER` |

**Relationships:**
- `company_id` → `companies.id`
- Referenced by `tickets.end_user_id`, `tickets.agent_ids`, `deal_stakeholders.contact_id`, `health_stakeholders.contact_id`

---

## conversations

Conversations (emails, meetings, chats) that involve at least one listed external contact.

| Column             | Type      | Description                                        |
| ------------------ | --------- | -------------------------------------------------- |
| id                 | text      | Conversation ID (PK, e.g. `cnv_123`)               |
| remote_id          | text      | ID in the source system                            |
| subject            | text      | Subject line                                       |
| created_at         | timestamp | Creation time                                      |
| updated_at         | timestamp | Last update time                                   |
| submitter_email    | text      | Email of the initial sender                        |
| title              | text      | Conversation title                                 |
| type               | text      | `EMAIL`, `MEETING_NOTES`, `CHAT`, `CALENDAR_EVENT` |
| summary            | text      | AI-generated summary                               |
| sentiment_score    | integer   | Sentiment score (0-100)                            |
| sentiment_enum     | text      | Sentiment classification                           |
| emotions           | text[]    | Detected emotion signal names                      |
| products           | text[]    | Referenced product names                           |
| product_features   | text[]    | Referenced product feature names                   |
| account_signals    | text[]    | Detected account signal names                      |
| tags               | text[]    | Tags                                               |
| first_message_date | timestamp | First message date                                 |
| last_message_date  | timestamp | Last message date                                  |
| message_count      | integer   | Total message count                                |
| from_us_count      | integer   | Messages sent by internal users                    |
| content            | jsonb     | Full conversation content                          |
| markdown_content   | text      | Content in markdown                                |
| meeting_urls       | text      | Meeting URLs (for calendar events)                 |
| status             | text      | Conversation status                                |
| end_time           | text      | End time (for calendar events)                     |
| source_table       | text      | Data source ID                                     |

**Relationships:**
- Join with `conversation_messages` on `conversation_messages.conversation_id = conversations.id`
- Join with `participants` on `participants.conversation_id = conversations.id`
- Join with `deal_interactions` on `deal_interactions.conversation_id = conversations.id`
- Join with `interaction_signals` on `interaction_signals.interaction_id = conversations.id`

---

## all_conversations

Same schema as `conversations` but includes ALL conversations regardless of whether they have listed external contacts.

---

## conversation_messages

Individual messages within conversations.

| Column              | Type      | Description                          |
| ------------------- | --------- | ------------------------------------ |
| id                  | text      | Message ID (PK, e.g. `cnm_123`)      |
| remote_id           | text      | ID in the source system              |
| conversation_id     | text      | Parent conversation ID (FK)          |
| subject             | text      | Message subject line (nullable)      |
| text_body           | text      | Plain text body (nullable)           |
| html_body           | text      | HTML body (nullable)                 |
| remote_participants | jsonb     | Participant metadata (JSON)          |
| created_by          | text      | Creator identifier (nullable)        |
| attachments         | jsonb     | Attachment metadata (JSON, nullable) |
| created_at          | timestamp | When the message was created         |
| updated_at          | timestamp | Last update time                     |
| source_table        | text      | Data source ID                       |

**Relationships:**
- `conversation_id` → `conversations.id`

---

## documents

Organization's living knowledge base. Document content is stored in object storage — use `list_documents` and `get_document_content` MCP tools to access.

| Column            | Type      | Description                                                                    |
| ----------------- | --------- | ------------------------------------------------------------------------------ |
| id                | text      | Document ID (PK, e.g. `docs_123`)                                              |
| title             | text      | Document title                                                                 |
| type              | text      | `PRODUCTS`, `EMOTION_SIGNALS`, `COMMERCIAL_SIGNALS`, `DEAL_PIPELINE`, `HEALTH_PIPELINE`, `OTHER` |
| is_active         | boolean   | Whether the document is currently active                                       |
| version           | integer   | Document version number                                                        |
| minio_document_id | text      | Object storage identifier (internal)                                           |
| created_at        | timestamp | Creation time                                                                  |
| updated_at        | timestamp | Last update time                                                               |

---

## participants

Resolved conversation participants with real contact IDs.

| Column            | Type   | Description                                 |
| ----------------- | ------ | ------------------------------------------- |
| id                | text   | Participant entry ID                        |
| conversation_id   | text   | Conversation ID (FK)                        |
| contact_id        | text   | Resolved contact ID (FK)                    |
| contact_type      | text   | `INTERNAL_USER` or `END_USER`               |
| name              | text   | Contact name                                |
| email             | text   | Participant email                           |
| participant_types | text[] | `FROM`, `TO`, `CC`, `BCC`, `INITIAL_SENDER` |

---

## ticket_participants

| Column     | Type   | Description                                            |
| ---------- | ------ | ------------------------------------------------------ |
| id         | text   | Participant entry ID                                   |
| ticket_id  | text   | Ticket ID (FK)                                         |
| contact_id | text   | Resolved contact ID (FK)                               |
| types      | text[] | `REQUESTER`, `SUBMITTER`, `ASSIGNEE`, `CC`, `FOLLOWER` |

---

## deals

Sales opportunities with extracted JSON fields.

| Column                  | Type      | Description                           |
| ----------------------- | --------- | ------------------------------------- |
| id                      | text      | Deal ID (PK, e.g. `deal_123`)         |
| company_id              | text      | Associated company ID (FK)            |
| owner_contact_id        | text      | Deal owner contact ID (FK)            |
| pipeline_id             | text      | Pipeline ID                           |
| name                    | text      | Deal name                             |
| summary                 | text      | AI-generated summary                  |
| created_at              | timestamp | Creation time                         |
| updated_at              | timestamp | Last update time                      |
| amount                  | bigint    | Deal amount in smallest currency unit |
| amount_currency         | text      | Currency code (e.g. `USD`)            |
| current_stage           | text      | Current stage name                    |
| expected_close_date     | date      | Expected close date                   |
| actual_close_date       | timestamp | Actual close date                     |
| status                  | text      | Deal status                           |
| health_score            | integer   | Health score (0-100)                  |
| health_trend            | text      | Health trend direction                |
| health_status           | text      | Health status label                   |
| forecast_period         | text      | Forecast period                       |
| forecast_category       | text      | Forecast category                     |
| forecast_fiscal_year    | integer   | Fiscal year                           |
| forecast_fiscal_quarter | integer   | Fiscal quarter                        |
| forecast_probability    | integer   | Win probability (0-100)               |
| current_gate            | text      | Current stage gate                    |
| next_gate               | text      | Next stage gate                       |
| gate_recommendation     | text      | Gate recommendation                   |

**Relationships:**
- `company_id` → `companies.id`
- `owner_contact_id` → `contacts.id`
- Join with `deal_risks` on `deal_id`
- Join with `deal_stakeholders` on `deal_id`
- Join with `deal_qualifications` on `deal_id`
- Join with `deal_interactions` on `deal_id`

---

## deal_risks

| Column      | Type | Description         |
| ----------- | ---- | ------------------- |
| id          | text | Risk entry ID       |
| deal_id     | text | Deal ID (FK)        |
| risk_id     | text | Risk identifier     |
| type        | text | Risk type           |
| severity    | text | Severity level      |
| name        | text | Risk name           |
| mitigation  | text | Mitigation strategy |
| description | text | Risk description    |

---

## deal_stakeholders

| Column           | Type   | Description              |
| ---------------- | ------ | ------------------------ |
| id               | text   | Stakeholder entry ID     |
| deal_id          | text   | Deal ID (FK)             |
| stakeholder_id   | text   | Stakeholder identifier   |
| name             | text   | Stakeholder name         |
| email            | text   | Stakeholder email        |
| title            | text   | Job title                |
| contact_id       | text   | Resolved contact ID (FK) |
| sentiment        | text   | Stakeholder sentiment    |
| engagement_level | text   | Engagement level         |
| roles            | text[] | Roles in the deal        |

---

## deal_qualifications

| Column           | Type   | Description              |
| ---------------- | ------ | ------------------------ |
| id               | text   | Qualification entry ID   |
| deal_id          | text   | Deal ID (FK)             |
| qualification_id | text   | Qualification identifier |
| name             | text   | Qualification name       |
| score            | text   | Score value              |
| status           | text   | Qualification status     |
| changes          | text   | Recent changes           |
| gaps             | text[] | Identified gaps          |

---

## deal_interactions

Junction table linking deals to conversations.

| Column          | Type   | Description          |
| --------------- | ------ | -------------------- |
| id              | bigint | Auto-increment ID    |
| deal_id         | text   | Deal ID (FK)         |
| conversation_id | text   | Conversation ID (FK) |

---

## entitlements

Customer contracts, licenses, and subscriptions.

| Column                | Type      | Description                                                             |
| --------------------- | --------- | ----------------------------------------------------------------------- |
| id                    | text      | Entitlement ID (PK, e.g. `ent_123`)                                     |
| company_id            | text      | Company ID (FK)                                                         |
| product               | text      | Product name                                                            |
| type                  | text      | Entitlement type                                                        |
| state                 | text      | `draft`, `live`, `expired`, `cancelled`                                 |
| revenue               | bigint    | Revenue amount                                                          |
| contract_value        | float     | Total contract value                                                    |
| revenue_type          | text      | `direct`, `revenue_share`, `commission`, `outcome`                      |
| pricing_model         | text      | `flat`, `usage`, `tiered`, `percentage`, `hybrid`                       |
| start_date            | timestamp | Start date                                                              |
| renewal_date          | timestamp | Renewal date                                                            |
| billing_cycle         | text      | `monthly`, `quarterly`, `yearly`, `three_yearly`, `permanent`, `custom` |
| unit                  | text      | `user`, `agent`, `resource`, `project`, `transaction`, `units`          |
| volume                | integer   | Licensed volume                                                         |
| components            | jsonb     | Entitlement components (JSON)                                           |
| channel               | text      | `direct`, `partner`, `marketplace`                                      |
| parent_entitlement_id | text      | Parent entitlement for renewals                                         |
| source_document_id    | text      | Source document ID                                                      |
| deal_id               | text      | Associated deal ID (FK)                                                 |
| created_by            | text      | `system`, `ai_inferred`, `user_manual`                                  |
| approved_by           | text      | Approver identifier                                                     |
| notes                 | text      | Free-form notes                                                         |
| created_at            | timestamp | Creation time                                                           |
| updated_at            | timestamp | Last update time                                                        |

---

## company_health

AI-assessed customer health records.

| Column                      | Type      | Description                             |
| --------------------------- | --------- | --------------------------------------- |
| id                          | text      | Health record ID (PK, e.g. `ch_123`)    |
| company_id                  | text      | Company ID (FK, unique)                 |
| health_pipeline_document_id | text      | Pipeline document ID                    |
| overall_status              | text      | `healthy`, `needs_attention`, `at_risk` |
| headline                    | text      | Health headline summary                 |
| executive_brief             | text      | Executive brief summary                 |
| created_at                  | timestamp | Creation time                           |
| updated_at                  | timestamp | Last update time                        |

---

## health_stakeholders

| Column              | Type   | Description                        |
| ------------------- | ------ | ---------------------------------- |
| id                  | text   | Stakeholder entry ID               |
| company_health_id   | text   | Health record ID (FK)              |
| contact_id          | text   | Contact ID                         |
| name                | text   | Stakeholder name                   |
| email               | text   | Stakeholder email                  |
| role                | text   | Role in the relationship           |
| title               | text   | Job title                          |
| action              | text   | Recommended action                 |
| sentiment           | text   | Current sentiment                  |
| role_confidence     | text   | Confidence in role assignment      |
| sentiment_change    | text   | Sentiment trend                    |
| sentiment_rationale | text   | Rationale for sentiment assessment |
| flags               | text[] | Warning flags                      |

---

## health_risks

| Column            | Type | Description           |
| ----------------- | ---- | --------------------- |
| id                | text | Risk entry ID         |
| company_health_id | text | Health record ID (FK) |
| risk              | text | Risk name             |
| type              | text | Risk type             |
| severity          | text | Severity level        |
| description       | text | Risk description      |
| mitigation        | text | Mitigation strategy   |

---

## health_assessment_dimensions

| Column            | Type | Description                                                                                                              |
| ----------------- | ---- | ------------------------------------------------------------------------------------------------------------------------ |
| id                | text | Dimension entry ID                                                                                                       |
| company_health_id | text | Health record ID (FK)                                                                                                    |
| dimension         | text | `engagement`, `sentiment`, `stakeholder_health`, `support_health`, `value_realization`, `strategic_alignment`, `overall` |
| status            | text | Dimension status                                                                                                         |
| change            | text | Change direction                                                                                                         |
| rationale         | text | Assessment rationale                                                                                                     |

---

## products

| Column      | Type      | Description          |
| ----------- | --------- | -------------------- |
| name        | text      | Product name         |
| description | text      | Product description  |
| product     | text      | Product name (alias) |
| created_at  | timestamp | Creation time        |
| updated_at  | timestamp | Last update time     |

---

## product_features

| Column      | Type      | Description                        |
| ----------- | --------- | ---------------------------------- |
| id          | text      | Feature ID (PK)                    |
| name        | text      | Feature name                       |
| description | text      | Feature description                |
| product     | text      | Parent product name (denormalized) |
| created_at  | timestamp | Creation time                      |
| updated_at  | timestamp | Last update time                   |

---

## product_usages

| Column          | Type      | Description                |
| --------------- | --------- | -------------------------- |
| company_id      | text      | Company ID (FK)            |
| end_user_id     | text      | Contact ID of the end user |
| product         | text      | Product name               |
| product_feature | text      | Feature name               |
| usage_count     | integer   | Usage count for this event |
| date            | timestamp | Event date                 |
| event_name      | text      | Event type name            |

---

## work_items

AI-generated work items representing actionable interactions.

| Column            | Type      | Description                                 |
| ----------------- | --------- | ------------------------------------------- |
| id                | text      | Work item ID (PK, e.g. `wrk_123`)           |
| contact_id        | text      | Contact ID of the related person (FK)       |
| title             | text      | Work item title                             |
| description       | text      | Work item description                       |
| urgency           | text      | `LOW`, `MEDIUM`, `HIGH`                     |
| due_date          | timestamp | Due date (nullable)                         |
| created_at        | timestamp | Creation time                               |
| updated_at        | timestamp | Last update time                            |
| last_message_date | timestamp | Date of the last message in the interaction |
| interaction_type  | text      | `EMAIL_THREAD`, `TICKET`, `MEETING`         |
| conversation_id   | text      | Linked conversation ID (FK, nullable)       |
| ticket_id         | text      | Linked ticket ID (FK, nullable)             |
| deal_id           | text      | Linked deal ID (FK, nullable)               |
| company_health_id | text      | Linked company health ID (FK, nullable)     |
| intents           | jsonb     | Array of detected intents (JSON)            |
| company_ids       | text[]    | Associated company IDs                      |
| thread_id         | text      | Thread identifier (nullable)                |

---

## action_items

Suggested actions generated by AI for work items.

| Column             | Type      | Description                                                  |
| ------------------ | --------- | ------------------------------------------------------------ |
| id                 | text      | Action item ID (PK, e.g. `act_123`)                          |
| work_item_id       | text      | Parent work item ID (FK)                                     |
| title              | text      | Action item title                                            |
| description        | text      | Action item description (nullable)                           |
| rationale          | text[]    | Reasons for the suggestion                                   |
| agent_name         | text      | Name of the AI agent that suggested this                     |
| button_title       | text      | Label for the execution button                               |
| instructions       | text      | Human-readable instructions                                  |
| agent_instructions | text      | Machine-readable instructions for AI execution               |
| status             | text      | `SUGGESTED_TO_USER`, `CANCELLED_BY_USER`, `EXECUTED_BY_USER` |
| outcome            | jsonb     | Execution outcome (nullable, JSON)                           |
| thread_id          | text      | Thread identifier (nullable)                                 |
| created_at         | timestamp | Creation time                                                |
| updated_at         | timestamp | Last update time                                             |
| task_id            | text      | Linked task ID (FK, nullable)                                |

---

## signals

Named signal definitions (emotions, request types, account signals).

| Column      | Type      | Description                                        |
| ----------- | --------- | -------------------------------------------------- |
| id          | text      | Signal ID (PK)                                     |
| name        | text      | Signal name                                        |
| description | text      | Signal description                                 |
| type        | text      | `EMOTION`, `REQUEST_TYPE`, `ADDITIONAL_CATEGORIES` |
| created_at  | timestamp | Creation time                                      |
| updated_at  | timestamp | Last update time                                   |

---

## interaction_signals

Signals detected in conversations and tickets, with rationale.

| Column         | Type | Description                  |
| -------------- | ---- | ---------------------------- |
| id             | text | Entry ID                     |
| interaction_id | text | Conversation or ticket ID    |
| signal_id      | text | Signal ID (FK)               |
| signal_name    | text | Signal name (denormalized)   |
| signal_type    | text | Signal type (denormalized)   |
| rationale      | text | Why this signal was detected |

---

## tasks

Internal tasks assigned to team members.

| Column           | Type      | Description                              |
| ---------------- | --------- | ---------------------------------------- |
| id               | text      | Task ID (PK)                             |
| company_id       | text      | Associated company ID (FK)               |
| title            | text      | Task title                               |
| status           | text      | `TODO`, `IN_PROGRESS`, `BLOCKED`, `DONE` |
| content          | jsonb     | Task content (rich text JSON)            |
| markdown_content | text      | Task content in markdown                 |
| priority         | text      | `LOW`, `NORMAL`, `HIGH`, `URGENT`        |
| tags             | text[]    | Tags                                     |
| position         | integer   | Sort position within status column       |
| sharing_status   | text      | `PUBLIC` or `PRIVATE`                    |
| created_at       | timestamp | Creation time                            |
| updated_at       | timestamp | Last update time                         |
| due_at           | timestamp | Due date                                 |
| created_by_id    | text      | Creator contact ID                       |

---

## task_assignees

| Column  | Type | Description                    |
| ------- | ---- | ------------------------------ |
| id      | text | Entry ID                       |
| task_id | text | Task ID (FK)                   |
| real_id | text | Resolved contact or company ID |
| name    | text | Assignee name                  |
| type    | text | `USER`, `TEAM`, `COMPANY`      |

---

## notes

Free-form notes attached to companies or contacts.

| Column              | Type      | Description                   |
| ------------------- | --------- | ----------------------------- |
| id                  | text      | Note ID (PK)                  |
| title               | text      | Note title                    |
| content             | text      | Note content in markdown      |
| tags                | text[]    | Tags                          |
| sharing_status      | text      | `PUBLIC` or `PRIVATE`         |
| related_entity_type | text      | `COMPANY` or `CONTACT`        |
| company_id          | text      | Associated company ID (FK)    |
| contact_id          | text      | Associated contact ID (FK)    |
| created_by_id       | text      | Creator ID                    |
| created_at          | timestamp | Creation time                 |
| updated_at          | timestamp | Last update time              |

---

## ticket_histories

Historical timeline entries for tickets with signal analysis.

| Column           | Type      | Description                   |
| ---------------- | --------- | ----------------------------- |
| ticket_id        | text      | Ticket ID (FK)                |
| comment_id       | text      | Associated comment ID         |
| sentiment_score  | integer   | Sentiment score at this point |
| sentiment_enum   | text      | Sentiment classification      |
| ticket_stage     | text      | Ticket stage at this point    |
| emotions         | text[]    | Emotion signal names          |
| products         | text[]    | Product names                 |
| product_features | text[]    | Feature names                 |
| account_signals  | text[]    | Account signal names          |
| created_at       | timestamp | Entry timestamp               |

---

## ticket_tasks

Junction table linking tickets to tasks.

| Column    | Type | Description    |
| --------- | ---- | -------------- |
| ticket_id | text | Ticket ID (FK) |
| task_id   | text | Task ID (FK)   |

---

## dynamic_field_mappings

Field mapping definitions used for data source value translation.

| Column          | Type      | Description                |
| --------------- | --------- | -------------------------- |
| id              | text      | Mapping ID (PK)            |
| data_source_id  | text      | Data source ID             |
| data_model      | text      | Model name (e.g. `ticket`) |
| field_key       | text      | Field key (e.g. `source`)  |
| field_key_value | text      | Original field value       |
| mapped_value    | text      | Mapped/translated value    |
| created_at      | timestamp | Creation time              |
| updated_at      | timestamp | Last update time           |
