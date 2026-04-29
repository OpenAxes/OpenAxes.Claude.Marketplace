---
name: investigation-management
description: >
  This skill should be used when the user asks to "list investigations",
  "show my investigations", "create an investigation", "close an investigation",
  "reopen an investigation", "get investigation details", "check investigation status",
  or any task involving viewing, creating, or managing the lifecycle of OpenAxes IA investigations.
metadata:
  version: "0.2.0"
---

# Investigation Management — OpenAxes IA

Manage the full lifecycle of investigations in OpenAxes IA through the MCP tools:
`list_investigations`, `get_investigation`, `get_investigation_templates`, `create_investigation`,
`close_investigation`, `reopen_investigation`.

## Conventions

Entity references in this skill accept **either a friendly identifier (name, email, or abbreviation) or a hashed ID**. The server resolves the value, scoped to the caller's tenant. Resolution is case-insensitive exact match.

When a name matches multiple entities, the tool returns `resolution_ambiguous` with a `candidates` array — present the choices to the user and re-call with one of the returned hashed IDs.

When a name matches no entity, the tool returns `resolution_not_found` with a `suggestions` array of close matches.

Mutating tools that require approval return `approval_required`. If `approver` and/or `comments` are missing, the response includes `missingFields` indicating what to ask the user for, plus `candidateApprovers` listing users who can approve.

Statuses to handle: `success`, `validation_failure`, `not_found`, `authorization_failure`, `domain_failure`, `resolution_ambiguous`, `resolution_not_found`, `approval_required`.

## Workflow guidance

### Listing investigations

Use `list_investigations`. Supports filtering by `criteria` (name/description match), `status` (`draft`, `active`, `closed`, `processing`, `deleted`), and `investigationType` (`normalcase`, `template`, `featureindex`). Paginate with `page` (default 1) and `pageSize` (default 25, max 100).

When the user asks vague questions like "show my investigations," start with active investigations and offer to filter further.

### Getting investigation details

Use `get_investigation` with an `investigation` parameter (name or hashed ID). Returns full detail including workspaces, reviewers, counts (custodians, endpoints, searches, exports), and flags (legal hold, indexable/analyzable endpoints).

### Creating investigations

Always start by calling `get_investigation_templates` to show the user available templates. Then call `create_investigation` with:

- `template` (required) — template name (case-insensitive exact match) or hashed ID from `get_investigation_templates`
- `name` (required)
- `fileDate` (required, ISO-8601)
- `description` (optional)
- `workspace` (optional) — workspace name, abbreviation, or hashed ID

Confirm the template choice and investigation name with the user before creating.

### Closing and reopening

Use `close_investigation` with an `investigation` parameter (name or hashed ID). Use `reopen_investigation` with an `investigation` parameter (name or hashed ID). Reopening optionally accepts `resendNotification` (default false) to re-send legal hold notifications.

Always confirm closure/reopen with the user — these are significant state changes.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
