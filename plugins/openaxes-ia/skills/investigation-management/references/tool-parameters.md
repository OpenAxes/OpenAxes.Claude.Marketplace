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
| `investigationId` | string | Yes | Hashed investigation ID |

Returns: `id`, `name`, `description`, `status`, `investigationType`, `fileDate`, `createdDate`, `createdBy`, `workspaces[]`, `reviewers[]`, `counts` (custodianCount, endpointCount, reviewerCount, workspaceCount, searchCount, exportCount, customFieldCount), `flags` (isLegalHold, hasIndexableEndpoints, hasAnalyzableEndpoints, isLinkedToPrr).

## get_investigation_templates

No arguments.

Returns: `templates[]` each with `id`, `name`, `description`.

## create_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `templateId` | string | Yes | Hashed template investigation ID |
| `name` | string | Yes | Investigation name |
| `description` | string | No | Investigation description |
| `fileDate` | string | Yes | ISO-8601 date/time |
| `workspaceId` | string | No | Optional hashed workspace ID |

Returns: `id` (hashed), `success`, `approvalRequired`.

## close_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |

Returns: `investigationId`, `status` ("Closed").

## reopen_investigation

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `resendNotification` | boolean | No | Default `false` |

Returns: `investigationId`, `status`, `resendNotification`.
