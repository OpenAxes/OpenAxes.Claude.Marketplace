---
name: documentation-qa
description: Answer questions from the user's locally indexed documentation (OpenAxes apps Radar, PuRe, IA and any other docs served by OAassist). Use whenever the user asks how something works in their projects, about specifications or requirements, status/progress/timelines from their documents, or technical details from manuals and guides -- any information that might live in their indexed documents.
---

# Documentation Q&A with OAassist

OAassist is a local search service over the user's indexed documentation. It
exposes two MCP tools: `ask_documentation` (search -- the primary tool) and
`read_document` (read one section verbatim -- the rare escalation). This is a
Q&A service, not document browsing: answer from search results, never dump
documents.

## ask_documentation -- the primary tool

This is a SEARCH ENGINE over a vector index, not a chat endpoint: it matches
your query against document chunks by semantic similarity. Phrase the query
the way the answer would appear in a document, not the way a user asks a
question.

Use it proactively when the user asks about:

- How something works in their projects
- Project specifications or requirements
- Status, progress, or timelines from their documents
- Technical details from manuals or guides
- Any information that might be in their personal documents

### How to phrase the query

Use search terms, not a conversational sentence. Keep the key nouns, entities,
product names, identifiers and dates from the request, and drop filler and
question words ("how", "can you", "what is", "?"). Prefer the document's own
vocabulary over the user's phrasing.

- Good: `Radar changelog April 2026 Medline signature detection`
- Bad: `Can you tell me what changed in Radar last April?`

If the first search misses, retry with different or broader terms -- refining
the search is the right instinct, not switching to read_document.

### The app filter

Pass `app` to restrict the search to one app's documentation: `"radar"`,
`"pure"`, or `"ia"`. Use it when the user's question is clearly about one app;
omit it for an unfiltered search across everything.

### Reading the result

The tool returns the relevant information followed by a `Sources:` list
(file name and section). Cite the sources when answering. The source file
names are also the input to `read_document` if escalation is ever needed.

## read_document -- the rare escalation

`ask_documentation` answers almost every question by itself, including factual
lookups. Reach for `read_document` ONLY to assemble a complete multi-step
PROCEDURE or list that search returned partially (e.g. an install guide whose
steps came back cut off or out of order), after `ask_documentation` already
located the document.

Do NOT use it to hunt for a single fact, value, port, name or setting: if a
fact is not in the search results, refine the search or say it is not
documented -- never read sections looking for it.

- `filename` comes from the Sources list of a previous `ask_documentation`
  call (e.g. `Radar_Installation_Guide.md`).
- `section` is matched by header substring (e.g. `"Phase 3"`).
- Omitting `section` returns only the document's outline -- it never returns
  the whole document body. Use the outline to pick the one section you need.

## Parameter reference

Exact types, optionality and semantics for both tools:
[references/tool-parameters.md](references/tool-parameters.md)
