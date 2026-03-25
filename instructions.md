# VibeCoded Utilities: AI Agent Instructions

Use AGENTS.md as the canonical source of project guidance. This file complements it with implementation style defaults.

## Core Ethos
- 100% free forever.
- 100% client-side processing.
- No uploads, no backend, no accounts.
- Solve the specific user problem with minimal complexity.

## Technical Architecture
- Stack: vanilla HTML5, CSS3, modern JavaScript.
- No runtime build requirement.
- Tool entry pages live in `/tools/*.html`.
- Shared runtime libraries for tool pages live in `/tools/vendor/**`.

## Design + UX Baseline
- Include clear preview/export flow when relevant.
- Include reset/clear/undo-style controls when relevant.
- Include quality/compression/resolution controls when fidelity can vary.
- Include clear validation errors and recovery paths.
- Use descriptive `<title>`, strong meta descriptions, and clear usage copy.

## Shared Styling Conventions
- Include `/tools/common.css` in every tool page.
- Keep shared primitives in `common.css` (tokens, typography, headers, badges, buttons, dropzones, status/error blocks).
- Keep tool-specific styles in `/tools/[feature-slug]/style.css` only.
- Start new tool pages from `/tools/template.html` and replace placeholders.

## Vendor Consolidation Conventions
- Use shared runtime assets from `/tools/vendor/**`.
- Do not duplicate runtime assets under `/tools/[feature-slug]/vendor/` when shared equivalents exist.
- Update shared vendor references rather than introducing per-tool copies.
