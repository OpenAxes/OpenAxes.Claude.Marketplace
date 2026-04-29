# Reports — Tool Parameter Reference

> **Statuses:** In addition to `success`, `validation_failure`, `not_found`, and `authorization_failure`, tools in this skill may return `resolution_ambiguous` (name matched multiple entities — `candidates` array supplied), `resolution_not_found` (no match — `suggestions` array supplied), and `approval_required`.

## list_reports

No arguments.

Returns: `reports[]` each with `id` (friendly), `name`, `description`, `type`, `isSavedReport`, `rawOutputFormats`, `scheduleFormats`, `sections[]`, `filters[]`, `savedFilters`, `schedule`.

Each section: `id` (hashed section identifier), `name`, `pageable`, `renderType`.
Each filter: `name`, `displayName`, `type`, `entity`, `required`, `multiple`, `description`.

## execute_report

| Field | Type | Required | Notes |
|---|---|---|---|
| `reportId` | string | Yes | Friendly report ID |
| `format` | string | Yes | `json` or `csv` |
| `sectionId` | string | No | Hashed section ID from `list_reports`; legacy integer IDs are still accepted for compatibility. Required for CSV when report has multiple sections |
| `timezoneOffset` | integer | No | Added to filter bag |
| `page` | integer | No | Must be > 0 when supplied |
| `pageSize` | integer | No | Must be > 0 when supplied |
| `sortBy` | string | No | Added to filter bag |
| `ascending` | boolean | No | Added to filter bag |
| `filters` | object | No | Report-specific filters |

JSON output returns: `reportId`, `reportName`, `format`, `sections[]` each with hashed `id`, `name`, `total`, `rowCount`, `columns`, `rows`.

CSV output returns: `reportId`, `reportName`, `format`, `section` with hashed `id`, `name`, `total`, `rowCount`, `content`.

Filter rules:
- Must be a JSON object
- Unknown filter names rejected
- Scalar values: string, boolean, number, or null
- Multi-value: arrays
- Date ranges: `{ "from": "...", "to": "..." }` or split as `filterFrom`/`filterTo`

## schedule_report

| Field | Type | Required | Notes |
|---|---|---|---|
| `reportId` | string | Yes | Friendly report ID |
| `schedule` | string | Yes | Quartz cron expression |
| `format` | string | No | `xlsx` or `pdf`; defaults based on report config |
| `name` | string | No | Auto-generated if omitted |
| `description` | string | No | Defaults from source report |
| `recipients` | array[string] | No | Currently limited to authenticated user's email |
| `filters` | object | No | Same rules as execute_report |

Returns: `success`, `scheduledReportId` (hashed), `reportId`, `name`, `frequency`, `format`, `recipient`, `nextRun`.

Note: `json` and `csv` are NOT valid scheduling formats. Only `xlsx` and `pdf`.
