# Legal Hold — Tool Parameter Reference

## get_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `criteria` | string | No | Matches custodian name or email |
| `status` | string | No | Exact legal hold status enum name |
| `page` | integer | No | Default `1`, must be >= 1 |
| `pageSize` | integer | No | Default `50`, range 1–200 |

Returns: `total`, `page`, `pageSize`, `custodians[]` each with `id`, `name`, `emailAddress`, `role`, `holdStatus`, `answerStatus`, `reviewStatus`.

## add_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `custodians` | array | Yes | At least one custodian entry |
| `comments` | string | No | Used when approval must be requested |
| `managerId` | string | No | Hashed manager user ID for approval routing |

Each custodian entry:

| Field | Type | Required | Notes |
|---|---|---|---|
| `account` | object | Yes | Must include `name` and `email`/`emailAddress`; `id` optional |
| `roleId` | string | Yes | Hashed role ID |
| `managers` | array | No | Each with `name` and `email`/`emailAddress`; `id` optional |

Returns: `success`, `approvalRequired`. Three modes: immediate update (success=true, approval=false), approval prompt (success=false, approval=true), approval submitted (success=true, approval=true).

## release_custodian

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `custodianId` | string | Yes | Hashed custodian ID |
| `comments` | string | No | Used when approval must be requested |
| `managerId` | string | No | Hashed manager user ID for approval routing |

Returns: same three modes as `add_custodians`.

## send_notification

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `custodianIds` | array[string] | No | Hashed custodian IDs; omit to target all pending |
| `includeReviewer` | boolean | No | Default `false` |
| `includeManagers` | boolean | No | Default `false` |

Returns: `success`, `sent`.

## apply_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `endpointId` | string | Yes | Hashed endpoint ID |

Returns: `success`, `endpointId`. Only M365Custodian endpoints; status must be None or Released.

## release_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `endpointId` | string | Yes | Hashed endpoint ID |

Returns: `success`, `endpointId`. Only M365Custodian endpoints; status must be Active.
