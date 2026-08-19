---
name: Monaco
description: Use when building integrations with Monaco's sales engagement platform, querying or managing contacts, accounts, opportunities, campaigns, sequences, and meetings. Reach for this skill when working with the REST API (api.monaco.com) or MCP server (mcp.monaco.com/mcp) to automate outbound workflows, manage CRM data, or build custom dashboards from sales data.
metadata:
    mintlify-proj: monaco
    version: "1.0"
---

# Monaco Skill

## Product summary

Monaco is a sales engagement platform with a REST API and MCP server for programmatic access to contacts, accounts, opportunities, campaigns, sequences, meetings, and tasks. The REST API is hosted at `api.monaco.com` and uses bearer token authentication with API keys (prefix `mks_`). The MCP server at `mcp.monaco.com/mcp` uses OAuth 2.0 and connects AI agents directly to Monaco workspaces. Both share a 100 requests/minute rate limit per organization. The API is currently in Beta — expect potential changes to methods, responses, and parameters.

**Key endpoints:**
- `POST /v1/{entity}/list` — paginated list with filters, sorting, pagination
- `POST /v1/{entity}/` — create resource
- `GET /v1/{entity}/{id}` — fetch single resource
- `PATCH /v1/{entity}/{id}` — update resource
- `DELETE /v1/{entity}/{id}` — delete resource
- `GET /v1/schemas/{entity}` — discover filterable/sortable fields

**Entities:** contacts, accounts, opportunities, tasks, meetings, sequences, campaigns, audiences, tags, sequence templates, users.

**Primary docs:** https://docs.monaco.com

## When to use

Reach for this skill when:
- Building integrations that read or write contacts, accounts, or opportunities
- Creating or managing campaigns and sequence enrollments
- Querying meetings with AI-generated summaries and transcripts
- Building custom dashboards or reports from Monaco data
- Automating outbound workflows (audiences → campaigns → sequences)
- Connecting AI agents to Monaco via MCP for natural-language data access
- Filtering, sorting, or paginating through large datasets
- Managing tags, tasks, or custom fields on records

## Quick reference

### Authentication

**REST API:** Include `Authorization: Bearer mks_<key>` header on every request.

**MCP:** OAuth 2.0 flow handled by client on first connection. For API key auth with Codex/ChatGPT: `codex mcp add monaco --url https://mcp.monaco.com/mcp --bearer-token-env-var MONACO_MCP_TOKEN`

### Common list patterns

All list endpoints accept `POST /v1/{entity}/list` with:

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `filters` | array or object | `[]` | Flat array (ANDed) or expression with `operator` and nested `filters` |
| `sort` | string | `-created_at` | Prefix with `-` for descending |
| `page` | int | 1 | 1-indexed |
| `page_size` | int | 50 | Max 500 |

### Filter conditions

| Condition | Value required | Use for |
| --- | --- | --- |
| `is` | yes | Exact match or enum value |
| `is_not` | yes | Not equal |
| `contains` | yes | Substring match |
| `is_empty` | no | Null or empty field |
| `is_not_empty` | no | Has a value |
| `is_before` / `is_after` | yes | Date comparison (ISO 8601) |

### Core entities and operations

| Entity | Create | Read | Update | Delete | List |
| --- | --- | --- | --- | --- | --- |
| Contact | `POST /v1/contacts/` | `GET /v1/contacts/{id}` | `PATCH /v1/contacts/{id}` | `DELETE /v1/contacts/{id}` | `POST /v1/contacts/list` |
| Account | `POST /v1/accounts/` | `GET /v1/accounts/{id}` | `PATCH /v1/accounts/{id}` | `DELETE /v1/accounts/{id}` | `POST /v1/accounts/list` |
| Opportunity | `POST /v1/opportunities/` | `GET /v1/opportunities/{id}` | `PATCH /v1/opportunities/{id}` | `DELETE /v1/opportunities/{id}` | `POST /v1/opportunities/list` |
| Campaign | `POST /v1/campaigns/` | `GET /v1/campaigns/{id}` | `PATCH /v1/campaigns/{id}` | — | `POST /v1/campaigns/list` |
| Audience | `POST /v1/audiences` | `GET /v1/audiences/{id}` | — | — | `POST /v1/audiences/list` |
| Sequence | — | `GET /v1/sequences/{id}` | — | — | `POST /v1/sequences/list` |
| Meeting | — | `GET /v1/meetings/{id}` | — | — | `POST /v1/meetings/list` |
| Task | `POST /v1/tasks/` | `GET /v1/tasks/{id}` | `PATCH /v1/tasks/{id}` | — | `POST /v1/tasks/list` |

### MCP tools

| Tool | Purpose |
| --- | --- |
| `list_contacts`, `get_contact` | Search and fetch contacts |
| `list_accounts`, `get_account` | Search and fetch accounts |
| `list_opportunities`, `get_opportunity` | Search and fetch opportunities |
| `list_meetings`, `get_meeting` | Fetch meetings with AI summary and transcript |
| `list_sequences`, `get_sequence` | List or get sequences with task details |
| `list_sequence_templates`, `get_sequence_template` | List or get templates with steps and transitions |
| `create_contact`, `update_contact`, `delete_contact` | Manage contacts |
| `create_opportunity`, `update_opportunity`, `delete_opportunity` | Manage opportunities |
| `upsert_account`, `delete_account` | Upsert or delete accounts |
| `create_task`, `update_task` | Manage tasks |

## Decision guidance

| Scenario | Use REST API | Use MCP |
| --- | --- | --- |
| Programmatic integration, backend service | ✓ | — |
| AI agent needs natural-language access | — | ✓ |
| Building custom dashboard in code | ✓ | — |
| One-off data queries in Claude/ChatGPT | — | ✓ |
| Bulk operations, batch processing | ✓ | — |
| Filtering contacts by complex criteria | ✓ | ✓ |
| Upsert vs create | Use `POST /v1/contacts/upsert` when you want to match on email or LinkedIn URL and update if found, otherwise create. Use `POST /v1/contacts/` when you always want a new record. |
| Percentage vs account owner distribution | Use `percentage` mode to split sends by fixed percentages across senders. Use `account_owner` mode to route each contact to its account owner (with a fallback sender). |

## Workflow

### Typical outbound campaign workflow

1. **Discover filterable fields** — Call `GET /v1/schemas/contacts` to see which fields support filtering and their allowed operators.

2. **Create or query an audience** — Either:
   - `POST /v1/audiences` with `contact_ids` (explicit list) or `filters` (query-based, max 500 contacts)
   - `POST /v1/audiences/list` to find existing audiences

3. **Get a sequence template** — `GET /v1/sequence-templates` to list available templates, then `GET /v1/sequence-templates/{id}` to inspect steps and transitions.

4. **Create a campaign** — `POST /v1/campaigns/` with:
   - `name` (unique within org)
   - `template_id` (from step 3)
   - `audience_ids` (from step 2)
   - `distribution` (percentage or account_owner mode with sender IDs and percentages/fallback)
   - Optional: `rules_of_engagement` overrides, `autopilot_settings`, `end_date`

5. **Monitor enrollment** — Check `enrollment_status` in campaign response (`pending`, `running`, `completed`, or `failed`). Contacts are enrolled asynchronously.

6. **Manage audiences** — Add/remove audiences from a running campaign with `POST /v1/campaigns/{id}/audiences` or `POST /v1/campaigns/{id}/audiences/remove`.

### Typical data query workflow

1. **List with filters** — `POST /v1/{entity}/list` with filters array or expression, sort, page, page_size.

2. **Iterate pages** — Increment `page` until `page >= total_pages` from pagination object.

3. **Get detail** — `GET /v1/{entity}/{id}` for full record including custom fields, AI summaries (opportunities, meetings), or task lists (sequences).

4. **Update or delete** — `PATCH /v1/{entity}/{id}` or `DELETE /v1/{entity}/{id}`.

## Common gotchas

- **Contact creation is slow** — Contact creation synchronously enriches data and resolves/creates accounts. Use generous timeouts (10s+) and keep creates off hot paths.
- **Upsert matching** — Upsert matches on email (case-insensitive) first, then LinkedIn URL. If both identifiers resolve to different contacts on the same account, returns 409.
- **Account resolution priority** — When creating a contact with only LinkedIn URL, also pass `account_id` or `domain`. Otherwise, domain is parsed from email.
- **Flat filters must be arrays** — A single filter rule sent as a bare object is invalid; wrap it in `[]`.
- **Enum values are keys, not display names** — Use the key value from `enum_field_settings.allowed_values` in schemas, not the human-readable name.
- **Custom fields are keyed by UUID** — Custom fields appear as `custom_field_<uuid>` in responses and filters. Use the full key in filters and sorts.
- **Removing all audiences fails** — A campaign must keep at least one audience; requests that would remove them all are rejected with 400.
- **Detaching audiences tears down sequences** — Removing an audience from a campaign stops in-flight sequences for its contacts.
- **Rate limit is org-wide** — 100 requests/minute shared across all users and API keys in the organization. Check `X-RateLimit-Remaining` and `Retry-After` headers.
- **API is in Beta** — Expect potential breaking changes to methods, responses, and parameters.
- **MCP permissions** — Adding a custom MCP connector may require workspace owner/admin approval. Reads are auto-approved; writes/deletes prompt for confirmation.
- **Sequence step editing is limited** — Only steps in active (non-suggested) sequences that haven't completed can be edited. Only `message` and `subject` fields are updatable.
- **Template variables must be namespaced** — Use `{{recipient.first_name}}`, `{{sender.company}}`, etc., not bare variable names.

## Verification checklist

Before submitting work:

- [ ] API key is valid and scoped to the correct organization
- [ ] All required fields are provided (e.g., `name` for campaigns, `template_id`, `audience_ids`, `distribution`)
- [ ] Filters use correct field keys and enum values (not display names)
- [ ] Pagination is handled correctly (loop until `page >= total_pages`)
- [ ] Rate limits are respected (implement backoff on 429)
- [ ] Contact creation requests have generous timeouts (10s+)
- [ ] Campaign distribution percentages sum to 100
- [ ] Campaigns have at least one audience
- [ ] Custom field keys include the full `custom_field_<uuid>` prefix
- [ ] Sequence template transitions use correct `transition_type` for channel crossings
- [ ] Template variables are namespaced (e.g., `{{recipient.first_name}}`)
- [ ] Error responses are handled (401, 403, 429, 4XX)

## Resources

- **Comprehensive page listing:** https://docs.monaco.com/llms.txt
- **Authentication & API keys:** https://docs.monaco.com/auth
- **Pagination, sorting, filtering:** https://docs.monaco.com/pagination-sorting-and-filtering
- **MCP overview & tools:** https://docs.monaco.com/mcp/overview

---

> For additional documentation and navigation, see: https://docs.monaco.com/llms.txt