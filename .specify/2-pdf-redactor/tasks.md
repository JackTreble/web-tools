---
description: "Implementation tasks for Privacy-First PDF Redactor"
---

# Tasks: Privacy-First PDF Redactor

**Input**: `.specify/2-pdf-redactor/spec.md`
**Feature Branch**: `2-pdf-redactor`
**Issue**: [#2 — [PROPOSAL] Privacy-First PDF Redactor](https://github.com/JackTreble/Web-Tools/issues/2)

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (no dependencies on other tasks in the same phase)
- **[Story]**: US1, US2, US3 — maps to user story in spec.md

---

## Phase 1: Setup (Project Scaffolding)

**Purpose**: Create the file structure and load CDN dependencies — no logic yet.

- [ ] T001 Create directory `tools/pdf-redactor/` at repository root
- [ ] T002 [P] Create `tools/pdf-redactor/index.html` with HTML5 boilerplate, `<script type="module" src="main.js">`, and CDN `<script>` tags for PDF.js and pdf-lib
- [ ] T003 [P] Create `tools/pdf-redactor/style.css` with base layout: centered container, toolbar row, canvas wrapper with overflow scroll
- [ ] T004 [P] Create `tools/pdf-redactor/main.js` as empty ES Module with top-level `// TODO` stubs for each major function

**Checkpoint**: `index.html` opens in browser, console shows no errors, CDN libraries load successfully.

---

## Phase 2: Foundational (Shared Infrastructure)

**Purpose**: Core plumbing that all user stories depend on — PDF loading, page state management, canvas sizing.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T005 [US1] Implement `loadPDF(file: File): Promise<PDFDocumentProxy>` in `main.js` — uses File API + PDF.js to parse the PDF bytes; surface error for password-protected or corrupted files
- [ ] T006 [P] [US1] Implement `PageStateStore` — a Map keyed by page index storing `{ redactionRects: Rect[] }` per page; expose `get(pageIndex)`, `set(pageIndex, state)`, `clear(pageIndex)` methods
- [ ] T007 [P] [US1] Implement `renderPageToCanvas(pdfPage, canvas, scale)` — renders a `PDFPageProxy` to a given `<canvas>` element at the specified scale using PDF.js `page.render()`
- [ ] T008 [US1] Implement `computeScale(pdfPage, containerWidth)` — returns a float scale factor so the page fits the container width; re-use whenever the container resizes

**Checkpoint**: Can load a PDF in devtools console, `renderPageToCanvas` draws page 1 to a test canvas, `PageStateStore` stores and retrieves rects.

---

## Phase 3: User Story 1 — Core Redaction Loop (Priority: P1) 🎯 MVP

**Goal**: Load a PDF → draw redaction boxes → export a truly-redacted PDF. Fully functional for single-page use.

**Independent Test**: Load a 1-page PDF, draw a rectangle over text, click Download, open the output PDF, attempt to copy text from the redacted area — zero text should be retrievable.

### Implementation for User Story 1

- [ ] T009 [US1] Wire file input & drag-and-drop in `index.html` + `main.js`: on file select, call `loadPDF()`, store `PDFDocumentProxy`, call `renderPageToCanvas()` for page 1, display canvas in the tool area
- [ ] T010 [US1] Implement mouse-event redaction drawing on the canvas: `mousedown` starts a rect, `mousemove` updates its dimensions with live black fill, `mouseup` commits the rect to `PageStateStore` for the current page
- [ ] T011 [US1] Implement `redrawCanvas(pageIndex)` — re-renders the cached PDF page image and overlays all stored `redactionRects` as filled black boxes; called after every state change
- [ ] T012 [US1] Implement `exportRedactedPDF(): Promise<Uint8Array>` — iterates all pages, for each page: render to an offscreen canvas, paint stored rects as solid black, export canvas to PNG via `toDataURL`, embed PNG in a new pdf-lib page; save and return bytes
- [ ] T013 [US1] Wire "Download Redacted PDF" button: calls `exportRedactedPDF()`, wraps result in a `Blob`, creates `URL.createObjectURL`, triggers `<a download>` click, revokes object URL after download
- [ ] T014 [US1] Disable "Download" button when no PDF is loaded; re-enable after a file is successfully parsed

**Checkpoint**: Full P1 user story is functional — load, draw, download, verify no text in output.

---

## Phase 4: User Story 2 — Multi-Page Navigation & Per-Page State (Priority: P2)

**Goal**: Navigate between pages; redaction state is preserved per page across navigation.

**Independent Test**: Load a 5-page PDF, draw boxes on pages 1 and 3, navigate away and back, confirm boxes persist, export and verify all redactions appear on the correct pages.

### Implementation for User Story 2

- [ ] T015 [US2] Add page navigation UI to `index.html`: "← Prev" button, "→ Next" button, and a `<span>` showing "Page X of N"
- [ ] T016 [US2] Implement `navigateToPage(pageIndex)` in `main.js`: saves current canvas state (redaction rects already in `PageStateStore`), fetches the target page via `PDFDocumentProxy.getPage()`, calls `renderPageToCanvas()` and `redrawCanvas()`, updates page counter display
- [ ] T017 [US2] Ensure Prev/Next buttons are disabled at boundaries (page 1 has no Prev; last page has no Next)
- [ ] T018 [US2] Update `exportRedactedPDF()` to iterate ALL pages (0 to N-1), not just the current one; for pages never navigated to (no cached render), fetch and render them on-the-fly during export

**Checkpoint**: Multi-page navigation works, per-page state persists, export includes all pages with correct per-page redactions.

---

## Phase 5: User Story 3 — Redaction Management (Undo & Clear) (Priority: P3)

**Goal**: Users can undo the last box or clear all boxes on the current page.

**Independent Test**: Draw 3 boxes, undo → 2 remain. Clear page → 0 remain. Export confirms the cleared page has no boxes.

### Implementation for User Story 3

- [ ] T019 [US3] Add "Undo" and "Clear Page" buttons to the toolbar in `index.html`
- [ ] T020 [US3] Implement undo: on "Undo" click, pop the last rect from `PageStateStore` for the current page and call `redrawCanvas()`
- [ ] T021 [US3] Implement clear page: on "Clear Page" click, call `PageStateStore.clear(currentPageIndex)` and call `redrawCanvas()`
- [ ] T022 [US3] Disable "Undo" when the current page's rect list is empty; disable "Clear Page" when there are no rects on the current page

**Checkpoint**: Undo and clear work correctly; other pages' state is unaffected; export reflects cleared state.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: UX improvements, error handling, and edge-case guards that apply across all stories.

- [ ] T023 [P] Add error display: if `loadPDF()` throws (password-protected, corrupted), show a visible inline error message rather than a silent failure
- [ ] T024 [P] Add a loading indicator (spinner or progress text) while PDF.js is rendering a page, especially for large documents
- [ ] T025 [P] Guard against zero-size rectangles: in the `mouseup` handler, discard rects smaller than 5×5 canvas pixels
- [ ] T026 [P] Store redaction rects in PDF-coordinate space (not canvas-pixel space) to ensure correct positioning across zoom/resize — convert using the current scale factor when drawing and when exporting
- [ ] T027 [P] Add responsive layout in `style.css`: canvas scrolls horizontally if wider than viewport; toolbar stays fixed at top
- [ ] T028 Add the tool to the main `index.html` tool directory listing (link card) so it is discoverable on the site
- [ ] T029 [P] Manual smoke-test checklist: single-page PDF, multi-page PDF, large PDF, password-protected PDF, landscape PDF — verify all edge cases from spec

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 — **BLOCKS all user stories**
- **Phase 3 (US1 — Core Loop)**: Depends on Phase 2 — this is the MVP
- **Phase 4 (US2 — Multi-page)**: Depends on Phase 2; can integrate with Phase 3 output
- **Phase 5 (US3 — Undo/Clear)**: Depends on Phase 2; can start after Phase 3 checkpoint
- **Phase 6 (Polish)**: Depends on all desired user stories being complete

### Within Each Phase

- Tasks marked [P] within a phase can run in parallel
- T018 (export all pages) depends on T012 (single-page export) being complete first
- T016 (navigation logic) depends on T009 (PDF loading wire-up) being complete

### MVP Delivery Path

1. Complete Phase 1 + Phase 2 → foundation ready
2. Complete Phase 3 (US1) → **stop and validate the core redaction loop**
3. Complete Phase 4 (US2) → multi-page support
4. Complete Phase 5 (US3) → undo/clear
5. Complete Phase 6 → polish and ship
