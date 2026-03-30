# Feature Specification: PDF Merger & Splitter

**Feature Branch**: `003-pdf-merger-splitter`
**Created**: 2026-03-30
**Status**: Draft
**Input**: Issue #21 — [PROPOSAL] - PDF Merger & Splitter

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Merge Multiple PDFs (Priority: P1)

A non-technical user (legal professional, HR admin, student) has multiple PDF files they need to combine into one document — for example, assembling a contract packet from separate signature pages and attachments. They drag the files into the tool, reorder them if needed, click Merge, and download the combined PDF — entirely in the browser with no upload.

**Why this priority**: Merging is the highest-volume use case ("merge PDF online free" is one of the most searched document tasks). It delivers immediate, standalone value and uses the simplest pdf-lib operations.

**Independent Test**: Load 3 PDF files, reorder them via drag-and-drop, click Merge, and verify the downloaded file contains all pages in the chosen order — without any network request leaving the browser.

**Acceptance Scenarios**:

1. **Given** a user loads 3 PDF files via the file picker or drag-and-drop, **When** they reorder file cards and click Merge, **Then** a merged PDF is downloaded containing all pages in the specified order.
2. **Given** a user attempts to remove a file card before merging, **When** they click the Remove button on a card, **Then** that file is excluded from the output and the remaining files merge correctly.
3. **Given** a user loads a password-protected PDF, **When** they attempt to merge, **Then** a clear error message is shown and no partial output is produced.
4. **Given** 5 PDFs each under 5 MB are loaded, **When** merge is triggered, **Then** the output is ready for download in under 10 seconds.

---

### User Story 2 - Split by Page Range (Priority: P2)

A user has a 20-page PDF — perhaps a multi-section report — and needs to extract specific page ranges as separate files. They load the PDF, define one or more page ranges (e.g. pages 1–5, 6–10), and download the split files, either individually or as a zip archive.

**Why this priority**: Split/extract is the natural complement to merge and represents the second-highest search volume. It adds meaningful value to P1 users without requiring the more complex thumbnail UI.

**Independent Test**: Load a multi-page PDF, enter two page ranges, click Split, and verify each downloaded file contains only the specified pages.

**Acceptance Scenarios**:

1. **Given** a user loads a 10-page PDF and enters range "1-5", **When** they click Split, **Then** a PDF containing only pages 1–5 is downloaded.
2. **Given** a user enters multiple ranges, **When** they click Split All, **Then** all split files are packaged into a zip archive and downloaded.
3. **Given** a user enters an invalid range (e.g. start > end, or page number out of bounds), **When** they attempt to split, **Then** an inline validation error is shown and the Split button is disabled until resolved.
4. **Given** a user clicks "Split into individual pages", **When** the operation completes, **Then** each page is exported as a separate PDF and all are bundled into a zip.

---

### User Story 3 - Page Organiser (Priority: P3)

A user wants to reorder or delete individual pages within a single PDF — for example, removing a blank page or rearranging sections. They load the PDF, see thumbnail previews of each page in a grid, drag pages to new positions or delete them, and download the revised document.

**Why this priority**: More advanced than merge/split; requires thumbnail rendering via PDF.js and a draggable grid. Delivers high value for power users but is not required for the MVP.

**Independent Test**: Load a PDF, drag page 3 before page 1, delete page 5, click Download, and verify the output matches the new page order with page 5 absent.

**Acceptance Scenarios**:

1. **Given** a user loads a 5-page PDF, **When** the page organiser renders, **Then** thumbnail images of all 5 pages are displayed in a draggable grid.
2. **Given** a user drags page 3 before page 1, **When** they click Download, **Then** the output PDF reflects the updated page order.
3. **Given** a user deletes a page thumbnail, **When** they click Download, **Then** the output PDF excludes that page.
4. **Given** a user clicks Reset, **When** the action completes, **Then** the page order is restored to the original loaded order.

---

### Edge Cases

- What happens when a user loads a PDF with 0 pages or a corrupt file? → Show a clear error, do not attempt processing.
- What happens if total input size is very large (e.g. 10 × 10 MB)? → Processing may be slow; show a progress indicator and do not block the UI thread.
- What happens if a range input field is left empty? → Treat as invalid; prevent split until all ranges are filled and valid.
- What happens if only one file is added to the merge list? → Allow merge (output is a copy); optionally warn the user.
- What happens when the browser tab is closed mid-operation? → All data is lost (acceptable — no server state to clean up).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The tool MUST load PDF files from disk using the File API with no server upload.
- **FR-002**: The tool MUST allow users to add multiple PDF files via file picker and/or drag-and-drop onto the file list area.
- **FR-003**: The tool MUST display each loaded file as a card showing file name and page count.
- **FR-004**: Users MUST be able to reorder file cards via drag-and-drop before merging.
- **FR-005**: Users MUST be able to remove individual files from the list before merging.
- **FR-006**: The tool MUST merge all listed PDFs in display order into a single valid PDF output.
- **FR-007**: The tool MUST trigger a local file download of the merged PDF using Blob + `URL.createObjectURL`.
- **FR-008**: The tool MUST allow users to load a single PDF and define one or more page ranges for splitting.
- **FR-009**: Page range inputs MUST be validated inline (bounds check, start ≤ end) with clear error messages.
- **FR-010**: The tool MUST provide a one-click "Split into individual pages" shortcut.
- **FR-011**: When multiple split ranges are defined, the tool MUST package all output files into a zip archive for download.
- **FR-012**: The tool MUST render page thumbnails for the page organiser view using PDF.js and Canvas API.
- **FR-013**: Users MUST be able to drag-reorder page thumbnails in the organiser grid.
- **FR-014**: Users MUST be able to delete individual pages in the organiser view.
- **FR-015**: The organiser view MUST include a Reset action to restore the original page order.
- **FR-016**: The tool MUST show a progress indicator during merge, split, and organiser export operations.
- **FR-017**: The tool MUST display clear error messages for password-protected PDFs, invalid/corrupt files, and invalid ranges.
- **FR-018**: Zero bytes of PDF content MUST be transmitted to any external server at any point.

### Key Entities

- **PDF File Entry**: A loaded file with metadata — name, page count, raw ArrayBuffer — used in the merge list.
- **Page Range**: A user-defined start/end pair used to define a split operation; validated before processing.
- **Page Thumbnail**: A Canvas-rendered image of a single PDF page used in the organiser grid; carries page index and current position.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Zero bytes of PDF content transmitted to any external server (verifiable via browser DevTools Network tab).
- **SC-002**: Full merge workflow (load 3 files → reorder → merge → download) completable in under 60 seconds.
- **SC-003**: Full split workflow (load PDF → define range → split → download) completable in under 30 seconds.
- **SC-004**: Merged output from 5 files each under 5 MB is ready for download in under 10 seconds.
- **SC-005**: All output PDFs open without errors in the latest versions of Adobe Acrobat Reader, Chrome PDF viewer, and macOS Preview.
- **SC-006**: All three user stories function correctly in the latest 2 major versions of Chrome, Firefox, and Safari.
