# Search and Review — Tool Parameter Reference

## run_search

| Field | Type | Required | Notes |
|---|---|---|---|
| `investigation` | string | Yes | Investigation name or hashed ID |
| `criteria` | string | No | Search expression |
| `search` | string | No | Saved-search name or hashed ID; defaults to first search or `0` |
| `start` | integer | No | Default `0`, must be >= 0 |
| `limit` | integer | No | Default `25`, range 1–100 |
| `sortBy` | string | No | `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance` |
| `ascending` | boolean | No | Default `false` |
| `applyTags` | array[string] | No | Tag names or hashed IDs to apply to all matching documents; additionally requires `InvestigationDataDocumentsTag` |

Returns: `total`, `start`, `limit`, `requestId`, `searchId`, `items[]`; when `applyTags` is supplied, also returns `taggedCount`.

Each item: `itemId`, `sourceInvestigationId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

Note: `itemId` format is `<hashed-document-investigation-id>.<hashed-document-id>`.

Tag constraints for `run_search.applyTags`: tags must be active, in the same tenant, and either global or owned by the target investigation. Investigation-scoped tags take precedence over same-name tenant/global tags, ambiguity is reported only within the winning scope, and not-found suggestions can include tags from either valid scope.
