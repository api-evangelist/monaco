---
name: Run pipeline hygiene on opportunities
description: >-
  Find stale or slipped opportunities with the Monaco Public API and update or
  close them in bulk, adding a note.
api: openapi/monaco-openapi-original.json
operations: [list_opportunities, get_opportunity, update_opportunity]
---

# Run pipeline hygiene

Keep the pipeline clean by finding opportunities that have slipped and acting on them.

## Auth
`Authorization: Bearer mks_<key>`, `Content-Type: application/json`. 100 requests/minute per org.

## Steps
1. **Find slipped deals.** `list_opportunities` (`POST /v1/opportunities/list`) with a `filters` expression — e.g. `stage is Proposal` AND `estimated_close_date is_before <today>`. Use `get_field_schemas_for_entity` for `opportunities` to learn the exact field keys and enum values (filter by key, not display name).
2. **Inspect if needed.** `get_opportunity` (`GET /v1/opportunities/{opportunity_id}`) for the AI summary and identified risks before acting.
3. **Update or close.** `update_opportunity` (`PATCH /v1/opportunities/{opportunity_id}`) to move stage (e.g. to Closed Lost) and set a note such as "No response after follow-up." A supplied `tags` list replaces the existing tags.

## Conventions
- Page through list results via `page`/`page_size` (max 500) until `total_pages`.
- Dates are ISO 8601 (trailing `Z` for UTC).
- Errors return `{"error": {"code","message"}}`; a 422 means an unsupported sort/filter field.
