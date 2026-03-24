# CVO Design Decision Log: Ledgea Logo System

**Date:** 2026-03-24
**Project:** Ledgea Brand Identity v1.0
**Decision Type:** Logo System Creation

## Design Rationale

### Icon Mark (Symbol)
- **3-layer cards**: Three rounded rectangles offset diagonally (top-right to bottom-left), creating depth through progressive teal gradation (#CCFBF1 -> #5EEAD4 -> #0D9488)
- **Ledger lines**: Three horizontal lines on the front card at varying lengths, evoking structured financial data
- **Anchor integration**: Abstract anchor form below the card stack -- ring + vertical stem + crossbar with curved flukes. Communicates stability and grounding without literal nautical imagery
- **Key tension resolved**: Anchor detail vs. small-size legibility. Chose simplified geometry (1.5px stroke minimum) to survive 16px rendering

### Typography
- Plus Jakarta Sans, weight 600 for wordmark
- Negative letter-spacing (-0.5) for tighter, more premium feel
- system-ui fallback chain for cross-platform reliability

### Color Application
- Primary Teal #0D9488: Front card, tagline text
- Dark #134E4A: Wordmark text, anchor strokes
- Mid #5EEAD4: Middle card layer
- Light #CCFBF1: Back card layer
- White variants use opacity layering (0.15, 0.35, 1.0) for depth on dark backgrounds

## Deliverables
1. `ledgea-icon.svg` -- 32x32 favicon/app icon
2. `ledgea-icon-white.svg` -- White variant for dark backgrounds
3. `ledgea-full-horizontal.svg` -- 300x60 header logo with tagline
4. `ledgea-full-horizontal-white.svg` -- White variant
5. `ledgea-full-vertical.svg` -- 160x130 LP/hero logo with tagline
6. `ledgea-full-vertical-white.svg` -- White variant
7. `preview.html` -- Brand sheet for visual QA

## Quality Assessment
- Scalability: Icon tested at 16/24/32/48/64px -- all legible
- Consistency: All 6 SVG files share identical icon geometry via same coordinate values
- Brand coherence: Teal palette + card metaphor + anchor = "structured trust for complex markets"
