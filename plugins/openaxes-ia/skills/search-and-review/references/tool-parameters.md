# Search and Review — Tool Parameter Reference

## run_search

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `criteria` | string | No | Search expression |
| `searchId` | string | No | Hashed saved-search ID; defaults to first search or `0` |
| `start` | integer | No | Default `0`, must be >= 0 |
| `limit` | integer | No | Default `25`, range 1–100 |
| `sortBy` | string | No | `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance` |
| `ascending` | boolean | No | Default `false` |

Returns: `total`, `start`, `limit`, `requestId`, `searchId`, `items[]`.

Each item: `itemId`, `sourceInvestigationId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

Note: `itemId` format is `<hashed-document-investigation-id>.<hashed-document-id>`.

## apply_tags

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigationId` | string | Yes | Hashed investigation ID |
| `tagId` | string | Yes | Hashed tag ID |
| `itemIds` | array[string] | No | Explicit item IDs from `run_search` |
| `searchCriteria` | string | No | Search query to resolve documents |
| `searchId` | string | No | Hashed saved-search ID (search mode only) |

Mode rule: exactly one of `itemIds` or `searchCriteria` must be supplied.

Returns: `success`, `taggedCount`, `tagId`.

Tag constraints: must be active, in the same tenant, and either global or owned by the target investigation.
