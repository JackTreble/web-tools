# Strategic Directives

> Updated by: Human operator (you)
> Read by: All squad leads and agents before every run
> This file overrides any conflicting guidance in SQUAD.md goals

## Mission

We create simple in browser local web tools to provide free alternatives to paid tools

## P0: First Value

**All squads work toward delivering visible, useful output from the very first run.**

Research focus: Commonly searched for tools that can be be implemented in browser for free.

### What "useful" means
- Intelligence produces a brief with sourced facts, not generic summaries
- Research identifies specific competitors and opportunities, not "the market is growing"
- Product translates findings into concrete next steps, not vague roadmaps
- Company evaluates whether outputs actually advanced the business goals

## Decision Framework

1. Does this advance the research focus above?
2. Is there a simpler approach for 80% of the value?
3. What's the opportunity cost — what are we NOT doing?

## Constraints

- Learning phase. Explore, experiment, report findings.
- Don't over-invest in process. Produce output first, optimize later.
- Every agent output should be something the human operator can act on today.

# web-tools Operational Directives

## Directive 0: The Golden Rule
All tools MUST be 100% client-side. If a feature requires a server-side API (excluding CDN-hosted libraries), it must be rejected or redesigned to use a Browser API (e.g., Web Workers, Canvas, Web Crypto).

## Directive 1: Technical Standards
- **Language:** HTML5, CSS3, Modern JavaScript.
- **Architecture:** ES Modules (ESM) only. No bundling steps unless strictly necessary for performance.
- **Dependencies:** Use CDN links for external libraries. Favor vanilla solutions over heavy frameworks.
- **Styling:** Mobile-first design using Tailwind CSS (via Play CDN) or standard CSS variables.

## Directive 2: Agent Workflow (Squad/Spec-Kit)
- **Spec-First:** No code is written until a `spec.md` is approved by the Human Board.
- **Context Preservation:** Every major change must be logged in `decisions.md` to prevent context drift.
- **Hallucination Check:** The `code-reviewer` agent must verify that all suggested NPM packages actually exist and are safe to use via CDN.
- **Duplicate Check:** Before proposing or scoping a new tool, agents must review `/specs/**` plus relevant open and closed GitHub issues.
- **Bounded Discovery:** Research agents should use short discovery passes, avoid long exploratory runs, and prefer a few validated opportunities over many speculative ones.
- **Issue Volume Control:** A single research run should create no more than 3 proposal issues, and only for clearly distinct, validated ideas.

## Directive 2.1: Usability Baseline
- Proposed tools should account for practical user controls when relevant: edit/undo/reset, preview before export, color selection, quality/compression settings, and clear recovery from invalid input.
- Default behaviors should be safe for non-technical users and avoid destructive surprises.

## Directive 3: Distribution
- **Deployment:** Automatic deployment to GitHub Pages on merge to `main`.
- **SEO:** Every tool published on `tools.treble.dev` must have a descriptive `<title>`, useful meta description, relevant meta tags, and a clear "How to use" section in the UI.
- **Search Intent:** Tool names, headings, and copy should reflect plain-language search terms users are likely to use.
- **Discoverability:** When proposing or specifying a tool, agents should note likely target queries and ensure the page can stand on its own as a searchable landing page.

---

*Update this file as your business goals evolve. Run `squads goal set` for squad-level goals.*
