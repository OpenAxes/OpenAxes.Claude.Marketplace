---
name: legal-hold
description: >
  This skill should be used when the user asks about "custodians", "legal hold",
  "add custodians", "release custodian", "send notification", "hold notification",
  "endpoint hold", "apply hold", "release hold", or any task involving managing
  legal hold custodians, notifications, or endpoint preservation in OpenAxes IA.
metadata:
  version: "0.1.0"
---

# Legal Hold Management — OpenAxes IA

Manage legal hold operations through the MCP tools: `get_custodians`, `add_custodians`,
`release_custodian`, `send_notification`, `apply_endpoint_hold`, `release_endpoint_hold`.

## Conventions

All identifiers (investigation, custodian, endpoint, role, account) are **hashed IDs**. The server returns `validation_failure` if a hashed ID cannot be decoded.

Mutation tools that modify legal hold state support an **approval workflow**: if the caller lacks direct write access, the tool returns `approvalRequired = true`. When this happens, re-call with both `managerId` (hashed) and `comments` to submit an approval request.

## Workflow guidance

### Listing custodians

Use `get_custodians` with a hashed `investigationId`. Supports filtering by `criteria` (name/email match) and `status` (legal hold status enum). Paginate with `page` (default 1) and `pageSize` (default 50, max 200).

### Adding custodians

Use `add_custodians`. Requires `investigationId` and at least one custodian entry with:

- `account.name` (required)
- `account.email` or `account.emailAddress` (one required)
- `account.id` (optional, hashed)
- `roleId` (required, hashed)
- `managers[]` (optional, each with name and email)

If the response indicates `approvalRequired = true` and `success = false`, inform the user that approval is needed and ask for a manager and comments to submit the request.

### Releasing a custodian

Use `release_custodian` with `investigationId` and `custodianId`. Same approval workflow applies — if approval is required, prompt for `managerId` and `comments`.

Domain failure if the custodian is already released or deleted.

### Sending notifications

Use `send_notification` with `investigationId`. Optionally provide:

- `custodianIds` — target specific custodians (omit to notify all pending)
- `includeReviewer` — copy reviewer (default false)
- `includeManagers` — copy managers (default false)

### Endpoint holds

Use `apply_endpoint_hold` or `release_endpoint_hold` with `investigationId` and `endpointId`.

Constraints:
- Only `M365Custodian` endpoints are supported
- Apply: endpoint hold status must be `None` or `Released`
- Release: endpoint hold status must be `Active`

## Important

Legal hold operations are consequential — always confirm with the user before executing mutations. Summarize what will happen (who is being added/released, which endpoints affected) and wait for explicit confirmation.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
