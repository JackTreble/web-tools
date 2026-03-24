# Feature Specification: Privacy-First PDF Redactor

**Feature Branch**: `2-pdf-redactor`
**Created**: 2026-03-24
**Status**: Draft
**Source**: [Issue #2 — [PROPOSAL] Privacy-First PDF Redactor](https://github.com/JackTreble/Web-Tools/issues/2)

## Problem Statement

Online PDF redaction is broken in two distinct ways:

1. **Privacy Violation via Server Uploads** — every major free tool (Smallpdf, ILovePDF, Sejda, PDFescape) requires uploading sensitive documents to cloud servers. This is a hard blocker for legal, medical, and financial professionals due to GDPR, HIPAA, and basic data hygiene.

2. **Fake Redaction** — most free tools draw a black rectangle *on top of* the PDF as a visual layer. The underlying text data remains in the file structure and is trivially reversible (select-all → copy → paste). This has caused real legal and compliance incidents.

The only trustworthy option is Adobe Acrobat Pro at $23/month. A clean, free, truly local tool does not exist.

## Solution

A 100% client-side PDF redaction tool using a Canvas-based rendering pipeline:

1. **PDF.js** (Mozilla CDN) renders each PDF page to an HTML5 `<canvas>` element — no server, no upload.
2. **Canvas API** (native) — user draws redaction rectangles over sensitive regions with live black-box preview.
3. **pdf-lib** (CDN) — exports a brand-new PDF built entirely from the flattened canvas images.

The output PDF contains **zero text data structures** — only pixel data rendered from the Canvas. The underlying text is irreversibly gone, not hidden behind a layer.

---

## User Scenarios & Testing

### User Story 1 — Core Redaction Loop (Priority: P1)

A legal professional has a contract PDF with sensitive client names and figures. They open the tool in their browser, drag-and-drop the PDF, draw black boxes over each sensitive region on each page, then download the redacted PDF — all without the file ever leaving their machine.

**Why this priority**: This is the entire value proposition. Without this, the tool does not exist.

**Independent Test**: Load a multi-page PDF, draw at least one redaction rectangle per page, export, and verify that (a) the exported PDF opens correctly, (b) the redacted regions appear as solid black boxes, and (c) copy-pasting text from the exported PDF does NOT reveal any content from the redacted regions.

**Acceptance Scenarios**:

1. **Given** a user opens the tool and has a PDF file ready, **When** they drag-and-drop (or click to select) the PDF, **Then** the first page renders as a canvas within ~3 seconds for a typical document.
2. **Given** a rendered page, **When** the user clicks and drags to draw a rectangle over a region, **Then** a filled black box appears immediately over that region as live preview feedback.
3. **Given** one or more redaction rectangles drawn across one or more pages, **When** the user clicks "Download Redacted PDF", **Then** a new PDF file downloads containing all pages with redaction boxes permanently baked into the pixel data.
4. **Given** the downloaded PDF, **When** the user opens it and attempts to select/copy text in any redacted region, **Then** no underlying text is accessible — the redacted content is truly gone.

---

### User Story 2 — Multi-Page Navigation & Per-Page State (Priority: P2)

A compliance officer has a 20-page financial statement. They need to redact items on pages 3, 7, and 15 independently. They navigate between pages, draw redaction boxes on each, and the boxes are retained when they move away and come back to a page.

**Why this priority**: Most real-world use cases involve multi-page documents. Without per-page state persistence, the tool is impractical for anything beyond a 1-page PDF.

**Independent Test**: Load a 5+ page PDF. Draw redaction boxes on pages 1 and 3. Navigate to page 2, then back to page 1. Verify the boxes on page 1 are still visible. Navigate to page 3 and verify its boxes are also intact. Export and verify all redactions appear in the correct pages of the output.

**Acceptance Scenarios**:

1. **Given** a multi-page PDF is loaded, **When** the tool renders it, **Then** page navigation controls (Previous / Next, page counter) are visible and functional.
2. **Given** the user is on page 3 and draws 2 redaction boxes, **When** they navigate to page 4 and back to page 3, **Then** the 2 boxes are still displayed correctly.
3. **Given** redaction boxes drawn on multiple pages, **When** the user downloads the redacted PDF, **Then** every page's redactions are present in the correct positions in the output file.

---

### User Story 3 — Redaction Management (Edit & Clear) (Priority: P3)

A user accidentally drew a redaction box over the wrong section. They need to undo that last action or clear all boxes on the current page and start over, without having to re-load the entire document.

**Why this priority**: Enhances usability and reduces frustration. The core workflow still functions without this, but errors are costly without an undo path.

**Independent Test**: Load a PDF, draw 3 boxes, use "Undo" to remove the last one, verify only 2 remain. Then use "Clear Page" to remove all boxes from the current page, verify the canvas shows none. Export and confirm no redaction boxes appear where they were cleared.

**Acceptance Scenarios**:

1. **Given** one or more boxes are drawn on the current page, **When** the user clicks "Undo", **Then** the most recently drawn box is removed.
2. **Given** multiple boxes are drawn on the current page, **When** the user clicks "Clear Page", **Then** all boxes on the current page are removed (other pages' boxes are unaffected).
3. **Given** the user clears boxes from page 2, **When** they export, **Then** the output PDF's page 2 has no redaction boxes, but page 1 and page 3 retain their redactions.

---

### Edge Cases

- What happens when the user loads a password-protected PDF? (Show a clear error message; do not attempt to process.)
- What happens when the user loads a very large PDF (100+ pages, 50MB+)? (Show a loading indicator; render pages on-demand rather than all at once.)
- What if the user draws a zero-size rectangle (accidental click without drag)? (Ignore rectangles below a minimum size threshold, e.g., 5×5px.)
- What happens when the PDF has landscape-oriented pages mixed with portrait? (Each page canvas must respect the page's own dimensions.)
- What if the user tries to download before loading a PDF? (Download button is disabled or shows a prompt to load a file first.)
- What if PDF.js or pdf-lib fails to load from CDN? (Show a user-friendly error with guidance to check network connection.)

---

## Requirements

### Functional Requirements

- **FR-001**: The tool MUST render PDF pages to `<canvas>` elements using PDF.js loaded from the Mozilla CDN — no server-side processing.
- **FR-002**: Users MUST be able to load a PDF via drag-and-drop onto the tool area OR by clicking a file-picker button.
- **FR-003**: Users MUST be able to draw redaction rectangles by clicking and dragging on the rendered canvas.
- **FR-004**: Redaction rectangles MUST be rendered as filled, solid black boxes in the live preview.
- **FR-005**: The tool MUST maintain independent per-page redaction state so boxes drawn on one page are not lost when navigating to another.
- **FR-006**: Users MUST be able to navigate between pages using Previous/Next controls; current page and total page count MUST be displayed.
- **FR-007**: Users MUST be able to undo the last drawn rectangle (single-level undo minimum).
- **FR-008**: Users MUST be able to clear all redaction boxes on the current page.
- **FR-009**: The tool MUST export a new PDF via pdf-lib where all pages are rendered as flattened pixel images — no text layer, no metadata from the original document.
- **FR-010**: The exported PDF MUST be downloaded directly to the user's device using `URL.createObjectURL` + a programmatic `<a>` click — no server upload.
- **FR-011**: The tool MUST display a clear error message if a password-protected or corrupted PDF is loaded.
- **FR-012**: The tool MUST NOT transmit any file data outside the browser at any point.

### Key Entities

- **PDFDocument**: The loaded PDF; has N pages, loaded via File API, rendered by PDF.js.
- **PageState**: Per-page record of `{ canvasImageData, redactionRects: Rect[] }`. Persisted in memory across page navigation.
- **Rect**: A redaction rectangle `{ x, y, width, height }` in page-coordinate space (not screen-pixel space, to be zoom-independent).
- **RedactedPDF**: The output PDF built by pdf-lib from flattened canvas images with redaction boxes baked in.

---

## Technical Approach

> **No-Backend Rule**: All processing MUST occur in the browser. No server-side logic. GitHub Pages compatible.

### Dependencies (CDN Only — No Build Step)

| Library | Source | Purpose |
|---------|--------|---------|
| PDF.js | `https://mozilla.github.io/pdf.js/` | Render PDF pages to canvas |
| pdf-lib | `https://unpkg.com/pdf-lib` | Build output PDF from canvas images |

### File Structure

```
tools/pdf-redactor/
├── index.html   # Single-page app shell, CDN script tags
├── style.css    # Layout, canvas container, toolbar styling
└── main.js      # ES Module: PDF loading, canvas rendering, redaction logic, export
```

### Architecture Notes

- **Zoom Independence**: Store redaction rectangles in PDF-coordinate space (not canvas-pixel space). Convert on render. This ensures rectangles stay correctly positioned if the user resizes the viewport.
- **Multi-page Strategy**: Render only the current page to the active canvas. Cache rendered page images (as `ImageData` or `offscreenCanvas`) to avoid re-rendering on every navigation. Store `redactionRects[]` per page index.
- **Export Pipeline**: For each page — get cached canvas image → paint redaction rects as filled black rectangles → convert canvas to PNG via `canvas.toDataURL('image/png')` → embed in pdf-lib page. Assemble all pages → `PDFDocument.save()` → `Uint8Array` → `Blob` → `URL.createObjectURL` → trigger download.
- **Web Workers (Optional / Phase 2)**: Offload PDF.js rendering to a Worker to keep the UI responsive on large documents.

---

## Success Criteria

### Measurable Outcomes

- **SC-001**: A user can load a PDF, draw redactions on all pages, and download the output in under 60 seconds for a typical 10-page document on a modern laptop.
- **SC-002**: Copy-pasting text from any redacted region in the output PDF yields zero characters of the original content.
- **SC-003**: Zero bytes of file data are transmitted to any external server (verifiable via browser DevTools Network tab with no outbound XHR/fetch/form-submit calls containing file data).
- **SC-004**: The tool functions correctly on the latest versions of Chrome, Firefox, and Safari without any native install or plugin.
- **SC-005**: The tool loads and is usable offline after the initial CDN assets are cached (or with CDNs available).
