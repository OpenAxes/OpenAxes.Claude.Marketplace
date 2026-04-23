---
name: investigation-management
description: >
  This skill should be used when the user asks to "list investigations",
  "show my investigations", "create an investigation", "close an investigation",
  "reopen an investigation", "get investigation details", "check investigation status",
  or any task involving viewing, creating, or managing the lifecycle of OpenAxes IA investigations.
metadata:
  version: "0.1.0"
---

# Investigation Management — OpenAxes IA

Manage the full lifecycle of investigations in OpenAxes IA through the MCP tools:
`list_investigations`, `get_investigation`, `get_investigation_templates`, `create_investigation`,
`close_investigation`, `reopen_investigation`.

## Conventions

All investigation identifiers are **hashed IDs** (opaque strings), not integer primary keys. Always use the hashed ID values returned by the server. If a hashed ID cannot be decoded, the server returns a `validation_failure`.

Every tool response follows a standard contract with `structuredContent.status` indicating outcome. Check for these statuses:

- `success` — completed normally
- `validation_failure` — bad input; check `validationErrors` for field-level details
- `not_found` — entity not accessible or does not exist
- `authorization_failure` — caller lacks permission
- `domain_failure` — business rule prevented the operation

## Workflow guidance

### Listing investigations

Use `list_investigations`. Supports filtering by `criteria` (name/description match), `status` (`draft`, `active`, `closed`, `processing`, `deleted`), and `investigationType` (`normalcase`, `template`, `featureindex`). Paginate with `page` (default 1) and `pageSize` (default 25, max 100).

When the user asks vague questions like "show my investigations," start with active investigations and offer to filter further.

### Getting investigation details

Use `get_investigation` with a hashed `investigationId`. Returns full detail including workspaces, reviewers, counts (custodians, endpoints, searches, exports), and flags (legal hold, indexable/analyzable endpoints).

### Creating investigations

Always start by calling `get_investigation_templates` to show the user available templates. Then call `create_investigation` with:

- `templateId` (required, hashed) — from the templates list
- `name` (required)
- `fileDate` (required, ISO-8601)
- `description` (optional)
- `workspaceId` (optional, hashed)

Confirm the template choice and investigation name with the user before creating.

### Closing and reopening

Use `close_investigation` or `reopen_investigation` with a hashed `investigationId`. Reopening optionally accepts `resendNotification` (default false) to re-send legal hold notifications.

Always confirm closure/reopen with the user — these are significant state changes.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
