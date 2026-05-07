---
name: investigation-management
description: >
  This skill should be used when the user asks to "list investigations",
  "show my investigations", "create an investigation", "close an investigation",
  "reopen an investigation", "get investigation details", "check investigation status",
  "configure prefilter", "start deep analysis", or any task involving viewing, creating,
  configuring, analyzing, or managing the lifecycle of OpenAxes IA investigations.
metadata:
  version: "0.3.0"
---

# Investigation Management — OpenAxes IA

Manage the full lifecycle of investigations in OpenAxes IA through the MCP tools:
`list_investigations`, `get_investigation`, `get_investigation_templates`, `create_investigation`,
`close_investigation`, `reopen_investigation`, `configure_investigation_prefilter`, and
`start_deep_analysis`.

## Conventions

Investigation references accepted by read/status tools can be an exact investigation name or a **hashed ID** (opaque string), not an integer primary key. Template references for creation accept an exact template name or hashed ID. Workspace references for creation accept an exact workspace name, abbreviation, or hashed ID. Prefer hashed ID values returned by the server when available. If a reference is ambiguous, the server returns candidates for disambiguation.

Every tool response follows a standard contract with `structuredContent.status` indicating outcome. Check for these statuses:

- `success` — completed normally
- `validation_failure` — bad input; check `validationErrors` for field-level details
- `not_found` — entity not accessible or does not exist
- `resolution_ambiguous` — a name matched multiple tenant-visible entities; choose one candidate and retry
- `resolution_not_found` — a name or hashed ID did not resolve; use any suggestions to retry
- `authorization_failure` — caller lacks permission
- `domain_failure` — business rule prevented the operation

## Workflow guidance

### Listing investigations

Use `list_investigations`. Supports filtering by `criteria` (name/description match), `status` (`draft`, `active`, `closed`, `processing`, `deleted`), and `investigationType` (`normalcase`, `template`, `featureindex`). Paginate with `page` (default 1) and `pageSize` (default 25, max 100).

When the user asks vague questions like "show my investigations," start with active investigations and offer to filter further.

### Getting investigation details

Use `get_investigation` with `investigation` set to an exact investigation name or hashed ID. Returns full detail including workspaces, reviewers, counts (custodians, endpoints, searches, exports), and flags (legal hold, indexable/analyzable endpoints).

### Creating investigations

Always start by calling `get_investigation_templates` to show the user available templates. Then call `create_investigation` with:

- `template` (required) — template name or hashed ID from the templates list
- `name` (required)
- `fileDate` (required, ISO-8601)
- `description` (optional)
- `workspace` (optional) — workspace name, abbreviation, or hashed ID

Confirm the template choice and investigation name with the user before creating.

### Closing and reopening

Use `close_investigation` or `reopen_investigation` with `investigation` set to an exact investigation name or hashed ID. Reopening optionally accepts `resendNotification` (default false) to re-send legal hold notifications.

Always confirm closure/reopen with the user — these are significant state changes.

### Prefilter configuration scaffold

Use `configure_investigation_prefilter` when the user asks to enable, disable, or update the governed prefilter subset for an investigation. The supported input is intentionally narrow: `enabled`, `method`, `dateStart`, `dateEnd`, `expression`, `extensions`, `contentKinds`, and `purviewAdmin`. Do not use it as a general data-processing configuration editor.

Known limitations: prefilter is persisted at investigation scope; datasource-specific enforcement depends on downstream product/import support. Content kinds normalize to `email`, `tasks`, `meetings`, `microsoftteams`, and `docs` (`teams` may normalize to `microsoftteams`). If the response is `approval_required`, collect both `comments` and an `approver`; approval submissions return `directMutationPerformed=false` and store the normalized prefilter state for later approval execution.

### Deep-analysis start scaffold

Use `start_deep_analysis` when the user asks to start deep analysis for selected eligible targets. Supported target families are `targets.custodianDataSources[]` (custodian plus `mailbox`/`onedrive`) and `targets.endpoints[]` (endpoint friendly name or hashed endpoint ID). The tool must report requested, resolved, eligible, excluded, and started targets separately.

Known limitations: omitted whole-investigation starts and unsupported target shapes must be rejected explicitly; already-running/not-ready/zero-eligible cases are non-success/no-op outcomes. Approval-only callers must provide `approver` and `comments`; approval submissions preserve the exact selected eligible endpoint set and do not start work until approval execution. If exact selected-target fidelity cannot be preserved, the path fails explicitly rather than broadening.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
