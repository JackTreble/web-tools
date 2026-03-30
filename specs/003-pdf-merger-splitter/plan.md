# Implementation Plan: PDF Merger & Splitter

**Branch**: `003-pdf-merger-splitter` | **Date**: 2026-03-30 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/003-pdf-merger-splitter/spec.md`

## Summary

Build a fully client-side PDF merge, split, and page-organiser tool that runs entirely in the browser. The tool uses **pdf-lib** (already vendored) for all PDF read/write operations, **PDF.js** (already vendored) for page thumbnail rendering, the **File API** for loading files, the **Drag-and-Drop API** for reordering, **JSZip** for packaging multi-file split output, and **Blob + `URL.createObjectURL`** for local downloads. No server communication occurs at any point.

## Technical Context

**Language/Version**: HTML5, CSS3, modern JavaScript (ES Modules, no bundler)
**Primary Dependencies**:
- `pdf-lib` — vendored at `/tools/vendor/pdf-lib/` (already present for PDF Redactor)
- `PDF.js` — vendored at `/tools/vendor/pdfjs/` (already present for PDF Redactor)
- `JSZip` — vendored at `/tools/vendor/jszip/` (needs vendor check; add if missing)

**Storage**: None — all data lives in-memory (ArrayBuffers, Blobs) for the duration of the session.
**Testing**: Manual browser testing per acceptance scenarios; no automated test framework.
**Target Platform**: GitHub Pages (static file hosting); latest 2 major versions of Chrome, Firefox, Safari.
**Project Type**: Static single-page web tool.
**Performance Goals**: Merge 5 × 5 MB PDFs in < 10 s; split operation < 30 s end-to-end in a modern browser.
**Constraints**: No backend, no CDN references in shipped code, offline-capable after first page load, < 200 KB of new tool-specific JS/CSS (excluding vendored libraries).
**Scale/Scope**: Single-user browser tool; no concurrency concerns.

## Constitution Check

- **No-Backend rule**: All processing uses browser-only APIs (File API, Canvas, Blob). ✅
- **GitHub Pages compatible**: Static HTML/CSS/JS only, no server-side rendering or API routes. ✅
- **Vendor files from `/tools/vendor/`**: pdf-lib and PDF.js are already vendored. JSZip must be verified and added to the vendor sync script if absent. ✅ (pending JSZip check)
- **ES Modules only**: `main.js` will use ES module syntax; no CommonJS. ✅
- **Shared styles from `common.css`**: Tool will import `/tools/common.css` for shared tokens and use a tool-specific `style.css` for local overrides. ✅

## Project Structure

### Documentation (this feature)

```text
specs/003-pdf-merger-splitter/
├── spec.md      ← feature specification
├── plan.md      ← this file
└── tasks.md     ← implementation task list
```

### Source Code

```text
tools/
├── pdf-merger-splitter.html          ← tool entry page (from template.html)
└── pdf-merger-splitter/
    ├── main.js                       ← ES module; all tool logic
    └── style.css                     ← tool-specific CSS (imports common.css tokens)

tools/vendor/
├── pdf-lib/                          ← already vendored (reused from PDF Redactor)
├── pdfjs/                            ← already vendored (reused from PDF Redactor)
└── jszip/                            ← add if not already present
```

**Structure Decision**: Single-file tool following the established `/tools/[feature-slug].html` + `/tools/[feature-slug]/main.js` + `/tools/[feature-slug]/style.css` pattern used by the PDF Redactor and Video Trimmer. No new directories beyond the tool subfolder.

## Component Design

### UI Sections (tab-based or scroll-based)

Three logically separate modes surfaced via a tab bar:

| Tab | Mode | Key UI Elements |
|-----|------|-----------------|
| Merge | Combine multiple PDFs | Drop zone, sortable file card list, Merge button, progress bar |
| Split | Extract page ranges | Single-file drop zone, range input rows, Split All / Split by Pages buttons |
| Organise | Reorder / delete pages | Thumbnail grid (drag-reorder), delete-per-page, Reset, Download button |

### Data Flow

```
User selects files (File API)
        ↓
ArrayBuffer stored in memory (no upload)
        ↓
pdf-lib loads PDFDocument from buffer
        ↓ (Merge)              ↓ (Split)              ↓ (Organise)
Copy all pages into new doc    Extract page ranges     Render thumbnails (PDF.js + Canvas)
        ↓                             ↓                       ↓
Save as Uint8Array             Save each range          Re-order / delete pages
        ↓                             ↓                       ↓
Blob → createObjectURL         JSZip → Blob             pdf-lib save → Blob
        ↓                             ↓                       ↓
<a download> trigger           <a download> trigger     <a download> trigger
```

### Module Structure (`main.js`)

```
main.js
├── tabController       — switches active tab, resets state between modes
├── merge/
│   ├── fileListState   — ordered array of { name, pageCount, buffer }
│   ├── dragSort        — drag-and-drop reorder for file cards
│   └── mergeAction     — pdf-lib merge + download trigger
├── split/
│   ├── rangeState      — array of { start, end } pairs
│   ├── rangeValidation — inline validation, enable/disable Split button
│   └── splitAction     — pdf-lib split + JSZip packaging + download
└── organise/
    ├── thumbnailRenderer  — PDF.js → Canvas thumbnail grid
    ├── pageDragSort       — drag-reorder for page thumbnails
    ├── pageDeleteAction   — removes a page from the working order
    └── organiseExport     — pdf-lib reorder/delete + download
```

### Key Implementation Notes

1. **JSZip vendor check**: Before implementing split, verify `/tools/vendor/jszip/` exists. If not, add it to `package.json` and `scripts/sync-vendor.mjs`, run `npm run vendor:update`, and commit the vendor files.
2. **pdf-lib loading**: Use the UMD bundle already vendored. Access via `window.PDFLib` — consistent with how the PDF Redactor loads it.
3. **PDF.js loading**: Use the already-vendored `pdf.js` worker. Set `pdfjsLib.GlobalWorkerOptions.workerSrc` to the vendored worker path — do not reference a CDN.
4. **Thumbnail rendering**: Render at scale 0.2–0.3 of the full page size to keep memory usage low for large PDFs.
5. **Progress indication**: Use a simple determinate progress bar (updated per page/file processed) to avoid blocking the UI. Use `setTimeout` yields or split processing into microtasks if operations freeze the tab.
6. **Error handling**: Wrap all pdf-lib operations in try/catch. Surface user-facing messages for: corrupt file, password-protected PDF, invalid range, empty file list.
7. **Template bootstrap**: Start from `/tools/template.html`; replace placeholders for title, description, and tool-specific content.

## Complexity Tracking

No constitution violations. All complexity is contained within a single static tool. pdf-lib and PDF.js are already vendored; JSZip is the only potential new vendor dependency.
