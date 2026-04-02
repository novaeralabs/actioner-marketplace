# Query Examples

Common query patterns for the Actioner data model.

## JOIN Patterns

```sql
-- Tickets with company info
SELECT t.*, c.name AS company_name
FROM tickets t
JOIN ticket_participants tp ON tp.ticket_id = t.id AND 'REQUESTER' = ANY(tp.types)
JOIN contacts ct ON ct.id = tp.contact_id
JOIN companies c ON c.id = ct.company_id;

-- Deals with company and health
SELECT d.*, c.name AS company_name, ch.overall_status
FROM deals d
JOIN companies c ON c.id = d.company_id
LEFT JOIN company_health ch ON ch.company_id = c.id;

-- Conversation participants
SELECT c.subject, p.name, p.email, p.participant_types
FROM conversations c
JOIN participants p ON p.conversation_id = c.id;
```

## Date Filtering

```sql
-- Tickets created in the last 30 days
SELECT * FROM tickets WHERE created_at > now() - interval '30 days';

-- Deals closing this quarter
SELECT * FROM deals
WHERE expected_close_date BETWEEN date_trunc('quarter', now())
  AND date_trunc('quarter', now()) + interval '3 months';
```

## Aggregations

```sql
-- Ticket count by status
SELECT status, count(*) FROM tickets GROUP BY status;

-- Total deal pipeline value
SELECT sum(amount), count(*) FROM deals WHERE status != 'closed_lost';

-- Average sentiment by company
SELECT c.name, avg(t.sentiment_score) AS avg_sentiment
FROM tickets t
JOIN ticket_participants tp ON tp.ticket_id = t.id AND 'REQUESTER' = ANY(tp.types)
JOIN contacts ct ON ct.id = tp.contact_id
JOIN companies c ON c.id = ct.company_id
GROUP BY c.name
ORDER BY avg_sentiment;
```

## Array Column Queries

```sql
-- Companies in a specific industry
SELECT * FROM companies WHERE 'SaaS' = ANY(industries);

-- Tickets with a specific emotion
SELECT * FROM tickets WHERE 'frustration' = ANY(emotions);

-- Display array as comma-separated string
SELECT name, array_to_string(industries, ', ') AS industry_list FROM companies;
```

## search_entities Example

```json
{
  "label": "Deals closing this quarter",
  "sql": "SELECT d.id, d.name, c.name AS company_name, c.id AS company_id, d.amount, d.current_stage, d.expected_close_date FROM deals d JOIN companies c ON c.id = d.company_id WHERE d.expected_close_date BETWEEN date_trunc('quarter', now()) AND date_trunc('quarter', now()) + interval '3 months'",
  "defaultSort": { "column": "amount", "direction": "DESC" },
  "pageSize": 50,
  "columns": [
    { "key": "id", "label": "ID", "dataType": "text", "visible": false },
    { "key": "company_id", "label": "Company ID", "dataType": "text", "visible": false },
    {
      "key": "name",
      "label": "Deal",
      "dataType": "link",
      "sortable": true,
      "linkAction": { "toolName": "get_deal", "args": { "dealId": "id" } }
    },
    {
      "key": "company_name",
      "label": "Company",
      "dataType": "link",
      "sortable": true,
      "linkAction": { "toolName": "get_company", "args": { "companyId": "company_id" } }
    },
    {
      "key": "amount",
      "label": "Amount",
      "dataType": "decimal",
      "alignment": "right",
      "sortable": true
    },
    { "key": "current_stage", "label": "Stage", "dataType": "enum", "sortable": true },
    { "key": "expected_close_date", "label": "Close Date", "dataType": "date", "sortable": true }
  ]
}
```
