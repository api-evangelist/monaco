---
name: Prep an account briefing before a meeting
description: >-
  Assemble a single briefing for one account — the account record, its open
  opportunities, recent meetings with AI summaries, engaged contacts, and
  outstanding tasks — using the Monaco Public API.
api: openapi/monaco-openapi-original.json
operations: [get_account, list_opportunities, list_meetings, get_meeting, list_contacts, list_tasks]
---

# Prep an account briefing

Use the Monaco Public API (`https://api.monaco.com`) to gather everything on one account.

## Auth
Send `Authorization: Bearer mks_<key>` and `Content-Type: application/json` on every request. Keys come from Settings > API Keys. Rate limit: 100 requests/minute per organization — honor `Retry-After` on 429.

## Steps
1. **Resolve the account.** `get_account` (`GET /v1/accounts/{account_id}`) for enriched company info, scoring, and custom fields. If you only have a domain, `list_accounts` (`POST /v1/accounts/list`) with a `filters` rule on `domain`.
2. **Open opportunities.** `list_opportunities` (`POST /v1/opportunities/list`) filtered by `account_id`; for detail (AI summary, risks) call `get_opportunity`.
3. **Recent meetings.** `list_meetings` (`POST /v1/meetings/list`) filtered by `account_id`, `sort: -created_at`; `get_meeting` returns the AI summary and full transcript.
4. **Engaged contacts.** `list_contacts` (`POST /v1/contacts/list`) filtered by `account_id`.
5. **Outstanding tasks.** `list_tasks` (`POST /v1/tasks/list`) filtered by `account_id` and open status.

## Conventions
- Lists use `{filters, sort, page, page_size}` bodies; page through until `pagination.page >= pagination.total_pages`.
- Discover filterable/sortable fields via `get_field_schemas_for_entity` (`GET /v1/schemas/{entity}`).
- Errors return `{"error": {"code","message"}}`.
