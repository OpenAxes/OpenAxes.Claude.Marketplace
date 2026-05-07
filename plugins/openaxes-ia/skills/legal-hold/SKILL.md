---
name: legal-hold
description: >
  This skill should be used when the user asks about "custodians", "legal hold",
  "add custodians", "release custodian", "send notification", "hold notification",
  "endpoint hold", "custodian data source", "apply hold", "release hold", or any task involving managing
  legal hold custodians, notifications, custodian data sources, or endpoint preservation in OpenAxes IA.
metadata:
  version: "0.3.0"
---

# Legal Hold Management — OpenAxes IA

Manage legal hold operations through the MCP tools: `get_custodians`, `add_custodians`,
`release_custodian`, `send_notification`, `apply_endpoint_hold`, `release_endpoint_hold`,
`add_custodian_data_sources`, `apply_custodian_data_source_hold`, and
`release_custodian_data_source_hold`.

## Conventions

For `get_custodians`, `add_custodians`, `release_custodian`, `send_notification`, `add_custodian_data_sources`, `apply_custodian_data_source_hold`, and `release_custodian_data_source_hold`, the investigation reference accepts an exact investigation name or a **hashed ID**. Custodian references for notification/release and custodian datasource tools accept name, email address, or hashed custodian ID scoped to the resolved investigation. Legacy endpoint hold tools (`apply_endpoint_hold`, `release_endpoint_hold`) intentionally preserve their preexisting `investigationId` + `endpointId` hashed-ID contract.

Mutation tools that modify legal hold state use two-stage authorization. `tools/list` visibility means the caller may attempt the tool. Direct execution requires both investigation write access and the action-specific legal-hold permission. If direct execution is denied but the caller has `ApprovalRequest` and approval applies, supported tools return `approval_required` instead of failing. Re-call with both `approver` (approver name, email, or hashed user ID for an active tenant user with `ApprovalApprove`) and `comments` to submit an approval request. Approval is offered only when the tenant feature, approval workflow, and applicable change approval configuration are enabled.

## Workflow guidance

### Listing custodians

Use `get_custodians` with `investigation` set to the exact investigation name or hashed investigation ID. Supports filtering by `criteria` (name/email match) and `status` (legal hold status enum). Paginate with `page` (default 1) and `pageSize` (default 50, max 200). If an investigation name is ambiguous or not found, the server returns a resolution status with candidates or suggestions.

### Adding custodians

Use `add_custodians` with `investigation` set to the exact investigation name or hashed investigation ID and at least one custodian entry. Each custodian must include:

- `role` — investigation role name, abbreviation, or hashed role ID
- Either `account` — existing account name, email address, or hashed account ID — or both `name` and `email` for a new account
- Optional `managers[]`; each manager uses the same pattern: `account`, or both `name` and `email`

Existing account tokens are resolved by hashed ID, email, or account name. Ambiguous account names return candidates that include account name and email information; never choose silently.

If direct execution is denied and approval applies, `approval_required` identifies missing fields. Ask for the missing `approver` and/or `comments`; candidate approvers are active tenant users with `ApprovalApprove`. Then re-call `add_custodians` with those fields to submit the approval request.

### Releasing a custodian

Use `release_custodian` with `investigation` and `custodian`. Resolve the investigation first; then provide `custodian` as the custodian's exact name, email address, or hashed custodian ID scoped to that investigation. Ambiguous or unknown custodians return resolution statuses instead of domain failures.

Same approval workflow applies — if approval is required, prompt for `approver` and `comments`. Approvers are active tenant users with `ApprovalApprove`; do not route by legacy manager role.

Domain failure if the resolved custodian is already released or deleted.

### Sending notifications

Use `send_notification` with `investigation`. Optionally provide:

- `custodians` — target specific custodians by exact name, email address, or hashed custodian ID scoped to the investigation (omit to notify all pending)
- `includeReviewer` — copy reviewer (default false)
- `includeManagers` — copy managers (default false)

### Endpoint holds

Use `apply_endpoint_hold` or `release_endpoint_hold` with legacy hashed IDs: `investigationId` and `endpointId`. These tools are not conversational selector tools; use the custodian datasource tools for datasource-scoped preservation requests.

Constraints:
- Only `M365Custodian` endpoints are supported
- The endpoint must belong to the supplied investigation
- Apply: endpoint hold status must be `None` or `Released`
- Release: endpoint hold status must be `Active`

### Custodian datasource preservation tools

Use these MCDT capability tools when the user asks to manage existing custodians by source scope instead of by endpoint ID:

- `add_custodian_data_sources` — add or reuse `mailbox` and `onedrive` datasource endpoints for existing investigation custodians; direct execution returns explicit per-request source outcomes, while approval fallback preserves requested scopes.
- `apply_custodian_data_source_hold` — apply hold directly to existing custodian datasource scopes.
- `release_custodian_data_source_hold` — resolve selected custodian datasource scopes for release only when exact release can be honored; do not broaden a partial request.

Known limitations:
- These tools do not create missing custodians; use `add_custodians` first.
- `sites` and `teams` are explicitly deferred/unsupported for v1 datasource scope requests.
- Datasource hold apply/release direct execution is supported for resolved, supported source scopes when the exact request can be honored; report the structured selector, approval, fidelity, unsupported-scope, and no-widening outcomes returned by the tool.

## Important

Legal hold operations are consequential — always confirm with the user before executing mutations. Summarize what will happen (who is being added/released, which endpoints affected) and wait for explicit confirmation.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
