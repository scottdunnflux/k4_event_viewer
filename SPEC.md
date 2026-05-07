# K4 Event Viewer — Product Specification

## Overview

A single-file, client-side HTML/JS web app that lets K4 administrators upload `k4_event*.log` files from a K4 diagnostics package, view the parsed events in a filterable/sortable table, and export a customized view as CSV. Hosted on GitHub Pages with no backend.

## Log Format

Each line follows the logback pattern defined in `K4_Event_Log_Config.logback`. Fields are **fixed-width, bracket-delimited**:

```
%date{ISO8601} %date{z} [thread(25)] [objectType(20)] [action(20)] [publicationName(20)] [publicationID(15)] [sessionID(25)] [userName(20)] [userID(15)] [clientType(15)] [objectName(50)] [objectID(15)] data
```

### Parsed Columns

| # | Field | Width | Alignment | Notes |
|---|-------|-------|-----------|-------|
| 1 | **date** | — | — | ISO8601 timestamp: `2026-04-29 10:46:39,638` |
| 2 | **timezone** | — | — | `EST` or `EDT` (varies by DST) |
| 3 | **thread** | 25 | left | Java thread name, truncated at 25 chars |
| 4 | **objectType** | 20 | left | `USER`, `OBJECT_VARIANT`, `OBJECT`, `ISSUES`, `QUERY`, `SECURITY_CONFIG` |
| 5 | **action** | 20 | left | `LOGIN`, `LOGOUT`, `ACCEPT_TASK`, `FINISH_TASK`, `SAVE_VERSION`, `VIEW`, `PLACE`, `UNPLACE`, `START_WORKFLOW`, `MANAGE_TASK`, `UNDO_ACCEPT_TASK`, `ASSIGNMENT_CHANGED`, `EDIT`, `EDIT_AUTO_SWITCH`, `DETACH` |
| 6 | **publicationName** | 20 | right | e.g. `CRM`, `CROH` |
| 7 | **publicationID** | 15 | right | Numeric ID |
| 8 | **sessionID** | 25 | right | Hex session token |
| 9 | **userName** | 20 | left | `Last, First` format; `SYSTEM_USER` for automated; `Administrator` for admin |
| 10 | **userID** | 15 | left | Numeric |
| 11 | **clientType** | 15 | left | `K4Edit`, `K4Layout`, `K4Overview`, `K4Admin`, `K4ContentPortal`, or blank |
| 12 | **objectName** | 50 | left | Truncated at 50 chars |
| 13 | **objectID** | 15 | left | Numeric |
| 14 | **data** | unbounded | — | Free-form: XML payloads (`<mail>`, `<logIn>`, `<psFileIDs>`, `<relatedObject>`), HTML comments (`<!-- Accept Task: '...' -->`), or empty |

### Parsing Considerations

- Multiple log files can be loaded (`k4_event.log`, `k4_event-1.log` through `k4_event-10.log`). They must be **merged and sorted by timestamp**.
- The rolling policy means `k4_event-1.log` is _older_ than `k4_event.log`.
- Lines are single-line records (no multi-line spans observed in sample data).
- Field widths are enforced by logback — parsing can rely on fixed character positions within the bracketed sections, or split on `] [` boundaries.
- The `data` column often contains XML with angle brackets that would need HTML-escaping for table display.
- The sample data has ~25K lines across two files (~14.5 MB total). The app should handle this volume without freezing.

---

## User Flow

### 1. Upload

- User opens the page (no login, no backend).
- Drag-and-drop zone + file picker button, accepting multiple `k4_event*.log` files.
- Files are read client-side via `FileReader`, parsed, merged (deduplicating), and sorted by timestamp.
- Uploading additional files later merges with already-loaded data. A "Reset" button clears everything.
- Show a status line: file count, total events loaded, date range covered.

### 2. Table View

- Display all parsed events in a scrollable data table.
- **Default visible columns**: date, objectType, action, publicationName, userName, clientType, objectName, data.
- **Default hidden columns**: timezone, thread, publicationID, sessionID, userID, objectID.

### 3. Column Controls

- Toggle individual column visibility via grouped checkboxes (following k4pub2list pattern).
- "Toggle All" master checkbox.
- No column reordering in v1 — show/hide is sufficient.

### 4. Filtering

Priority filters (always visible in a filter bar above the table):

| Filter | Type | Behavior |
|--------|------|----------|
| **Date range** | Date picker (from/to) | Inclusive, based on date column |
| **Time range** | Time picker (from/to) | Within each day, or absolute time window |
| **userName** | Multi-select dropdown | Auto-populated from loaded data |
| **clientType** | Multi-select dropdown | Auto-populated from loaded data |
| **objectType** | Multi-select dropdown | Auto-populated from loaded data |
| **action** | Multi-select dropdown | Auto-populated from loaded data |
| **publicationName** | Multi-select dropdown | Auto-populated from loaded data |
| **Free text search** | Text input | Searches across all visible columns |

Filters are AND-combined. Dropdowns show only values present in the data. A "Clear filters" button resets all.

Display a count of matching rows vs total rows (e.g. "Showing 1,234 of 25,265 events").

### 5. Sorting

- Click a column header to sort ascending; click again for descending; third click removes sort.
- Visual indicator (arrow) on sorted column.
- Shift+click for multi-column sorting (secondary, tertiary, etc.) — matching k4pub2list behavior.

### 6. CSV Export

- "Export CSV" button exports the **currently filtered and sorted** view.
- Only includes currently visible columns, in their current order.
- Properly escapes commas, quotes, and newlines in field values.
- Triggers browser download as `k4_events_export.csv`.

---

## UI Design

### Layout

Follow the established pattern from k4pub2list — clean, functional, no unnecessary chrome:

- **Header**: Title ("K4 Event Viewer"), attribution ("Flux Consulting, Inc."), help button.
- **Upload area**: Prominent drag-drop zone with file input (collapses after files are loaded, but remains accessible to add more files). "Reset" button to clear all loaded data and start over.
- **Filter bar**: Horizontal row of filter controls above the table.
- **Column toggles**: Collapsible section between filters and table.
- **Data table**: Full-width, horizontally scrollable, with alternating row shading.
- **Footer**: Row count, export button.

### Style

- **No CSS framework** (matching k4pub2list approach): custom CSS, clean and readable.
- Responsive down to ~768px, though this is primarily a desktop tool.
- Light background, readable table fonts, color-coded status messages.

---

## Technical Approach

- **Single HTML file**: all CSS and JS embedded. No external dependencies, no CDN libraries, no build step.
- **Vanilla JavaScript**: no frameworks.
- **Parsing strategy**: split each line on the `] [` pattern, then trim brackets and whitespace. The first field (before the first `[`) contains date + timezone; the last field (after the last `]`) contains data. Fixed-width within brackets provides a fallback if field content contains `] [`.

### Virtual Scrolling

Render only the visible rows plus a **1,000-row buffer** above and below the viewport. The full filtered/sorted dataset lives in a JS array in memory; only the visible window is rendered to the DOM. As the user scrolls, rows are recycled — old rows removed, new rows inserted.

- Use a container with a calculated total height (row count x row height) to maintain accurate scrollbar behavior.
- Row height should be fixed (truncated data column helps here) for simple offset math.
- On filter/sort changes, recompute the filtered array, reset scroll to top, re-render the visible window.

### CSV Export

Export operates on the **full filtered/sorted dataset in memory**, not the rendered DOM. Iterate the JS array directly and build the CSV string. This means all matching rows are exported regardless of what's currently in the scroll viewport.

### File Merge & Reset

- Uploading additional files **merges** with already-loaded data, deduplicating by full line content (or timestamp + all fields).
- A **"Reset / Clear All"** button discards all loaded data, resets filters, and returns to the initial upload state.

### Timezone

- **Default display**: original log timezone (EST/EDT) as-is.
- **Toggle**: a checkbox or switch to convert all displayed timestamps to the user's local browser timezone. The underlying data stays in original timezone; conversion is display-only. Filter date/time pickers operate in whichever timezone is currently displayed.

---

## Data Column: Smart Parsing

The `data` field is truncated in the table (first ~80 characters) with an **expand-on-click** action that opens a detail panel or modal. The detail view **dynamically parses the XML/comment content** and presents it in a readable, structured format.

### XML Data Types (observed in sample data)

| Root Element | Frequency | Key Fields to Extract | Summary Display |
|---|---|---|---|
| `<mail>` | ~540 | `recipient`, `subject` | "Mail to {recipient}: {subject}" |
| `<logIn>` | ~524 | `clientIP`, `clientVersionAndBuild` | "Login from {clientIP}, v{version}" |
| `<psFileIDs>` | ~3,037 | `oldPSFileID`, `newPSFileID` | "File version {old} → {new}" |
| `<relatedObject>` | ~220 | `type`, `id`, `name`, `role` | "{type} {name} (ID:{id}, role:{role})" |
| `<publicationImportExport>` | ~5 | `oldXML`, `newXML` (contain nested escaped XML) | "Config change: {nested root element}" |
| `<queryXML>` | ~1 | `oldXML`, `newXML` (contain nested escaped query XML) | "Query definition change" |

### HTML Comment Patterns

| Pattern | Frequency | Summary Display |
|---|---|---|
| `<!-- Accept Task: '{taskName}' -->` | ~3,231 | "Accept: {taskName}" |
| `<!-- Undo Accept Task(s): ... -->` | ~215 | "Undo accept: {taskName}" |
| `<!-- Changed '{taskName}' [{status}, {user}] to: {newStatus}, {newUser} -->` | ~30 | "Task change: {taskName} → {newStatus}" |

### Detail View Behavior

- **Truncated cell**: shows first ~80 chars, styled as muted/monospace. Click to expand.
- **Expanded view** (modal or inline panel):
  - For recognized XML: render parsed fields as a clean key-value list. For `<mail>`, show recipient, subject, and body (with the body's `&#10;` decoded to line breaks). For `<psFileIDs>`, show old → new with labels. For nested XML (`publicationImportExport`, `queryXML`), unescape and pretty-print the inner XML.
  - For HTML comments: extract and display the task name and state change clearly.
  - For unrecognized/empty: show the raw text, monospace-formatted.

---

## Text Object Grouping (Collapse Mode)

### The Problem

A single user action on an article generates one log line per child text object. For example, one ACCEPT_TASK on "Kitchen Essentials" produces 35 lines — one for the article and 34 for its child text frames (product names, scores, body text, photos). This makes the raw log overwhelming: ~79% of lines are child text objects that obscure the actual user actions.

**Sample burst** (same user, same session, same action, same second):
```
10:48:52,535  ACCEPT_TASK  Kitchen Essentials          (article)
10:48:52,535  ACCEPT_TASK  Hand mixers                 (child text object)
10:48:52,535  ACCEPT_TASK  ! Breville Handy Mix...     (child text object)
10:48:52,535  ACCEPT_TASK  overall  Score               (child text object)
10:48:52,536  ACCEPT_TASK  76                           (child text object)
... 30 more lines ...
```

### Grouping Logic

Group consecutive lines that share all of:
- Same **sessionID**
- Same **action**
- Timestamps within a **configurable time window** of each other

**Default window: 7 seconds.** Server-side processing can occasionally take 15-30 seconds for large articles, so this is a user-adjustable preference (slider or input field in a settings area, range ~1-60 seconds). The grouping algorithm walks forward from the first line in a candidate group; each subsequent line that matches session+action and falls within the window (relative to the *previous* line in the group, not the first) extends the group.

The **first objectName** in the group becomes the group label, prefixed with `ARTICLE: ` — e.g. `ARTICLE: Kitchen Essentials (+34 objects)`. This prefix distinguishes grouped article rows from ungrouped singleton events.

### Display

- **Collapsed (default)**: show one row per group. Display: timestamp, user, action, group label (`ARTICLE: Kitchen Essentials (+34 objects)`), and **task name** (extracted from the data column comment, e.g. "Content Team Imports/Fits") as a separate column. The row has an expand indicator.
- **Expanded**: click to expand inline, revealing all child text object rows indented beneath the group header.
- **Toggle**: a global toggle ("Group text objects" checkbox, on by default) lets users switch between grouped and flat/raw views. Filters and export operate on whichever view is active.
- **Task name column**: visible only in grouped mode. Extracted from the `<!-- Accept Task: '...' -->` or `<!-- Changed '...' ... -->` comment pattern in the data field. Displayed as its own column so it's sortable/filterable independently of the article name.

### Impact

In the sample data, grouping reduces 7,289 lines to ~1,499 visible rows (79% reduction), making it much easier to scan actual user activity.

### Export Behavior

- **Grouped mode off**: CSV exports all individual rows as-is.
- **Grouped mode on**: two export options:
  - **"Export Summary"** (default): one row per group with columns for article name, task name, child count, timestamp range, and the standard filterable columns. No child rows.
  - **"Export with Details"**: summary rows plus all child rows beneath each group, with an indentation marker or "group ID" column to preserve the parent-child relationship.

### User-Defined Article Labels (v1)

Admins investigating log data often recognize what an article group represents but the auto-detected first objectName may not be meaningful (e.g. a truncated text fragment). They should be able to rename any article group with a custom label.

- **Inline rename**: click the article group label to edit it. The edited name replaces the auto-detected name in the display and in exports.
- **Persistence**: custom labels are stored in **localStorage**, keyed by a stable group identifier (combination of the child objectIDs in the group). Labels survive page reloads and new sessions — when the same log files (or overlapping files) are loaded again, the custom labels reappear.
- **Visual indicator**: renamed groups show a small icon or style change (e.g. italic, or a pencil icon) so the user knows it's a custom label vs. auto-detected.
- **Reset**: right-click or a small "x" on a custom label reverts to the auto-detected name and removes it from localStorage.
- **Bulk management**: a "Manage Labels" option (in settings or a modal) lists all saved custom labels with their associated objectIDs, allowing bulk delete or export/import of labels as JSON for sharing between admins.

### Future: Article-to-Text-Object Mapping

The internal data model should be structured to support an **optional article key** — a user-supplied lookup table that maps article names (or IDs) to their child text object IDs. This would allow deterministic grouping independent of time proximity, enabling accurate grouping even when a user works on an article's text objects across widely separated time windows.

**Not implemented in v1**, but the grouping data structures should anticipate it:
- Each group should store its list of child objectIDs.
- The grouping function should be pluggable — currently using time-proximity heuristics, but replaceable with a key-based lookup.
- A future UI could allow uploading a mapping file (CSV or JSON) that associates article names/IDs with their text object IDs, or the app could build this map from PLACE events in the log (which contain explicit parent-child `<relatedObject>` data).

---

## Session Tracking

The sessionID field links all events from a single user login session. To make this actionable:

- **Clickable sessionID**: when the sessionID column is visible, clicking a sessionID value auto-populates the sessionID filter, instantly narrowing the table to that session's events. This lets an admin click one event and see everything that user did in that session.
- The sessionID filter also appears in the filter bar as a text input (for pasting a known session ID).
- Consider a subtle visual grouping — a thin left border color that changes per session — when viewing unfiltered data, to help distinguish session boundaries in the timeline.

---

## Help Modal

A help button in the header opens a modal covering:

1. **Purpose & Use** — what this tool is for, who it's for, and the typical workflow (download diagnostics package → locate k4_event*.log → drag to page → filter → export). This content should **stay in sync with README.md** — either README.md is the source of truth and the modal quotes/paraphrases it, or the modal content is canonical and README.md links to the live page for details.
2. **Field Reference** — a table explaining each column: what it means in K4 terms, what values to expect, and what's useful for troubleshooting. Example: "**clientType** — which K4 client application generated the event: K4Edit (InCopy plugin), K4Layout (InDesign plugin), K4Overview (web dashboard), K4Admin (server administration), K4ContentPortal (web content portal)."
3. **How to enable event logging** — brief note that k4_event.log is disabled by default, enabled in `K4_Event_Log_Config.logback` at `C:\ProgramData\vjoon\K4_Server`, with a pointer to vjoon documentation.
4. **Tips** — e.g. "Click a session ID to filter all events from that session," "Shift+click column headers for multi-column sort," "Export only includes visible columns."

---

## Resolved Decisions

| # | Decision | Resolution |
|---|----------|------------|
| 1 | Data truncation length | 80 characters, adjust later if needed |
| 2 | Session color banding | Implement it, plan to disable if it's visual noise |
| 3 | Dedup on file merge | Not needed — log lines won't be duplicated. The major UX win is text object grouping. |
| 4 | README ↔ Help sync | Help content is authoritative in the HTML; README.md stays short and points to the live page |
| 5 | Column reorder | Show/hide only for v1 |
| 6 | File merge behavior | Merge by default + Reset button |
| 7 | Timezone | Original by default, toggle for local conversion |
| 8 | Grouping window | Default 7 seconds, user-adjustable (1-60s). Sliding window relative to previous line. |
| 9 | Grouped export | "Export Summary" (default) = one row per group. "Export with Details" = groups + children. |
| 10 | Group label | Prefix with `ARTICLE: `. Task name as a separate column, not appended to article name. |
| 11 | Article key mapping | Not in v1, but data model structured to support a future user-supplied or PLACE-derived mapping. |
| 12 | User-defined article labels | v1 feature. Click to rename group labels, persisted in localStorage, exportable/importable as JSON. |
