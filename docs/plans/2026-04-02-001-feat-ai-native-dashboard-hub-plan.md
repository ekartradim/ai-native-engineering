---
title: "feat: AI-Native Engineering Dashboard Hub"
type: feat
status: active
date: 2026-04-02
---

# AI-Native Engineering Dashboard Hub

## Overview

Build a static HTML dashboard hub that serves as the central navigation point for all AI-native engineering reference material — tool comparisons, migration guides, architecture diagrams, and methodology docs. Deploy to surge.sh for free, zero-friction hosting.

## Problem Frame

The project contains rich content across two folders (artifacts/ with interactive HTML dashboards, AI-native-tools-compatibility/ with markdown comparisons) but no unified entry point. Users need a single URL to browse all material, organized by relevance and type.

## Requirements Trace

- R1. Hub landing page with cards linking to all sub-dashboards, grouped by type, ordered by importance
- R2. New HTML dashboard pages for markdown-based comparison content (Claude Code vs Cursor, Superpowers vs CE, Migration Guide)
- R3. Existing artifact HTML files accessible via navigation without modification
- R4. Consistent nav bar across all pages for easy cross-navigation
- R5. Deploy to surge.sh with a single command
- R6. Simplicity over engineering — plain HTML + CSS, zero build step

## Scope Boundaries

- No JavaScript frameworks, bundlers, or build steps
- No server-side logic — pure static files
- No modifications to existing artifact HTML files (they stay as-is, linked directly)
- Design refinement deferred to next step (Chrome MCP extension + frontend skill)

## Key Technical Decisions

- **Plain HTML + shared CSS**: One `css/style.css` file shared across new pages. Keeps deployment to `surge ./` with no build step.
- **Artifacts linked, not rewritten**: The 6 artifact HTML files are self-contained with embedded styles. Link to them directly from the hub rather than extracting/reformatting their content.
- **Markdown content converted to HTML pages**: The 3 comparison/migration markdown docs become new styled HTML pages with the shared nav and CSS.
- **Shared nav as HTML snippet**: Copy a consistent nav bar into each new HTML page. No includes system needed for ~5 new files.
- **surge.sh deployment**: Just `npx surge ./` from the project root. Free tier, custom subdomain, no config needed.

## Dashboard Organization

Cards on the hub landing page, grouped by type, ordered top-to-bottom as specified:

### Group 1: AI-Native Methodology (foundations first)
1. **AI-Native Philosophy** — 6 foundational principles/constraints (link to `artifacts/ai_native_philosophy.html`)
2. **AI-Native Workflow** — 6-stage development lifecycle (link to `artifacts/ai_native_workflow.html`)
3. **AI-Native Best Practices** — 19 daily practices across 6 categories (link to `artifacts/ai_native_best_practices.html`)

### Group 2: Architecture & Internals (how the tools work)
4. **Cursor Invocation Graph** — Who calls what: agent → commands → skills → subagents (link to `artifacts/cursor_invocation_graph.html`)
5. **Cursor Harness Architecture** — IDE startup, rules, commands, skills, MCP (link to `artifacts/cusor_harness.html`)
6. **Skill Creator Workflow** — How to create and manage skills (link to `artifacts/skill_creator_workflow.html`)

### Group 3: Tool Comparisons (side-by-side analysis)
7. **Claude Code vs Cursor** — Side-by-side component comparison (new page from `Cursor-vs-Claude-Code.md`)
8. **Superpowers vs Compound Engineering** — Portability analysis & when to use which (new page from `Superpowers-vs-CompoundEngineering.md`)

### Group 4: Migration & Setup (getting started)
9. **CE → Cursor Migration Guide** — Setup symlink bridge, get CE working in Cursor (new page from `README.md`)

## Open Questions

### Resolved During Planning

- **Surge subdomain name?** — Defer to deployment time; user picks at `surge ./` prompt or via `CNAME` file.
- **Fix typo in `cusor_harness.html` filename?** — Yes, renamed to `cursor_harness.html`.
- **Surge subdomain?** — `ai-native-engineering.surge.sh`

### Deferred to Implementation

- **Visual design details** — Will be refined in a subsequent step using Chrome MCP extension and frontend skill. Initial implementation should be clean and readable but doesn't need pixel-perfect design yet.
- **Responsive breakpoints** — Keep layout simple enough that it works on desktop; mobile polish can come in design refinement pass.

## Implementation Units

- [ ] **Unit 1: Project structure and shared CSS**

  **Goal:** Create the file structure and a clean shared stylesheet

  **Requirements:** R5, R6

  **Dependencies:** None

  **Files:**
  - Create: `css/style.css`

  **Approach:**
  - CSS with clean typography, card layout for hub, table styles for comparisons
  - Color scheme: neutral/dark with accent colors per group type
  - Nav bar styles, responsive basics
  - Keep CSS under 200 lines — simplicity first

  **Patterns to follow:**
  - Look at the existing artifact HTML files for visual tone/inspiration (they use dark themes with colored accents)

  **Verification:**
  - CSS file exists, is well-organized, under 200 lines

- [ ] **Unit 2: Hub landing page (index.html)**

  **Goal:** Create the main dashboard with grouped, ordered cards linking to all sub-dashboards

  **Requirements:** R1, R4

  **Dependencies:** Unit 1

  **Files:**
  - Create: `index.html`

  **Approach:**
  - Nav bar at top with links to all pages
  - 4 groups of cards as defined in Dashboard Organization section above
  - Each card: title, short description, link (to new page or artifact file)
  - Cards for artifact links open the HTML file directly
  - Cards for comparison pages link to new HTML pages

  **Test scenarios:**
  - All 9 cards render, grouped correctly
  - Links to artifact files resolve (relative paths)
  - Links to new comparison pages resolve

  **Verification:**
  - Open `index.html` in browser, all cards visible and grouped, all links work

- [ ] **Unit 3: Claude Code vs Cursor comparison page**

  **Goal:** Convert `Cursor-vs-Claude-Code.md` into a styled HTML dashboard page

  **Requirements:** R2, R4

  **Dependencies:** Unit 1

  **Files:**
  - Create: `comparison.html`
  - Source: `AI-native-tools-compatibility/Cursor-vs-Claude-Code.md`

  **Approach:**
  - Shared nav bar for cross-navigation
  - Convert markdown tables to HTML tables with clear styling
  - Preserve all comparison sections and detail
  - Use collapsible sections (`<details>`) for dense subsections to keep page scannable

  **Verification:**
  - All comparison data from source markdown is present
  - Tables are readable, nav works, links back to hub

- [ ] **Unit 4: Superpowers vs CE page**

  **Goal:** Convert `Superpowers-vs-CompoundEngineering.md` into a styled HTML dashboard page

  **Requirements:** R2, R4

  **Dependencies:** Unit 1

  **Files:**
  - Create: `plugins.html`
  - Source: `AI-native-tools-compatibility/Superpowers-vs-CompoundEngineering.md`

  **Approach:**
  - Same structure as Unit 3 — shared nav, tables, collapsible details
  - Highlight the "who should use which" recommendation clearly
  - Include installation comparison prominently

  **Verification:**
  - All content from source markdown present
  - Recommendation section is prominent and scannable

- [ ] **Unit 5: CE → Cursor Migration Guide page**

  **Goal:** Convert `README.md` (CE setup guide) into a styled HTML page

  **Requirements:** R2, R4

  **Dependencies:** Unit 1

  **Files:**
  - Create: `migration.html`
  - Source: `AI-native-tools-compatibility/README.md`

  **Approach:**
  - Step-by-step setup guide with clear numbering
  - Code blocks for symlink commands styled for readability
  - Troubleshooting section with collapsible items
  - Note that CE skills are Claude Code native — this guide helps Cursor users get the same capabilities

  **Verification:**
  - Setup steps are clear and complete
  - Code blocks render properly
  - Nav works, links back to hub

- [ ] **Unit 6: Deploy to surge.sh**

  **Goal:** Deploy the complete dashboard to surge.sh

  **Requirements:** R5

  **Dependencies:** Units 1-5

  **Files:**
  - Optionally create: `CNAME` (if user wants custom subdomain)
  - Optionally create: `200.html` (copy of index.html for surge SPA fallback — not needed for multi-page but good practice)

  **Approach:**
  - Run `npx surge ./` from project root
  - All files including `artifacts/` folder deploy as-is
  - Surge serves static files with correct MIME types
  - User picks subdomain at prompt (e.g., `ai-native-engineering.surge.sh`)

  **Verification:**
  - Site loads at surge URL
  - All 9 dashboard links work
  - Artifact HTML files render correctly when accessed via surge

## System-Wide Impact

- **No build step**: Adding `css/style.css` and new HTML files to the project root. No package.json, no node_modules.
- **Artifact files untouched**: Existing HTML files in `artifacts/` are linked but never modified.
- **Deployment footprint**: ~200KB total (6 artifact HTMLs + 4 new HTMLs + 1 CSS file).

## Risks & Dependencies

- **surge.sh requires npm/npx**: User needs Node.js installed. Alternatively `npm install -g surge` for a global command.
- **Artifact HTML files have embedded styles**: They won't match the hub's nav/theme. This is acceptable — they're self-contained interactive dashboards. The hub provides the unified navigation layer.
- **No hot-reload during development**: Just open HTML files in browser and refresh. Could use `npx serve` for local dev if desired.

## Sources & References

- Artifact HTML files: `artifacts/*.html` (6 interactive dashboards from claude.ai)
- Comparison docs: `AI-native-tools-compatibility/*.md` (4 markdown reference docs)
- surge.sh docs: https://surge.sh/help/getting-started-with-surge
