---
description: "Task list for PDF Merger & Splitter implementation"
---

# Tasks: PDF Merger & Splitter

**Input**: Design documents from `/specs/003-pdf-merger-splitter/`
**Prerequisites**: plan.md ✅, spec.md ✅

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story. P1 (Merge) can be shipped as a standalone MVP before P2 and P3 are complete.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel with other [P]-tagged tasks in the same phase
- **[Story]**: Which user story this task belongs to (US1 = Merge, US2 = Split, US3 = Organise)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project scaffolding, vendor verification, and shared HTML/CSS structure. Must complete before any user story work begins.

- [ ] T001 Verify `/tools/vendor/jszip/` exists; if absent, add JSZip to `package.json` and `scripts/sync-vendor.mjs`, run `npm run vendor:update`, and commit vendor files
- [ ] T002 [P] Copy `/tools/template.html` to `/tools/pdf-merger-splitter.html` and replace all placeholders (title, meta description, tool name, script/style paths)
- [ ] T003 [P] Create `/tools/pdf-merger-splitter/style.css` with tool-specific layout tokens (tab bar, file card grid, thumbnail grid, progress bar, range input rows); import `/tools/common.css` for shared tokens
- [ ] T004 [P] Create `/tools/pdf-merger-splitter/main.js` as an ES module with top-level import stubs and a `tabController` that switches between Merge / Split / Organise tabs and resets per-tab state

**Checkpoint**: Entry page loads in the browser with the three tabs visible and no console errors.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared utilities used by all three user stories.

**⚠️ CRITICAL**: No user story implementation can begin until this phase is complete.

- [ ] T005 [P] Implement `loadPdfFile(file) → { name, pageCount, buffer }` — reads a File object with the File API, returns an ArrayBuffer; wraps pdf-lib's `PDFDocument.load()` to extract page count; surfaces errors for corrupt or password-protected files
- [ ] T006 [P] Implement `downloadBlob(blob, filename)` — creates a temporary `<a>` element, triggers click, revokes the object URL; used by all three export actions
- [ ] T007 [P] Implement `showProgress(pct)` / `hideProgress()` — updates a shared determinate progress bar element; called during long-running operations to keep the UI responsive
- [ ] T008 [P] Implement `showError(message)` / `clearError()` — renders a user-facing inline error banner; called on pdf-lib exceptions and validation failures

**Checkpoint**: All shared utilities pass smoke tests (manual calls in browser console confirm correct behaviour).

---

## Phase 3: User Story 1 — Merge Multiple PDFs (Priority: P1) 🎯 MVP

**Goal**: Users can load multiple PDFs, reorder them by dragging file cards, and download a merged PDF — entirely in the browser.

**Independent Test**: Load 3 PDFs via file picker, drag card 3 before card 1, click Merge, open the downloaded file and verify it contains all pages in the new order. Check DevTools Network tab confirms zero outbound requests.

### Implementation for User Story 1

- [ ] T009 [P] [US1] Build the Merge tab HTML structure inside `pdf-merger-splitter.html`: drop zone, file card list container, Merge button, progress bar placeholder, error banner placeholder
- [ ] T010 [P] [US1] Implement file-picker and drag-and-drop-onto-drop-zone handlers in `main.js`; call `loadPdfFile()` for each selected file and push results into `fileListState` array; render a file card for each entry showing name and page count badge
- [ ] T011 [US1] Implement drag-to-reorder for file cards using the HTML5 Drag-and-Drop API; update `fileListState` order on drop; provide visual position feedback (placeholder gap) during drag (depends on T010)
- [ ] T012 [US1] Add a Remove button to each file card that splices the entry from `fileListState` and removes the card from the DOM (depends on T010)
- [ ] T013 [US1] Implement `mergeAction()` in `main.js`: iterate `fileListState`, load each buffer into a `PDFDocument` via pdf-lib, copy all pages into a new `PDFDocument`, save to `Uint8Array`, wrap in a Blob, call `downloadBlob()`; call `showProgress()` / `hideProgress()` around the operation (depends on T005, T006, T007, T011)
- [ ] T014 [US1] Wire Merge button click to `mergeAction()`; disable button while merge is in progress; re-enable on completion or error (depends on T013)
- [ ] T015 [US1] Add error handling in `mergeAction()` for: empty file list, password-protected PDFs, corrupt files; call `showError()` with a descriptive message (depends on T008, T013)

**Checkpoint**: User Story 1 fully functional. Load 3 PDFs, reorder, merge, download, verify output. Zero network requests. Clear error shown for protected PDF.

---

## Phase 4: User Story 2 — Split by Page Range (Priority: P2)

**Goal**: Users can load a single PDF, define page ranges, and download each range as a separate PDF (or all ranges as a zip archive).

**Independent Test**: Load a 10-page PDF, add ranges "1-3" and "7-10", click Split All, open the downloaded zip and verify two PDFs with 3 and 4 pages respectively.

### Implementation for User Story 2

- [ ] T016 [P] [US2] Build the Split tab HTML structure: single-file drop zone, range-input row component (start field, end field, remove-range button), Add Range button, Split All button, Split into Pages button, progress bar placeholder, error banner placeholder
- [ ] T017 [P] [US2] Implement single-file load for the Split tab (reuse `loadPdfFile()`); display loaded file name and total page count; store buffer in `splitFileState`
- [ ] T018 [US2] Implement dynamic Add Range / Remove Range UI: each row holds a start/end input pair; Add Range appends a new row; Remove Range removes the row and its entry from `rangeState` (depends on T016)
- [ ] T019 [US2] Implement inline range validation: on each input change, check start ≥ 1, end ≤ pageCount, start ≤ end; show per-field error text; disable Split All button if any row is invalid or empty (depends on T018)
- [ ] T020 [US2] Implement `splitAction(ranges)` in `main.js`: for each range, load the PDF buffer, copy the specified page slice into a new `PDFDocument`, save to `Uint8Array`; if single range, call `downloadBlob()` directly; if multiple ranges, use JSZip to bundle all files and download the archive; call `showProgress()` / `hideProgress()` (depends on T005, T006, T007, T017, T019)
- [ ] T021 [US2] Implement "Split into individual pages" shortcut: auto-populate `rangeState` with one range per page and call `splitAction()` (depends on T020)
- [ ] T022 [US2] Add error handling in `splitAction()` for password-protected or corrupt input; call `showError()` (depends on T008, T020)

**Checkpoint**: User Stories 1 and 2 both independently functional. Split produces correct per-range PDFs; zip download contains all split files.

---

## Phase 5: User Story 3 — Page Organiser (Priority: P3)

**Goal**: Users can load a PDF, view page thumbnails in a drag-reorderable grid, delete pages, and download the revised document.

**Independent Test**: Load a 5-page PDF, drag page 3 before page 1, delete page 5, click Download, verify output has 4 pages in the correct order.

### Implementation for User Story 3

- [ ] T023 [P] [US3] Build the Organise tab HTML structure: single-file drop zone, thumbnail grid container, Reset button, Download button, progress bar placeholder, error banner placeholder
- [ ] T024 [P] [US3] Implement single-file load for the Organise tab (reuse `loadPdfFile()`); store buffer and initial page order array in `organiseState`
- [ ] T025 [US3] Implement `renderThumbnails(pdfBuffer, pageOrder)` using PDF.js: for each page index in `pageOrder`, render the page to a `<canvas>` at scale 0.25, wrap in a thumbnail card with a delete button and drag handle; display in the grid (depends on T024)
- [ ] T026 [US3] Implement drag-to-reorder for thumbnail cards using the HTML5 Drag-and-Drop API; update `organiseState.pageOrder` on drop and re-render affected cards (depends on T025)
- [ ] T027 [US3] Implement per-page delete: clicking the delete button on a thumbnail card removes that page index from `organiseState.pageOrder` and removes the card from the DOM (depends on T025)
- [ ] T028 [US3] Implement Reset button: restore `organiseState.pageOrder` to the original loaded order and call `renderThumbnails()` to rebuild the grid (depends on T025)
- [ ] T029 [US3] Implement `organiseExport()`: load the original PDF buffer via pdf-lib, copy pages in the order specified by `organiseState.pageOrder` into a new `PDFDocument`, save and call `downloadBlob()`; call `showProgress()` / `hideProgress()` (depends on T005, T006, T007, T026, T027)
- [ ] T030 [US3] Add error handling in `organiseExport()` and `renderThumbnails()` for corrupt or password-protected PDFs; call `showError()` (depends on T008, T029)

**Checkpoint**: All three user stories independently functional. Thumbnail grid renders correctly, drag-reorder and delete update the output, Reset restores original order.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: SEO, accessibility, cross-browser validation, and final quality pass.

- [ ] T031 [P] Add SEO metadata to `pdf-merger-splitter.html`: descriptive `<title>`, `<meta name="description">`, relevant meta tags; copy aligned to search queries "merge PDF online free", "split PDF online free", "combine PDF files browser"
- [ ] T032 [P] Add a "How to use" section to the tool page covering all three modes (Merge, Split, Organise)
- [ ] T033 [P] Verify the tool renders correctly and all features work on mobile viewports (responsive layout check for file card list, thumbnail grid, range inputs)
- [ ] T034 Manual smoke test in Chrome, Firefox, and Safari (latest 2 versions each): run the full acceptance scenarios from spec.md for all three user stories
- [ ] T035 [P] Confirm zero outbound network requests during any operation (DevTools Network tab check for all three modes)
- [ ] T036 [P] Code review pass: remove debug logs, confirm no CDN references in shipped HTML/JS/CSS, confirm all vendor paths point to `/tools/vendor/`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 completion — blocks all user stories
- **Phase 3 (US1 Merge)**: Depends on Phase 2 — can be delivered as standalone MVP
- **Phase 4 (US2 Split)**: Depends on Phase 2 — can proceed in parallel with Phase 3 if staffed
- **Phase 5 (US3 Organise)**: Depends on Phase 2 — can proceed in parallel with Phases 3 & 4 if staffed
- **Phase 6 (Polish)**: Depends on all desired user stories being complete

### Parallel Opportunities

- T002, T003, T004 (Phase 1) can run in parallel after T001 vendor check
- T005, T006, T007, T008 (Phase 2) can all run in parallel
- T009, T010 (Phase 3 setup) can run in parallel; T011–T015 are sequential within US1
- T016, T017 (Phase 4 setup) can run in parallel; T018–T022 are sequential within US2
- T023, T024 (Phase 5 setup) can run in parallel; T025–T030 are sequential within US3
- All Phase 6 tasks marked [P] can run in parallel

### MVP Delivery Path

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1 (Merge)
4. **STOP and VALIDATE**: Test merge independently → ship as MVP
5. Add Phase 4 (Split) → validate → ship
6. Add Phase 5 (Organise) → validate → ship
7. Complete Phase 6: Polish before final release

---

## Notes

- [P] tasks = different files or independent concerns, no shared state dependencies
- All vendor references must use committed files under `/tools/vendor/` — no CDN links
- Commit after each phase checkpoint at minimum
- Verify zero network requests at each phase checkpoint using DevTools
- Password-protected PDF error handling must be implemented before any mode is considered complete
