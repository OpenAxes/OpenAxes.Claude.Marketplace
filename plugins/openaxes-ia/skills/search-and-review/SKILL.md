---
name: search-and-review
description: >
  This skill should be used when the user asks to "search documents",
  "find documents", "run a search", "tag documents", "apply tags",
  "review documents", "search investigation", or any task involving
  querying or tagging documents within an OpenAxes IA investigation.
metadata:
  version: "0.2.0"
---

# Search and Review — OpenAxes IA

Search investigation documents and apply review tags through the MCP tools:
`run_search`.

## Conventions

Entity references in this skill accept **either a friendly identifier (name, email, or abbreviation) or a hashed ID**. The server resolves the value, scoped to the caller's tenant. Resolution is case-insensitive exact match.

When a name matches multiple entities, the tool returns `resolution_ambiguous` with a `candidates` array — present the choices to the user and re-call with one of the returned hashed IDs.

When a name matches no entity, the tool returns `resolution_not_found` with a `suggestions` array of close matches.

Mutating tools that require approval return `approval_required`. If `approver` and/or `comments` are missing, the response includes `missingFields` indicating what to ask the user for, plus `candidateApprovers` listing users who can approve.

Statuses to handle: `success`, `validation_failure`, `not_found`, `authorization_failure`, `domain_failure`, `resolution_ambiguous`, `resolution_not_found`, `approval_required`.

## Workflow guidance

### Running searches

Use `run_search` with an `investigation` name or hashed ID. Key parameters:

- `investigation` — investigation name (case-insensitive exact match) or hashed ID (required)
- `criteria` — search expression (omit for all documents)
- `search` — saved-search name (case-insensitive) or hashed ID; defaults to first search
- `start` — offset for pagination (default 0)
- `limit` — results per page (default 25, max 100)
- `sortBy` — one of: `from`, `subject`, `recipients`, `hasattachment`, `maildate`, `documentdate`, `relevance`
- `ascending` — sort direction (default false)
- `applyTags` — optional array of tag names or hashed IDs; when supplied, tags ALL matching documents and includes `taggedCount` in the response

Each result item includes: `itemId`, `name`, `documentDate`, `owner`, `ownerEmail`, `mediaType`, `extension`, `sizeMB`, `relevance`, `hasAttachments`, `recipients`, `previewable`.

When the user asks to search, clarify the investigation context first. If they want to browse broadly, omit `criteria`. Present results in a readable summary (name, owner, date, type) and offer to paginate or refine.

### Tagging search results

Tagging is now part of `run_search`. Pass `applyTags: ["Hot", "Privileged"]` to apply those tags to every document the search matches. Tag names follow standard resolution: investigation-scoped tags win over global tags of the same name; ambiguous names within the same scope return `resolution_ambiguous`.

Response includes `taggedCount` = total documents tagged across all listed tags. When the search returns zero documents, `taggedCount` is `0` and the call still succeeds.

When tagging via `applyTags`, warn the user that this applies to ALL matching documents (not just the current page) and confirm before executing.

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
