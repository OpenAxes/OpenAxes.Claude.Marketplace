---
name: search-and-review
description: >
  This skill should be used when the user asks to "search documents",
  "find documents", "run a search", "tag documents", "apply tags",
  "review documents", "search investigation", or any task involving
  querying or tagging documents within an OpenAxes IA investigation.
metadata:
  version: "0.1.0"
---

# Search and Review — OpenAxes IA

Search investigation documents and apply review tags through the MCP tools:
`run_search`, `apply_tags`.

## Conventions

All entity identifiers are **hashed IDs**. Search results return a special `itemId` formatted as `<hashed-document-investigation-id>.<hashed-document-id>` — this composite ID is what `apply_tags` accepts when tagging specific items.

## Workflow guidance

### Running searches

Use `run_search` with a hashed `investigationId`. Key parameters:

- `criteria` — search expression (omit for all documents)
- `searchId` — hashed saved-search ID (defaults to first search or `0`)
- `start` — offset for pagination (default 0)
- `limit` — results per page (default 25, max 100)
- `sortBy` — one of: `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance`
- `ascending` — sort direction (default false)

Each result item includes: `itemId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

When the user asks to search, clarify the investigation context first. If they want to browse broadly, omit `criteria`. Present results in a readable summary (name, owner, date, type) and offer to paginate or refine.

### Tagging documents

Use `apply_tags` with `investigationId` and `tagId` (hashed). Choose exactly one mode:

1. **Explicit items** — provide `itemIds` array with `itemId` values from `run_search`
2. **Search-based** — provide `searchCriteria` (and optionally `searchId`) to tag all matching documents

Providing both `itemIds` and `searchCriteria`, or neither, results in a validation failure.

When tagging by search criteria, warn the user that this applies to ALL matching documents and confirm before executing. Returns `taggedCount` so the user knows how many documents were affected.

### Typical search-then-tag workflow

1. Run `run_search` to find relevant documents
2. Present results to the user
3. User selects which items to tag (or confirms tagging all search results)
4. Call `apply_tags` with either the selected `itemIds` or the `searchCriteria`

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
