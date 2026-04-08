---
title: "feat: Unify Bull brand HTML presentation slides"
type: feat
status: completed
date: 2026-04-08
---

# feat: Unify Bull brand HTML presentation slides

## Overview

Standardize 9 existing HTML presentation slides to follow Bull brand guidelines consistently, extract shared CSS, add Tosh A headline font, and create a title slide and closing Q&A slide. The result is a polished 11-slide deck for "Compound Engineering Systems in Practice" — the first presentation from the AI Engineering initiative.

## Problem Frame

The 9 slides in `artifacts_bull_brand/` were created independently and have accumulated significant visual drift: 6 different background blues, headline sizes ranging 14–48px, inconsistent font choices (one slide uses DM Sans), varied padding/spacing, mixed logo placement, and no shared design tokens. They need to look like one cohesive deck.

## Requirements Trace

- R1. All slides use the Bull brand color palette via shared CSS variables
- R2. Headlines use Tosh A font; body text uses Roboto
- R3. Fixed 1280x720 dimensions across all slides
- R4. Minimalistic white Bull symbol in a consistent corner position where space allows
- R5. Tree monogram as subtle decorative watermark (lower-right, low opacity)
- R6. New title slide (slide 00) with Bull logo, presentation title, and initiative name
- R7. New closing slide (slide 10) with "Thank you" and "Ask me anything" prompt
- R8. Slide 08 restyled from white/blue dual-panel to unified dark blue
- R9. Consistent typography scale, spacing, and accent patterns across all slides
- R10. Shared CSS file for brand tokens and common patterns

## Scope Boundaries

- Content of existing slides stays unchanged — only visual styling and brand alignment
- No slide navigation, transitions, or JavaScript interactivity
- No responsive/adaptive layout — fixed 1280x720 for projection
- No print stylesheet

## Context & Research

### Reference Presentation

The presentation at `D:\Playground\new-skill-demo\bull-brand-workspace\iteration-2\eval-2-presentation\with_skill\outputs\presentation.html` demonstrates the target quality:
- White Bull symbol (150px) at bottom-left on content slides, bottom-center on closing
- Tree monogram watermark at bottom-right (380px, 0.07–0.12 opacity)
- Tosh A for headlines via @font-face with base64-encoded woff2
- Full Bull CSS variable palette in `:root`
- Consistent 80px/120px padding, 12px border-radius on cards

### Current Slide Inventory

| # | File | Layout | Key Issues |
|---|------|--------|------------|
| 01 | `01_claim_slide.html` | 2-panel | Font weight 900, non-standard padding |
| 02 | `02_tiny_gains_slide.html` | 2-panel | **DM Sans font**, dark bg #001340, 16px radius |
| 03 | `03_compound-engineering-loop.html` | Centered canvas | Has animations, dot-grid bg, good CSS vars |
| 04 | `04_ai_native_mindset_slide.html` | Fragment-style | Minimal structure, needs full slide wrapper |
| 05 | `05_compound_memory_slide.html` | 2-column grid | Decorative rings, 56px padding |
| 06 | `06_memory_lifecycle_slide.html` | 2-column + strip | Decorative rings, varied margins |
| 07 | `07_skill_creator_bull_slide.html` | Timeline steps | Already uses Bull symbol in header |
| 08 | `08_safe-ai-coding-slide.html` | **White/blue dual panel** | Must unify to dark blue |
| 09 | `09_datasentics_plugin_repo_before_last_slide.html` | 2-panel flex | Uses clamp(), percentage padding |

### Brand Assets

- **Bull Symbol (white):** `Bull Symbol/SVG/Bull_symbol_white.svg` — viewBox 0 0 600 574, single path
- **Tosh A fonts:** `assets/fonts/ToshA-{Light,Medium,Black}.woff2` — for @font-face embedding
- **Tree monogram:** Extract from existing slides (05, 06 have it) or reference presentation

### Institutional Learnings

- Bull brand skill defines the full design system: colors, typography, logo rules, tone
- Previous presentation used base64-encoded Tosh A woff2 in @font-face for portability

## Key Technical Decisions

- **Shared CSS file over inline styles:** Extract `slide-base.css` with brand tokens, typography, logo placement, and common patterns. Each slide imports it and adds only slide-specific styles. Rationale: single source of truth, easier future edits.
- **Tosh A via @font-face with local woff2 files** (not base64): Slides will reference relative paths to font files copied into `artifacts_bull_brand/assets/fonts/`. This keeps HTML files readable. If portability becomes a concern later, base64 can be inlined.
- **Bull symbol bottom-left at ~40px from edges, ~32–40px wide:** Minimalistic, unobtrusive, consistent with reference presentation pattern but scaled down for a smaller, subtler presence per user preference.
- **Tree monogram watermark bottom-right, 300–380px, opacity 0.06–0.10:** Decorative brand element, pointer-events: none, behind content.
- **Standardized typography scale:** Eyebrow 10px/600/uppercase/2.5px tracking, Headlines 28–36px Tosh A Black, Body 14–16px Roboto 400, Small labels 11–12px Roboto 500.
- **Standardized spacing:** Slide padding 60px, card border-radius 10px, accent bar 4px Bull Orange.
- **Background:** Dream in Blue #002870 for all slides. Subtle gradient variations allowed per slide for depth.

## Open Questions

### Resolved During Planning

- **Tosh A or Roboto for headlines?** → Tosh A (brand-compliant), user confirmed
- **Slide 08 layout?** → Unify to dark blue, user confirmed
- **Shared CSS vs self-contained?** → Shared CSS, user confirmed
- **Fixed or responsive dimensions?** → Fixed 1280x720, user confirmed
- **Logo style?** → Minimalistic Bull symbol (no text), bottom-left corner, user requested

### Deferred to Implementation

- **Exact sizing of content within each slide** — will be tuned visually during implementation using browser screenshots
- **Whether file 03 animations should be preserved** — likely yes, but verify they work with new shared styles
- **Tree monogram SVG source** — extract from existing slides or copy from brand assets

## High-Level Technical Design

> *This illustrates the intended approach and is directional guidance for review, not implementation specification. The implementing agent should treat it as context, not code to reproduce.*

```
artifacts_bull_brand/
├── assets/
│   └── fonts/
│       ├── ToshA-Light.woff2
│       ├── ToshA-Medium.woff2
│       └── ToshA-Black.woff2
├── slide-base.css          ← NEW: shared brand tokens + common patterns
├── 00_title_slide.html     ← NEW: title slide
├── 01_claim_slide.html     ← RESTYLE
├── 02_tiny_gains_slide.html ← RESTYLE (fix font, colors)
├── 03_compound-engineering-loop.html ← RESTYLE
├── 04_ai_native_mindset_slide.html ← RESTYLE (add full slide wrapper)
├── 05_compound_memory_slide.html ← RESTYLE
├── 06_memory_lifecycle_slide.html ← RESTYLE
├── 07_skill_creator_bull_slide.html ← RESTYLE
├── 08_safe-ai-coding-slide.html ← RESTYLE (major: remove white panel)
├── 09_datasentics_plugin_repo_before_last_slide.html ← RESTYLE
└── 10_closing_slide.html   ← NEW: thank you + Q&A
```

**Shared CSS provides:**
- `:root` with full Bull color palette
- `@font-face` for Tosh A weights
- `.slide` base: 1280x720, overflow hidden, position relative, Dream in Blue bg
- `.slide-logo`: absolute bottom-left, 36px wide Bull symbol
- `.slide-watermark`: absolute bottom-right tree monogram, low opacity
- `.eyebrow`: standardized label styling
- `.headline`: Tosh A, standardized scale
- `.accent-bar`: 4px Bull Orange horizontal rule

## Implementation Units

- [ ] **Unit 1: Create shared CSS and copy font assets**

  **Goal:** Establish the brand foundation that all slides import.

  **Requirements:** R1, R2, R3, R10

  **Dependencies:** None

  **Files:**
  - Create: `artifacts_bull_brand/slide-base.css`
  - Create: `artifacts_bull_brand/assets/fonts/ToshA-Light.woff2` (copy from brand skill)
  - Create: `artifacts_bull_brand/assets/fonts/ToshA-Medium.woff2` (copy)
  - Create: `artifacts_bull_brand/assets/fonts/ToshA-Black.woff2` (copy)

  **Approach:**
  - Define `:root` CSS variables for the full Bull palette (primary, tints, secondary)
  - Define `@font-face` rules for Tosh A Light/Medium/Black pointing to `assets/fonts/`
  - Define `.slide` base class: 1280x720, Dream in Blue bg, Roboto body, overflow hidden, position relative
  - Define `.slide-logo` class: absolute positioned, bottom-left, white Bull symbol SVG as inline background or child element pattern
  - Define `.slide-watermark` class: absolute bottom-right, tree monogram, opacity 0.08, pointer-events none
  - Define shared component classes: `.eyebrow`, `.headline`, `.sub-headline`, `.accent-bar`, `.card`
  - Define typography scale using CSS variables for consistent sizes

  **Patterns to follow:**
  - Reference presentation's `:root` variable block
  - Reference presentation's `.slide-logo` positioning (bottom: 40px, left: 60px)

  **Test scenarios:**
  - Importing the CSS in a blank HTML file shows correct font loading
  - CSS variables resolve correctly in browser dev tools
  - Logo and watermark position correctly in a 1280x720 container

  **Verification:**
  - A minimal test HTML file importing `slide-base.css` renders with correct fonts, colors, and positioned logo/watermark

- [ ] **Unit 2: Create title slide (00)**

  **Goal:** New opening slide with Bull logo, presentation title, and initiative attribution.

  **Requirements:** R6, R1, R2, R3, R4, R5

  **Dependencies:** Unit 1

  **Files:**
  - Create: `artifacts_bull_brand/00_title_slide.html`

  **Approach:**
  - Import `slide-base.css`
  - Centered layout with vertical stacking: Bull logo (larger, ~120px symbol) at top, title text below, initiative name at bottom
  - Title: "Compound Engineering Systems in Practice" in Tosh A Black, white, large (40–48px)
  - Subtitle/attribution: "AI Engineering Initiative" in Roboto, lighter weight, orange or white muted
  - Optional: "DataSentics, a Bull company" small footer text
  - Tree monogram watermark in background
  - Clean, minimal — let the logo and title breathe
  - Consider a subtle accent bar between title and subtitle

  **Patterns to follow:**
  - Reference presentation's title slide (slide--title): centered text, accent bar, logo at bottom

  **Test scenarios:**
  - Title is legible and well-spaced at 1280x720
  - Bull symbol renders correctly in white
  - Typography hierarchy is clear (title > subtitle > attribution)

  **Verification:**
  - Screenshot shows clean, branded title slide with correct fonts, colors, and logo

- [ ] **Unit 3: Create closing slide (10)**

  **Goal:** Thank-you and Q&A slide to close the presentation.

  **Requirements:** R7, R1, R2, R3, R4, R5

  **Dependencies:** Unit 1

  **Files:**
  - Create: `artifacts_bull_brand/10_closing_slide.html`

  **Approach:**
  - Import `slide-base.css`
  - Centered layout: "Thank You" headline in Tosh A, large
  - Below: "Ask me anything" or similar invitation in Roboto, lighter
  - Bull symbol centered at bottom (following reference presentation's closing slide pattern)
  - Background: Bull Orange (#FF5539) for energy and contrast, or Dream in Blue for consistency — follow whichever feels right during implementation
  - Tree monogram watermark
  - Minimal content — this slide should feel open and inviting

  **Patterns to follow:**
  - Reference presentation's closing slide: orange background, centered text, bottom-center logo

  **Test scenarios:**
  - Text is centered and well-proportioned
  - Feels like a natural conclusion to the deck

  **Verification:**
  - Screenshot shows branded closing slide with clear "Thank You" and Q&A invitation

- [ ] **Unit 4: Restyle slides 01–03**

  **Goal:** Align the first three content slides to the shared brand system.

  **Requirements:** R1, R2, R3, R4, R5, R8, R9

  **Dependencies:** Unit 1

  **Files:**
  - Modify: `artifacts_bull_brand/01_claim_slide.html`
  - Modify: `artifacts_bull_brand/02_tiny_gains_slide.html`
  - Modify: `artifacts_bull_brand/03_compound-engineering-loop.html`

  **Approach:**
  - Add `<link rel="stylesheet" href="slide-base.css">` to each
  - Remove duplicate CSS variable definitions, use shared ones
  - **Slide 01:** Replace inline font styles with Tosh A headlines, standardize padding, add Bull symbol logo
  - **Slide 02 (critical):** Replace DM Sans with Roboto/Tosh A, fix background to #002870, standardize border-radius to 10px, add logo
  - **Slide 03:** Preserve animations but migrate color variables to shared ones, replace custom `:root` vars with standard names, add logo if space allows
  - For each: add `.slide-watermark` tree monogram if not present
  - Standardize eyebrow label styling to shared `.eyebrow` class
  - Ensure 1280x720 fixed dimensions

  **Patterns to follow:**
  - Shared CSS classes from Unit 1
  - Reference presentation's accent bar and card patterns

  **Test scenarios:**
  - Slide 02 no longer loads DM Sans
  - All three slides show consistent background blue
  - Animations on slide 03 still work
  - Logo appears in bottom-left corner

  **Verification:**
  - Side-by-side browser screenshots show consistent brand language across all three slides

- [ ] **Unit 5: Restyle slides 04–06**

  **Goal:** Align middle content slides to the shared brand system.

  **Requirements:** R1, R2, R3, R4, R5, R9

  **Dependencies:** Unit 1

  **Files:**
  - Modify: `artifacts_bull_brand/04_ai_native_mindset_slide.html`
  - Modify: `artifacts_bull_brand/05_compound_memory_slide.html`
  - Modify: `artifacts_bull_brand/06_memory_lifecycle_slide.html`

  **Approach:**
  - Add shared CSS import to each
  - **Slide 04:** Add full HTML5 slide wrapper if currently a fragment, standardize to 1280x720, apply brand typography, add logo
  - **Slide 05:** Migrate hardcoded hex to CSS variables, standardize decorative rings to shared pattern, ensure logo placement
  - **Slide 06:** Same as 05 — migrate colors, standardize rings, add logo, align typography scale
  - Standardize padding to ~60px across all three
  - Replace any hardcoded color values with `var(--bull-orange)`, `var(--dream-in-blue)`, etc.

  **Patterns to follow:**
  - Existing ring decorations on slides 05/06 can stay but should use consistent sizing and brand colors

  **Test scenarios:**
  - Slide 04 renders as a proper full-size slide
  - Decorative rings on 05/06 use brand-consistent colors
  - Typography hierarchy matches other slides

  **Verification:**
  - Browser screenshots confirm consistent styling with slides 01–03

- [ ] **Unit 6: Restyle slides 07–09**

  **Goal:** Align final content slides to the shared brand system, including the major restyle of slide 08.

  **Requirements:** R1, R2, R3, R4, R5, R8, R9

  **Dependencies:** Unit 1

  **Files:**
  - Modify: `artifacts_bull_brand/07_skill_creator_bull_slide.html`
  - Modify: `artifacts_bull_brand/08_safe-ai-coding-slide.html`
  - Modify: `artifacts_bull_brand/09_datasentics_plugin_repo_before_last_slide.html`

  **Approach:**
  - Add shared CSS import to each
  - **Slide 07:** Already uses Bull symbol in header — migrate to shared logo pattern (bottom-left), standardize colors and typography
  - **Slide 08 (major restyle):** Remove white left panel, unify to Dream in Blue background, reorganize content to work on dark background (white text, orange accents), preserve the safety layers content and feedback loop diagram, adapt SVG diagram colors for dark bg
  - **Slide 09:** Replace clamp() responsive sizing with fixed 1280x720, migrate percentage padding to px, standardize colors
  - Ensure all three have consistent eyebrow/headline/body typography

  **Patterns to follow:**
  - Other slides' single-background approach for slide 08 rework

  **Test scenarios:**
  - Slide 08 SVG diagram is legible on dark background
  - Slide 08 content hierarchy preserved despite layout change
  - Slide 09 renders correctly at fixed dimensions (no clamp artifacts)

  **Verification:**
  - All three slides visually consistent with the rest of the deck
  - Slide 08 no longer has white panel

- [ ] **Unit 7: Visual QA pass with agent-browser**

  **Goal:** Open each slide in the browser, take screenshots, and verify end-to-end consistency.

  **Requirements:** All (R1–R10)

  **Dependencies:** Units 1–6

  **Files:**
  - All 11 slide HTML files

  **Approach:**
  - Use agent-browser to open each slide at 1280x720 viewport
  - Screenshot each slide
  - Compare for: consistent background color, typography scale, logo placement, accent colors, spacing
  - Fix any remaining drift discovered during visual review
  - Verify Tosh A font renders correctly (not falling back to sans-serif)
  - Verify tree monogram watermark appears subtly on all applicable slides

  **Test scenarios:**
  - All 11 slides show Dream in Blue (or Bull Orange for closing) backgrounds
  - Bull symbol appears in correct position on all slides
  - No DM Sans or fallback fonts visible
  - Headlines are visually Tosh A (distinct from Roboto body text)
  - Consistent spacing and padding across the deck

  **Verification:**
  - Full set of 11 screenshots demonstrates a cohesive, professionally branded presentation

## System-Wide Impact

- **File structure change:** New `assets/fonts/` directory and `slide-base.css` added to `artifacts_bull_brand/`
- **Font dependency:** Slides now require Tosh A woff2 files to be co-located in `assets/fonts/`
- **Breaking portability:** Individual slides can no longer render standalone without `slide-base.css` and `assets/` — this is an acceptable tradeoff for consistency

## Risks & Dependencies

- **Tosh A font loading:** If woff2 files fail to load, headlines will fall back to Roboto. Mitigation: test font loading explicitly in Unit 7.
- **Slide 08 restyle complexity:** Converting from white/blue dual-panel to single dark background may require significant content reorganization. Mitigation: preserve content structure, change only colors and backgrounds.
- **Slide 03 animations:** Changing CSS variable names could break animation keyframes. Mitigation: test animations explicitly after migration.
- **Slide 04 fragment structure:** May need significant HTML restructuring to become a proper standalone slide. Mitigation: wrap existing content in standard slide container.

## Sources & References

- **Brand skill:** `C:\Users\radim\.claude\plugins\cache\local-curated\compound-engineering\2.52.0\skills\bull-brand\`
- **Reference presentation:** `D:\Playground\new-skill-demo\bull-brand-workspace\iteration-2\eval-2-presentation\with_skill\outputs\presentation.html`
- **Bull Symbol SVG (white):** `Bull Symbol/SVG/Bull_symbol_white.svg`
- **Tosh A fonts:** `assets/fonts/ToshA-{Light,Medium,Black}.woff2`
