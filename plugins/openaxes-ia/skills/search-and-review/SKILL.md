---
name: search-and-review
description: >
  This skill should be used when the user asks to "search documents",
  "find documents", "run a search", "tag documents", "apply tags",
  "review documents", "search investigation", or any task involving
  querying or tagging documents within an OpenAxes IA investigation.
metadata:
  version: "0.3.0"
---

# Search and Review — OpenAxes IA

Search investigation documents and apply review tags through the MCP tool:
`run_search`.

## Conventions

`run_search` accepts conversational investigation, saved-search, and `applyTags` tag tokens by exact name or hashed ID. If a friendly name is ambiguous, the tool returns `resolution_ambiguous`; if it cannot be resolved, it returns `resolution_not_found` with suggestions when available. Search results return a special `itemId` formatted as `<hashed-document-investigation-id>.<hashed-document-id>` for document-reference display and downstream non-MCP review workflows.

## Workflow guidance

### Running searches

Use `run_search` with an `investigation` name or hashed ID. Key parameters:

- `criteria` — search expression (omit for all documents)
- `search` — saved-search name or hashed ID (defaults to first search or `0`)
- `start` — offset for pagination (default 0)
- `limit` — results per page (default 25, max 100)
- `sortBy` — one of: `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance`
- `ascending` — sort direction (default false)
- `applyTags` — optional array of tag names or hashed IDs to apply to **all matching documents**, not just the returned page; requires document-tag permission and returns `taggedCount`

Each result item includes: `itemId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

When the user asks to search, clarify the investigation context first. If they want to browse broadly, omit `criteria`. Present results in a readable summary (name, owner, date, type) and offer to paginate or refine.

### Tagging documents

Use `run_search.applyTags`: include `applyTags` with tag names or hashed IDs when running `run_search`. Tags are resolved before search executes; investigation-scoped tags win over tenant/global tags with the same name. This applies each tag to all matching documents and returns `taggedCount` (including `0` when there are no matches).

When tagging, warn the user that `applyTags` applies to ALL documents matching the search criteria, not just the returned page, and confirm before executing. Returns `taggedCount` so the user knows how many documents were affected.

### Typical search-then-tag workflow

1. Run `run_search` to identify the candidate result set.
2. Present a summary of the returned page and the total matching count.
3. If the user wants tags applied, refine the search criteria or saved search until the intended set is exactly the set of all matching documents.
4. Warn and confirm that `applyTags` tags every matching document, not individual result choices and not only the returned page.
5. Rerun `run_search` with `applyTags` only after that confirmation.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
