# Legal Hold — Tool Parameter Reference

## get_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID returned by list_investigations |
| `criteria` | string | No | Optional name/email substring filter applied within the investigation |
| `status` | string | No | Exact legal hold status enum name |
| `page` | integer | No | Default `1`, must be >= 1 |
| `pageSize` | integer | No | Default `50`, range 1–200 |

Returns: `total`, `page`, `pageSize`, `custodians[]` each with `id`, `name`, `emailAddress`, `role`, `holdStatus`, `answerStatus`, `reviewStatus`.

## add_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID |
| `custodians` | array | Yes | At least one custodian entry |
| `approver` | string | No | User name or email of the approval routing manager; required if caller lacks direct write permission |
| `comments` | string | No | Required when `approver` is supplied — shown in the approval queue |

Each custodian entry:

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | Yes | Custodian display name |
| `email` | string | Yes | Custodian email; resolves existing Account row or creates one |
| `role` | string | Yes | Role name or abbreviation (case-insensitive exact match) |
| `managers` | array | No | Each with `name` (required) and `email` (required) |

Returns three modes:
- **Direct write** (`success=true`, `approvalRequired=false`) — custodians added immediately.
- **Approval prompt** (`approval_required` status, `success=false`) — one or more fields are missing. `missingFields` lists `"approver"` and/or `"comments"`. When `approver` is missing, `candidateApprovers` is also returned (up to 25 active managers).
- **Approval submitted** (`approval_required` status, `success=true`) — request dispatched; `approver` object is echoed back.

## release_custodian

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID returned by list_investigations |
| `custodian` | string | Yes | Custodian name or email (case-insensitive exact match within the investigation), or hashed ID |
| `approver` | string | No | User name or email of the approval routing manager; required if caller lacks direct write permission |
| `comments` | string | No | Required when `approver` is supplied — shown in the approval queue |

Returns three modes:
- **Direct write** (`success=true`, `approvalRequired=false`) — custodian released immediately.
- **Approval prompt** (`approval_required` status, `success=false`) — one or more fields are missing. `missingFields` lists `"approver"` and/or `"comments"`. When `approver` is missing, `candidateApprovers` is also returned (up to 25 active managers).
- **Approval submitted** (`approval_required` status, `success=true`) — request dispatched; `approver` object is echoed back.

## send_notification

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID returned by list_investigations |
| `custodians` | array[string] | No | Names or emails of custodians to notify, scoped to the investigation; omit to target all pending custodians |
| `includeReviewer` | boolean | No | Default `false` |
| `includeManagers` | boolean | No | Default `false` |

Returns: `success`, `sent`.

## apply_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID returned by list_investigations |
| `endpoint` | string | Yes | Endpoint friendly name (case-insensitive exact match within the investigation), or hashed ID |

Returns: `success`, `endpointId`. Only M365Custodian endpoints; status must be None or Released.

## release_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID returned by list_investigations |
| `endpoint` | string | Yes | Endpoint friendly name (case-insensitive exact match within the investigation), or hashed ID |

Returns: `success`, `endpointId`. Only M365Custodian endpoints; status must be Active.
