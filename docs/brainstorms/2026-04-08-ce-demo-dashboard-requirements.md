---
date: 2026-04-08
topic: ce-demo-dashboard
---

# Compound Engineering Demo: API Status Dashboard

## Problem Frame

We need a self-contained demo project that showcases the compound engineering workflow (`/ce:work` -> `/ce:review` -> browser agent -> PR) for a presentation to engineers and tech leads. The demo must run fast (~5-8 min), be visually impressive, and exercise multiple agent types (review personas, browser agent). The project itself is secondary — the workflow execution is the star.

## Requirements

- R1. **Standalone project**: Fresh git repo in a new directory (`demo-api-dashboard/`) outside the current repo, initialized with git
- R2. **API status dashboard page**: Single-page HTML+CSS+JS dashboard showing a grid of service health cards with: service name, status indicator (up/down/degraded), response time, last-checked timestamp
- R3. **Mock data engine**: JS module that generates realistic mock API status data (no real network calls) with randomized response times and occasional failures — keeps the demo deterministic and fast
- R4. **Visual polish**: Color-coded status badges (green/amber/red), responsive card grid, clean typography — must look good when the browser agent screenshots it
- R5. **Testable logic**: Extract status-classification logic (healthy/degraded/down thresholds) and data-formatting functions into a separate module with Node.js test runner tests (`node --test`)
- R6. **Intentional review bait**: Include 1-2 minor code quality issues (e.g., a magic number threshold, an unused variable) so review agents have something non-trivial to catch and autofix — makes the review step visually interesting
- R7. **Pre-written plan**: A complete `/ce:plan`-style plan document with 3-4 implementation units, ready to hand to `/ce:work`

## Success Criteria

- The full demo sequence (`/ce:work` -> `/ce:review` -> browser test -> PR) completes in under 10 minutes
- At least 3 review agents spawn during `/ce:review` and produce visible findings
- Browser agent opens the dashboard page, takes a screenshot, and confirms it renders correctly
- A PR is created with a proper description summarizing the changes
- The audience can see the task list progressing in real-time (the checklist-in-motion pattern)

## Scope Boundaries

- No real API calls or backend server — mock data only
- No build tools, bundlers, or frameworks — vanilla HTML/CSS/JS
- No deployment step — the demo ends at PR creation
- No database, auth, or complex state management
- Python is not used — this is a pure frontend demo for visual impact

## Key Decisions

- **Vanilla JS over frameworks**: No React/Vue — keeps the demo instant (no npm install, no build step). The focus is the workflow, not the tech stack.
- **Node.js test runner**: Built-in `node --test` (Node 18+) — zero dependencies, tests run in <2 seconds.
- **Standalone repo**: Fresh project avoids confusion with existing repo files during the presentation.
- **Review bait is intentional**: We plant minor issues so the review step is demonstrably useful, not a rubber stamp. The plan document will note these as "known" so we don't lose track.

## Outstanding Questions

### Deferred to Planning

- [Affects R5][Technical] Exact threshold values for healthy/degraded/down classification — decide during implementation
- [Affects R6][Technical] Which specific code smells to plant — decide during plan writing to ensure review agents detect them

## Next Steps

-> `/ce:plan` for structured implementation planning (or pre-write the plan manually since this is a demo)
