---
name: legal-hold
description: >
  This skill should be used when the user asks about "custodians", "legal hold",
  "add custodians", "release custodian", "send notification", "hold notification",
  "endpoint hold", "apply hold", "release hold", or any task involving managing
  legal hold custodians, notifications, or endpoint preservation in OpenAxes IA.
metadata:
  version: "0.2.0"
---

# Legal Hold Management — OpenAxes IA

Manage legal hold operations through the MCP tools: `get_custodians`, `add_custodians`,
`release_custodian`, `send_notification`, `apply_endpoint_hold`, `release_endpoint_hold`.

## Conventions

Entity references in this skill accept **either a friendly identifier (name, email, or abbreviation) or a hashed ID**. The server resolves the value, scoped to the caller's tenant. Resolution is case-insensitive exact match.

When a name matches multiple entities, the tool returns `resolution_ambiguous` with a `candidates` array — present the choices to the user and re-call with one of the returned hashed IDs.

When a name matches no entity, the tool returns `resolution_not_found` with a `suggestions` array of close matches.

Mutating tools that require approval return `approval_required`. If `approver` and/or `comments` are missing, the response includes `missingFields` indicating what to ask the user for, plus `candidateApprovers` listing users who can approve.

Statuses to handle: `success`, `validation_failure`, `not_found`, `authorization_failure`, `domain_failure`, `resolution_ambiguous`, `resolution_not_found`, `approval_required`.

## Approval flow

`add_custodians` and `release_custodian` may require approval. The flow is:

1. Caller invokes the tool **without** `approver` and `comments`.
2. If the caller has direct write permission, the tool acts and returns `success`.
3. If the caller lacks permission, the tool returns `approval_required` with `missingFields` (subset of `["approver", "comments"]`) and a `candidateApprovers` list of users who can approve.
4. Present `candidateApprovers` to the user, ask for the approver name/email and a reason (comments).
5. Re-invoke the tool with `approver` and `comments` populated.
6. The tool resolves the approver, queues the approval request, and returns `approval_required` with `success: true` and the resolved `approver` echo.

Empty/whitespace `comments` counts as missing. `candidateApprovers` is omitted when only `comments` is missing (it's already known who the approver is).

## Workflow guidance

### Listing custodians

Use `get_custodians` with `investigation` (investigation name or hashed ID). Supports filtering by `criteria` (name/email match) and `status` (legal hold status enum). Paginate with `page` (default 1) and `pageSize` (default 50, max 200).

### Adding custodians

Use `add_custodians`. Requires `investigation` and at least one custodian entry with:

- `name` (required) — custodian display name
- `email` (required) — custodian email address; resolves an existing Account or creates one
- `role` (required) — role name or abbreviation (case-insensitive exact match)
- `managers[]` (optional) — each with `name` and `email`

#### Approval flow

The tool uses a **partial-form-fill** contract:

1. If the caller has direct write permission → custodians are added immediately.
2. If the caller lacks permission and `approver` and/or `comments` are missing → the server returns `approval_required` (`success: false`) with a `missingFields` list.
   - When `approver` is missing, `candidateApprovers` is also populated (up to 25 active AttorneyManager users).
   - When `approver` is supplied but `comments` is missing, `candidateApprovers` is **not** included.
3. If the caller lacks permission and both `approver` (user name or email) and `comments` are supplied → the approval request is dispatched and the server returns `approval_required` (`success: true`) with the resolved `approver` echoed back.

When you receive `approval_required` with `success: false`, ask the user for the missing fields listed in `missingFields`, then re-call the tool with all fields populated.

### Releasing a custodian

Use `release_custodian` with `investigation` (name or hashed ID) and `custodian` (name, email, or hashed ID scoped to that investigation).

#### Approval flow

The tool uses the same **partial-form-fill** contract as `add_custodians`:

1. If the caller has direct write permission → custodian is released immediately.
2. If the caller lacks permission and `approver` and/or `comments` are missing → the server returns `approval_required` (`success: false`) with a `missingFields` list.
   - When `approver` is missing, `candidateApprovers` is also populated (up to 25 active AttorneyManager users).
   - When `approver` is supplied but `comments` is missing, `candidateApprovers` is **not** included.
3. If the caller lacks permission and both `approver` (user name or email) and `comments` are supplied → the approval request is dispatched and the server returns `approval_required` (`success: true`) with the resolved `approver` echoed back.

Domain failure if the custodian is already released or deleted.

### Sending notifications

Use `send_notification` with `investigation` (investigation name or hashed ID). Optionally provide:

- `custodians` — array of custodian names or emails scoped to the investigation; omit to notify all pending custodians. If a token matches more than one custodian the server returns `resolution_ambiguous`; if it matches none it returns `resolution_not_found`.
- `includeReviewer` — copy reviewer (default false)
- `includeManagers` — copy managers (default false)

### Endpoint holds

Use `apply_endpoint_hold` or `release_endpoint_hold` with `investigation` (investigation name or hashed ID) and `endpoint` (endpoint friendly name or hashed ID scoped to that investigation).

If a name matches multiple endpoints, the server returns `resolution_ambiguous` with a `candidates` list. If no match is found, the server returns `resolution_not_found` with nearest suggestions.

Constraints:
- Only `M365Custodian` endpoints are supported
- Apply: endpoint hold status must be `None` or `Released`
- Release: endpoint hold status must be `Active`

## Important

Legal hold operations are consequential — always confirm with the user before executing mutations. Summarize what will happen (who is being added/released, which endpoints affected) and wait for explicit confirmation.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
