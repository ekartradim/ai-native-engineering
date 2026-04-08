---
title: "feat: API Status Dashboard — CE Workflow Demo"
type: feat
status: active
date: 2026-04-08
origin: docs/brainstorms/2026-04-08-ce-demo-dashboard-requirements.md
---

# feat: API Status Dashboard — CE Workflow Demo

## Overview

Build a self-contained API status dashboard in a fresh standalone repo. The project exists to demo the compound engineering workflow (`/ce:work` → `/ce:review` → browser agent → PR) for an engineer/tech-lead audience. Speed and visual impact matter more than the code itself.

## Problem Frame

We need a live demo that shows Claude Code autonomously executing a multi-step engineering workflow — task-list progression, incremental commits, parallel review agents catching real issues, browser verification of visual output, and PR creation. The dashboard is the vehicle; the workflow is the product. (see origin: docs/brainstorms/2026-04-08-ce-demo-dashboard-requirements.md)

## Requirements Trace

- R1. Fresh standalone git repo in `demo-api-dashboard/`
- R2. Single-page dashboard with service health cards (name, status badge, response time, last-checked)
- R3. Mock data engine — no real network calls, randomized but realistic
- R4. Visual polish — color-coded badges (green/amber/red), responsive card grid
- R5. Testable status-classification and formatting logic with `node --test`
- R6. Intentional review bait — magic numbers, unused variable, loose equality
- R7. This plan document itself

## Scope Boundaries

- No real API calls, backend server, or database
- No npm, no bundlers, no frameworks — vanilla HTML/CSS/JS only
- No deployment — demo ends at PR creation
- No Python

## Key Technical Decisions

- **Vanilla JS with ES modules**: Use `<script type="module">` in the HTML and standard ES module exports in JS files. Node.js test runner supports ESM natively.
- **Node.js built-in test runner**: `node --test` (Node 18+) — zero dependencies, runs in <2 seconds.
- **Status thresholds**: healthy < 200ms, degraded 200–500ms, down >= 500ms or non-'up' status. These are intentionally left as magic numbers in the classification function (review bait — R6).
- **Review bait placement**: Woven into Units 1 and 3 naturally, not isolated in a separate unit. This ensures review agents encounter them in realistic context.

## Open Questions

### Resolved During Planning

- **Threshold values**: healthy < 200ms, degraded 200–500ms, down >= 500ms. Chosen for clear visual separation in the demo.
- **Which code smells to plant**: (1) magic number thresholds inline in `classifyStatus()`, (2) unused `let lastRefresh = null` in `app.js`, (3) `==` instead of `===` in one status comparison. These are known review bait — reviewers should flag all three.

### Deferred to Implementation

- Exact mock service names and response time distributions — cosmetic, decide at implementation time
- CSS color palette specifics — choose visually appealing values during implementation

## Implementation Units

- [ ] **Unit 1: Project scaffold + status logic module**

  **Goal:** Initialize the standalone repo and create the core status logic — the testable heart of the project.

  **Requirements:** R1, R3, R5, R6

  **Dependencies:** None

  **Files:**
  - Create: `demo-api-dashboard/` (directory + `git init`)
  - Create: `demo-api-dashboard/src/status.js`
  - Create: `demo-api-dashboard/.gitignore`

  **Approach:**
  - `git init` a fresh repo in `demo-api-dashboard/`
  - `status.js` exports three functions:
    - `classifyStatus(responseTime, status)` — returns `'healthy'` / `'degraded'` / `'down'` using inline magic number thresholds (intentional review bait)
    - `formatResponseTime(ms)` — formats milliseconds to human-readable string
    - `generateMockServices()` — returns an array of ~6 mock service objects with randomized response times and statuses
  - Use `==` instead of `===` in one comparison inside `classifyStatus` (review bait)
  - `.gitignore`: standard Node.js ignores

  **Patterns to follow:**
  - ES module exports (`export function`)
  - Pure functions for classification and formatting (query/command separation)

  **Test scenarios:** (covered in Unit 2)

  **Verification:**
  - `demo-api-dashboard/` exists as a git repo
  - `node -e "import('./demo-api-dashboard/src/status.js')"` loads without errors
  - Functions are importable and callable

- [ ] **Unit 2: Tests for status logic**

  **Goal:** Add tests that validate classification thresholds, formatting, and mock data shape. These run during the demo to show the "tests pass" moment.

  **Requirements:** R5

  **Dependencies:** Unit 1

  **Files:**
  - Create: `demo-api-dashboard/src/status.test.js`

  **Approach:**
  - Use `node:test` and `node:assert` (built-in, zero deps)
  - Test suites:
    - `classifyStatus`: healthy at 100ms, degraded at 300ms, down at 600ms, down when status is not 'up'
    - `formatResponseTime`: formatting for various values (0ms, 150ms, 1200ms)
    - `generateMockServices`: returns array, each item has required shape (name, responseTime, status, lastChecked)

  **Patterns to follow:**
  - `import { describe, it } from 'node:test'` + `import assert from 'node:assert/strict'`
  - Descriptive test names that read as behavior specifications

  **Test scenarios:**
  - `classifyStatus(100, 'up')` → `'healthy'`
  - `classifyStatus(300, 'up')` → `'degraded'`
  - `classifyStatus(600, 'up')` → `'down'`
  - `classifyStatus(50, 'error')` → `'down'` (non-'up' status overrides)
  - `formatResponseTime(1200)` → includes 's' or 'ms' appropriately
  - `generateMockServices()` returns 6 items, each with `name`, `responseTime`, `status`, `lastChecked`

  **Verification:**
  - `node --test demo-api-dashboard/src/status.test.js` — all tests pass, zero failures

- [ ] **Unit 3: Dashboard page with visual polish**

  **Goal:** Build the visual dashboard that the browser agent will screenshot and verify. This is the "wow" moment of the demo.

  **Requirements:** R2, R4, R6

  **Dependencies:** Unit 1

  **Files:**
  - Create: `demo-api-dashboard/index.html`
  - Create: `demo-api-dashboard/src/styles.css`
  - Create: `demo-api-dashboard/src/app.js`

  **Approach:**
  - `index.html`: minimal HTML shell, loads `styles.css` and `app.js` as ES module
  - `app.js`:
    - Imports `generateMockServices` and `classifyStatus` from `status.js`
    - Renders a header ("API Status Dashboard") and a card grid
    - Each card shows: service name, colored status badge, response time, last-checked time
    - Declare `let lastRefresh = null` at the top but never use it (review bait — unused variable)
  - `styles.css`:
    - CSS Grid for responsive card layout (auto-fit, minmax)
    - Status badge colors: green for healthy, amber for degraded, red for down
    - Dark or light theme with clean typography (system font stack)
    - Subtle card shadows and rounded corners for visual polish

  **Patterns to follow:**
  - Semantic HTML (`<main>`, `<article>`, `<header>`)
  - CSS custom properties for the color palette
  - ES module import in `<script type="module">`

  **Test scenarios:** (browser agent verifies visually)
  - Dashboard renders 6 service cards
  - Each card has a visible colored status badge
  - Layout is responsive and doesn't overflow

  **Verification:**
  - Opening `index.html` in a browser shows the dashboard with 6 status cards
  - Cards have color-coded badges matching their status
  - Page looks polished enough for a presentation screenshot

## System-Wide Impact

Minimal — this is a standalone greenfield project with no external integrations, no shared state, and no production implications.

## Risks & Dependencies

- **Node.js version**: `node --test` requires Node 18+. Verify at start of `/ce:work`.
- **Browser agent file access**: Browser agent needs to open a local `file://` URL. If this fails, fall back to a simple HTTP server (`npx serve` or `python -m http.server`).

## Review Bait Summary (for demo awareness)

These are intentionally planted for the review step. Do not fix them during `/ce:work`:

| Bait | Location | Expected reviewer |
|------|----------|-------------------|
| Magic numbers `200`, `500` as thresholds | `src/status.js` → `classifyStatus()` | maintainability-reviewer |
| `==` instead of `===` | `src/status.js` → `classifyStatus()` | correctness-reviewer |
| Unused `let lastRefresh = null` | `src/app.js` | maintainability-reviewer |

## Sources & References

- **Origin document:** [docs/brainstorms/2026-04-08-ce-demo-dashboard-requirements.md](docs/brainstorms/2026-04-08-ce-demo-dashboard-requirements.md)
- Node.js test runner: built-in since Node 18, `node:test` module
