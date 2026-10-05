# Expiry Manager — AGENTS.md

## 1. Project Purpose

Expiry Manager is an offline-first pharmacy branch tool for tracking product batches approaching expiry.

Primary workflow:
1. Add a product/batch.
2. Store product, batch, quantity, unit, expiry month/year, barcode, and notes.
3. Classify each batch by expiry status.
4. Search/filter batches.
5. Export data.
6. Create and restore local backups.

The application is intended for use by individual pharmacy branches. Each branch's data is local and independent.

## 2. Core Constraints

- Offline-first.
- No server, cloud database, authentication, or mandatory internet connection.
- No paid external services.
- Prefer browser-native APIs.
- Use IndexedDB for persistent application data.
- Do not replace IndexedDB with localStorage for the main dataset.
- Mobile-first and responsive; Android and iOS browsers are primary targets.
- Keep the application lightweight and fast.
- Preserve existing working functionality unless a change is explicitly required.
- Make the smallest safe change necessary.
- Do not rewrite unrelated code.
- Do not introduce frameworks or dependencies unless there is a clear technical reason.

## 3. Current Project Stage

The project currently starts from a single-file Phase 1 application:

`Expiry_Manager_Phase1.html`

Phase 1 focuses on:
- Batch CRUD
- IndexedDB persistence
- Search/filtering
- Expiry classification
- CSV export
- JSON backup/restore
- Basic responsive UI

Do not assume a feature exists. Inspect the actual code before modifying it.

## 4. Expiry Classification — AUTHORITATIVE RULE

Expiry is stored as MONTH + YEAR. Do not invent day-level expiry dates.

The current calendar month is the reference point.

Definitions:

### Expired
The expiry month/year is before the current month/year.

### Remove from Shelf
The expiry month is:
- current calendar month
- OR the next 1 calendar month
- OR the next 2 calendar months

In other words: current month + next two calendar months.

### Near Expiry
The expiry month is the next 3rd or 4th calendar month after the current month.

### Future
Anything later than the Near Expiry window.

Example: on October 1, 2026:

- September 2026 and earlier -> Expired
- October 2026 -> Remove from Shelf
- November 2026 -> Remove from Shelf
- December 2026 -> Remove from Shelf
- January 2027 -> Near Expiry
- February 2027 -> Near Expiry
- March 2027 and later -> Future

IMPORTANT:
- Month/year calculations must correctly cross December -> January.
- Do not compare month numbers without accounting for year.
- Use a normalized month index such as `year * 12 + monthIndex` when appropriate.
- Keep this policy centralized in one testable function.
- Any proposed change to this policy requires explicit approval.

## 5. Data Integrity

Validate user input before saving:
- Product name should not be empty.
- Quantity must be valid and non-negative.
- Expiry month must be 1–12.
- Expiry year must be valid.
- Barcode is optional.
- Batch number is optional.
- Notes are optional.

Do not silently corrupt or discard existing records.

When editing a record:
- Preserve its stable ID.
- Update only intended fields.

When deleting:
- Require a clear user action.
- Do not accidentally delete unrelated records.

## 6. IndexedDB

- IndexedDB is the source of truth for persistent batch data.
- Preserve existing database compatibility whenever possible.
- Any schema/database version change must use a proper IndexedDB migration.
- Never casually delete or recreate the database.
- Do not introduce destructive migrations.
- Keep database operations centralized and predictable.
- Handle database errors gracefully in the UI.

Before changing the database schema:
1. Inspect the current schema.
2. Explain the migration impact.
3. Implement an upgrade path.
4. Preserve existing user data.

## 7. Backup and Restore

JSON backup/restore is local data protection.

Backup should contain:
- application/schema version
- export timestamp
- stored batch records

Restore must:
- Validate the JSON structure before modifying IndexedDB.
- Reject malformed or incompatible backups safely.
- Never partially destroy existing data because of an invalid backup.
- Clearly communicate what will happen before destructive operations.
- Prefer merge/import behavior unless the UI explicitly supports full replacement.

Do not trust imported JSON blindly.

## 8. CSV Export

CSV export must:
- Correctly escape commas, quotes, and line breaks.
- Preserve Unicode/Arabic text.
- Include clear column headers.
- Export all relevant batch fields.
- Use a UTF-8 BOM when needed for Excel compatibility.
- Generate a sensible filename.

## 9. UI / UX

Primary users are pharmacy staff, often using a phone.

Requirements:
- Mobile-first.
- Touch-friendly controls.
- Clear status labels.
- Fast search.
- Simple filtering.
- Avoid unnecessary screens and clicks.
- Do not hide critical information behind complicated interactions.
- Keep quantity and expiry information visually clear.
- Maintain accessible contrast and readable font sizes.

Do not make large visual redesigns unless explicitly requested.

## 10. Security / Privacy

The application is intended to keep branch data local.

- Do not send product or branch data to external APIs.
- Do not add analytics/tracking.
- Do not add external AI/OCR services without explicit approval.
- Do not expose stored data unnecessarily.
- Treat backup files as potentially sensitive operational data.

## 11. PWA / Offline Roadmap

PWA functionality is planned but must not be added destructively.

Future PWA work may include:
- manifest
- service worker
- offline asset caching
- installability

Do not claim the application is a fully installable PWA unless the required manifest and service worker are actually implemented and tested.

## 12. Future Roadmap

Implement in controlled stages:

### Phase 1 — Core
CRUD, IndexedDB, search/filter, expiry classification, CSV, JSON backup/restore.

### Phase 2 — Mobile workflow
Improved mobile UX, faster data entry, better batch management.

### Phase 3 — Barcode / Camera / OCR
Barcode scanning and product identification.
OCR should be introduced only after the core workflow is stable.

### Phase 4 — PWA
Manifest, service worker, offline caching, installation.

### Phase 5 — Reporting
Expiry dashboard, summaries, counts, filters, and operational reports.

Do not implement future phases automatically just because they are listed here.

## 13. Development Rules

Before making changes:
1. Read this AGENTS.md.
2. Inspect the actual existing code.
3. Identify the smallest relevant change.
4. Preserve unrelated functionality.

After making changes:
1. Check for syntax errors.
2. Test the affected workflow.
3. Test edge cases.
4. Verify existing core features still work.
5. Summarize exactly what changed.

Never claim something was tested if it was not actually tested.

## 14. Testing Requirements for Expiry Logic

At minimum, test these cases using current month = October 2026:

- September 2026 -> Expired
- October 2026 -> Remove
- November 2026 -> Remove
- December 2026 -> Remove
- January 2027 -> Near Expiry
- February 2027 -> Near Expiry
- March 2027 -> Future

Also test year boundaries, especially:
- December 2026 -> January 2027
- November 2026 -> January 2027
- December 2026 -> February 2027

## 15. Agent Behavior

When asked to modify the project:

- Do not guess about existing code.
- Inspect files first.
- Do not rebuild the project from scratch unless explicitly instructed.
- Do not change requirements silently.
- Do not modify unrelated files.
- Keep changes minimal and maintainable.
- Prefer clear vanilla JavaScript over unnecessary abstraction.
- Explain important architectural changes briefly.
- If requirements conflict, stop and ask for clarification instead of choosing silently.
