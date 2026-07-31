---
name: Build a tagged outbound audience
description: >-
  Find contacts matching firmographic/demographic criteria with the Monaco
  Public API, create an audience tag, and apply it.
api: openapi/monaco-openapi-original.json
operations: [list_contacts, list_tags, create_tag, update_contact]
---

# Build a tagged outbound audience

Segment contacts and tag them as a reusable audience.

## Auth
`Authorization: Bearer mks_<key>`, `Content-Type: application/json`. 100 requests/minute per org.

## Steps
1. **Segment contacts.** `list_contacts` (`POST /v1/contacts/list`) with a `filters` expression on firmographic/demographic fields (title, location, etc.). Confirm field keys with `get_field_schemas_for_entity` for `contacts`. Page through all matches.
2. **Reuse or create the tag.** `list_tags` (`GET /v1/tags/?object=contacts`) to check for an existing tag; otherwise `create_tag` (`POST /v1/tags/`) with `object: contacts` and a unique `name` (names are unique per object type per org).
3. **Apply the tag.** `update_contact` (`PATCH /v1/contacts/{contact_id}`) with a `tags` list — note a supplied `tags` list **replaces** the contact's existing tags, so include current tags you want to keep.

## Notes
- The REST API models sequences as read/update only; there is no enroll-in-sequence operation, so drive enrollment through the Monaco app or MCP where exposed.
- Errors return `{"error": {"code","message"}}`.
