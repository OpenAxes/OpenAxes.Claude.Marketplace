---
name: reports
description: >
  This skill should be used when the user asks to "run a report", "list reports",
  "generate a report", "schedule a report", "export report data", "get report data",
  or any task involving listing, executing, or scheduling reports in OpenAxes IA.
metadata:
  version: "0.1.0"
---

# Reports — OpenAxes IA

List, execute, and schedule reports through the MCP tools:
`list_reports`, `execute_report`, `schedule_report`.

## Conventions

Report identifiers use **friendly IDs** (derived from report name), NOT hashed integer IDs. This is different from all other IA entities.

## Workflow guidance

### Listing reports

Use `list_reports` (no arguments). Returns all accessible reports with their definitions, sections, filters, formats, and schedule info. Use this to discover what reports are available before executing or scheduling.

Each report definition includes:

- Supported `rawOutputFormats` (for execution: `json`, `csv`)
- Supported `scheduleFormats` (for scheduling: typically `xlsx`, `pdf`)
- `sections[]` with names and render types
- `filters[]` with names, types, and whether required

### Executing reports

Use `execute_report` with `reportId` (friendly) and `format` (`json` or `csv`).

Key considerations:

- If the report has multiple sections and format is `csv`, exactly one `sectionId` must be specified
- Filters are report-specific — check `list_reports` to see available filters
- Date-range filters accept either `{ "from": "...", "to": "..." }` as a single object, or split as `filterNameFrom` / `filterNameTo`
- Unknown filter names are rejected

When the user asks to run a report, first call `list_reports` to find the right one and understand its filters. Present filter options to the user if the report has required filters.

For `json` output, results include structured `columns` and `rows` per section — ideal for analysis, summarization, or document creation.

For `csv` output, results include raw `content` text — useful for export.

### Scheduling reports

Use `schedule_report` with:

- `reportId` (friendly, required)
- `schedule` (Quartz cron expression, required)
- `format` (`xlsx` or `pdf`; `json`/`csv` are NOT valid for scheduling)
- `name`, `description` (optional)
- `recipients` (optional, but currently limited to the authenticated user's email)
- `filters` (optional, same rules as `execute_report`)

Common Quartz cron examples:

- `0 0 8 ? * MON-FRI` — weekdays at 8 AM
- `0 0 9 ? * MON` — every Monday at 9 AM
- `0 0 0 1 * ?` — first of every month at midnight

Returns the `scheduledReportId` and `nextRun` timestamp on success.

### Creating documents from report data

When the user asks to create a document (Word, Excel, PDF, presentation) from report data:

1. Execute the report in `json` format to get structured data
2. Use the appropriate Cowork document skill (docx, xlsx, pptx, pdf) to create the output
3. Transform the report rows/columns into the document format

This is a key integration point — the user can say things like "run the investigation summary report and create a Word document from it."

## Reference

Read `references/tool-parameters.md` for complete field-level parameter documentation.
