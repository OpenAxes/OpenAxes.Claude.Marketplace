# Search and Review — Tool Parameter Reference

> **Statuses:** In addition to `success`, `validation_failure`, `not_found`, and `authorization_failure`, tools in this skill may return `resolution_ambiguous` (name matched multiple entities — `candidates` array supplied), `resolution_not_found` (no match — `suggestions` array supplied), and `approval_required`.

## run_search

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name (case-insensitive exact match) or hashed ID from `list_investigations` |
| `criteria` | string | No | Search expression; omit to return all documents |
| `search` | string | No | Saved-search name (case-insensitive) or hashed ID; defaults to investigation's first saved search |
| `start` | integer | No | Default `0`, must be >= 0 |
| `limit` | integer | No | Default `25`, range 1–100 |
| `sortBy` | string | No | `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance` |
| `ascending` | boolean | No | Default `false` |
| `applyTags` | array[string] | No | Tag names (investigation-scoped first, then global) or hashed IDs. When supplied, tags ALL matching documents across all pages and adds `taggedCount` to the response. All tags are resolved before the search executes; ambiguous or unknown tags return early. |

Returns: `total`, `start`, `limit`, `requestId`, `searchId`, `items[]`, `taggedCount` (present only when `applyTags` is supplied).

Each item: `itemId`, `sourceInvestigationId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

Note: `itemId` format is `<hashed-document-investigation-id>.<hashed-document-id>`.

