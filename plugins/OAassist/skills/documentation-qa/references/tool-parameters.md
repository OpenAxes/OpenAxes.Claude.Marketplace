# OAassist MCP tool parameters

## ask_documentation

Search the indexed documentation by semantic similarity. Read-only.

| Parameter | Type | Required | Semantics |
|---|---|---|---|
| `query` | string | yes | Search terms, not a conversational sentence. Key nouns, entities, product names, identifiers, dates; drop filler and question words. Prefer the document's own vocabulary over the user's phrasing. If a search misses, retry with different or broader terms. |
| `app` | string or null | no | Restricts the search to one app's documentation. Known values: `"radar"`, `"pure"`, `"ia"` (the first directory segment under the service's documentation root). Omit for an unfiltered search. |
| `version` | string or null | no | Further restricts the search to one version of that app's documentation (e.g. `"1.4.0"`). Omit for all versions. |

Returns: a text block with the relevant information found, followed by a
`Sources:` list of `<file name> -- <section>` entries. If nothing relevant is
indexed, returns the no-results answer with no sources.

## read_document

Read one section of an indexed document in full, verbatim and in order.
Read-only. Escalation tool only -- see SKILL.md for when (and when not) to use
it.

| Parameter | Type | Required | Semantics |
|---|---|---|---|
| `filename` | string | yes | Document file name taken from a previous `ask_documentation` Sources list (e.g. `Radar_Installation_Guide.md`). Not a path -- just the file name. |
| `section` | string or null | no | Header of the section to read, matched by header substring (e.g. `"Phase 3"` matches the header `Phase 3: Configuration`). Omit to get only the document's outline; the tool never returns the whole document body. |

Returns: the section text verbatim, or the document outline when `section` is
omitted.
