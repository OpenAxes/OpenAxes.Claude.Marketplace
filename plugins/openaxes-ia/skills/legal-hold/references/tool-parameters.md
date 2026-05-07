# Legal Hold — Tool Parameter Reference

## get_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `criteria` | string | No | Matches custodian name or email |
| `status` | string | No | Exact legal hold status enum name |
| `page` | integer | No | Default `1`, must be >= 1 |
| `pageSize` | integer | No | Default `50`, range 1–200 |

Returns: `total`, `page`, `pageSize`, `custodians[]` each with `id`, `name`, `emailAddress`, `role`, `holdStatus`, `answerStatus`, `reviewStatus`.

## add_custodians

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `custodians` | array | Yes | At least one custodian entry |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID; must be an active tenant user with `ApprovalApprove` |

Each custodian entry:

| Field | Type | Required | Notes |
|---|---|---|---|
| `account` | string | Conditional | Existing account name, email address, or hashed account ID. Required unless both `name` and `email` are supplied. Ambiguous account names return candidates with email data. |
| `name` | string | Conditional | New custodian account display name. Required with `email` when `account` is not supplied. |
| `email` | string | Conditional | New custodian account email address. Required with `name` when `account` is not supplied. |
| `role` | string | Yes | Investigation role name, abbreviation, or hashed role ID |
| `managers` | array | No | Optional manager entries; each manager uses `account`, or both `name` and `email` |

Each manager entry:

| Field | Type | Required | Notes |
|---|---|---|---|
| `account` | string | Conditional | Existing manager account name, email address, or hashed account ID. Required unless both `name` and `email` are supplied. |
| `name` | string | Conditional | New manager account display name. Required with `email` when `account` is not supplied. |
| `email` | string | Conditional | New manager account email address. Required with `name` when `account` is not supplied. |

Returns: `success`, `approvalRequired`. Direct success returns status `success` with `success=true, approvalRequired=false`. If direct execution is denied and approval applies, missing `approver` and/or `comments` returns status `approval_required` with `success=false, approvalRequired=true`, `missingFields`, and candidate approvers when `approver` is missing. Supplying both fields submits the approval request and returns status `approval_required` with `success=true, approvalRequired=true`.

## release_custodian

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `custodian` | string | Yes | Custodian name, email address, or hashed custodian ID scoped to the investigation |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID; must be an active tenant user with `ApprovalApprove` |

Returns: same three modes as `add_custodians`; approval prompt/submission requires approval request permission when direct execution is denied. If direct execution is denied and approval applies, missing `approver`/`comments` returns `approval_required` with candidate approvers when `approver` is missing.

## send_notification

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `custodians` | array[string] | No | Custodian names, email addresses, or hashed custodian IDs scoped to the investigation; omit to target all pending |
| `includeReviewer` | boolean | No | Default `false` |
| `includeManagers` | boolean | No | Default `false` |

Returns: `success`, `sent`.

## apply_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `endpointId` | string | Yes | Hashed endpoint ID belonging to the investigation |

Returns: `success`, `endpointId`. This legacy endpoint-hold tool intentionally preserves its preexisting hashed-ID contract. Only M365Custodian endpoints; status must be None or Released.

## release_endpoint_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `endpointId` | string | Yes | Hashed endpoint ID belonging to the investigation |

Returns: `success`, `endpointId`. This legacy endpoint-hold tool intentionally preserves its preexisting hashed-ID contract. Only M365Custodian endpoints; status must be Active.

## add_custodian_data_sources

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `requests` | array | Yes | At least one request row for an existing investigation custodian |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID |

Each `requests[]` item:

| Field | Type | Required | Notes |
|---|---|---|---|
| `custodian` | string | Yes | Existing investigation custodian name, email address, or hashed custodian ID |
| `dataSources` | array[string] | Yes | Supported v1 values: `mailbox`, `onedrive`; `sites` and `teams` are explicitly deferred/unsupported |
| `applyHold` | boolean | No | Optional immediate-hold intent for hold-capable scopes |

Returns: MCDT selector-resolution data plus direct `requests[]` / `outcomes[]` rows showing requested scope, canonical scope, `added`/`reused`/`skipped`/`unsupported`/`capability_limited` outcome, and hashed endpoint/root references when materialized. Direct execution emits endpoint-change audit metadata with hashed custodian and endpoint references. Approval fallback returns the preserved requested/effective selectors and approval outcome. Existing endpoint-hold tools remain separate on their legacy hashed-ID contract.

## apply_custodian_data_source_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `requests` | array | Yes | At least one existing custodian/source request row |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID |

Each `requests[]` item:

| Field | Type | Required | Notes |
|---|---|---|---|
| `custodian` | string | Yes | Existing investigation custodian name, email address, or hashed custodian ID |
| `dataSources` | array[string] | Yes | Supported v1 values: `mailbox`, `onedrive`; `sites` and `teams` are explicitly deferred/unsupported |

Returns: resolved selector and approval-scaffold details for later custodian datasource hold. The tool must not report hold for unsupported scopes or widen the requested selection.

## release_custodian_data_source_hold

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Exact investigation name or hashed investigation ID |
| `requests` | array | Yes | At least one existing custodian/source request row |
| `comments` | string | No | Required when submitting an approval request |
| `approver` | string | No | Approval approver name, email address, or hashed user ID |

Each `requests[]` item:

| Field | Type | Required | Notes |
|---|---|---|---|
| `custodian` | string | Yes | Existing investigation custodian name, email address, or hashed custodian ID |
| `dataSources` | array[string] | Yes | Supported v1 values: `mailbox`, `onedrive`; `sites` and `teams` are explicitly deferred/unsupported |

Returns: exactness-preflight details and either direct release/approval submission for the exact requested datasource set or an explicit domain failure. Known limitation: partial release must fail explicitly unless exact requested-scope release can be honored; never broaden a partial request.
