# Investigation Management — Tool Parameter Reference

## list_investigations

| Field | Type | Required | Notes |
|---|---|---|---|
| `criteria` | string | No | Matches investigation name or description |
| `status` | string | No | `draft`, `active`, `closed`, `processing`, `deleted`, or exact enum name |
| `investigationType` | string | No | `normalcase`, `template`, `featureindex`, or exact enum name |
| `page` | integer | No | Default `1`, must be >= 1 |
| `pageSize` | integer | No | Default `25`, range 1–100 |

Returns: `total`, `page`, `pageSize`, `investigations[]` where each investigation has `id`, `name`, `description`, `status`, `investigationType`, `createdDate`, `fileDate`, `isLegalHold`, `hasDataProcessing`, `hasIndexableEndpoints`, `hasAnalyzableEndpoints`, `workspaceCount`, `reviewerCount`.

## get_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name or hashed ID. Exact names are resolved case-insensitively; ambiguous names return `resolution_ambiguous`, and unknown names can return `resolution_not_found` with suggestions. |

Returns: `id`, `name`, `description`, `status`, `investigationType`, `fileDate`, `createdDate`, `createdBy`, `workspaces[]`, `reviewers[]`, `counts` (custodianCount, endpointCount, reviewerCount, workspaceCount, searchCount, exportCount, customFieldCount), `flags` (isLegalHold, hasIndexableEndpoints, hasAnalyzableEndpoints, isLinkedToPrr).

## get_investigation_templates

No arguments.

Returns: `templates[]` each with `id`, `name`, `description`.

## create_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `template` | string | Yes | Investigation template name or hashed ID. Exact names are resolved case-insensitively; ambiguous names return `resolution_ambiguous`, and unknown names can return `resolution_not_found` with suggestions. |
| `name` | string | Yes | Investigation name |
| `description` | string | No | Investigation description |
| `fileDate` | string | Yes | ISO-8601 date/time |
| `workspace` | string | No | Optional workspace name, abbreviation, or hashed ID. Ambiguous workspace references return `resolution_ambiguous`. |

Returns: `id` (hashed), `success`, `approvalRequired`.

## close_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name or hashed ID. Exact names are resolved case-insensitively; ambiguous names return `resolution_ambiguous`, and unknown names can return `resolution_not_found` with suggestions. |

Returns: `investigationId`, `status` ("Closed").

## reopen_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name or hashed ID. Exact names are resolved case-insensitively; ambiguous names return `resolution_ambiguous`, and unknown names can return `resolution_not_found` with suggestions. |
| `resendNotification` | boolean | No | Default `false` |

Returns: `investigationId`, `status`, `resendNotification`.

## configure_investigation_prefilter

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `prefilter` | object | Yes | Narrow prefilter subset only; unrelated `DataConfiguration` fields are out of scope |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID |

Supported `prefilter` fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| `enabled` | boolean | No | Enables or disables prefilter; disabling follows existing reset semantics |
| `method` | string | No | `legacy` or `purview` |
| `dateStart` | string | No | Date value; must be <= `dateEnd` when both are supplied |
| `dateEnd` | string | No | Date value; persisted with existing end-of-day normalization semantics |
| `expression` | string | No | Prefilter expression |
| `extensions` | array[string] | No | Extension filters |
| `contentKinds` | array[string] | No | Canonical values: `email`, `tasks`, `meetings`, `microsoftteams`, `docs`; friendly `teams` normalizes to `microsoftteams` where accepted |
| `purviewAdmin` | string | No | Configured tenant Purview admin email when Purview method requires it |

Returns: effective normalized prefilter state, resolved investigation/admin details, `executionMode`, `approvalRequired`, `directMutationPerformed`, and explicit validation/supportability failures. Approval submissions capture the normalized prefilter state and do not mutate configuration until approval execution. Known limitation: configuration is investigation-scoped; datasource-specific enforcement is not universally claimed.

## start_deep_analysis

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `targets` | object | Yes | Supported selected-target families only |
| `comments` | string | No | Required when submitting an approval request if approval is supported for the exact target set |
| `approver` | string | No | Approval approver name, email address, or hashed user ID |

Supported `targets` fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| `custodianDataSources` | array | Conditional | Each item has `custodian` plus `dataSources`; at least one supported target family is required |
| `endpoints` | array[string] | Conditional | Endpoint friendly names or hashed endpoint IDs scoped to the investigation |

Each `targets.custodianDataSources[]` item:

| Field | Type | Required | Notes |
|---|---|---|---|
| `custodian` | string | Yes | Existing investigation custodian name, email address, or hashed custodian ID |
| `dataSources` | array[string] | Yes | Supported v1 values: `mailbox`, `onedrive` |

Returns: requested selectors, resolved endpoints, eligible targets, excluded/ineligible targets, start result, diagnostics/audit summaries, and execution mode. Approval-only callers receive `approval_required` until `approver` and `comments` are supplied; complete approval submissions store the exact selected eligible endpoint set and do not start work until approval is approved. Known limitations: omitted whole-investigation starts and unsupported target shapes are rejected; already-running, not-ready, zero-eligible, underlying start failures, or approval payloads that cannot preserve exact selected-target fidelity return explicit non-success/no-op outcomes.
