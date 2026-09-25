# Jam session — 2026-09-25 · Swarm overview mockups

Operator asked for a page with live mockups of the ten approaches in
`docs/musings/swarm-overview-legibility.md` — one tab per design, and every
mockup must show motion/transition states, not static images.

## Approach

- Single self-contained HTML page: `docs/mockups/swarm-overview-mockups.html`.
  No build step, no dependencies, so it runs inside bb's inline-vis iframe.
- One simulated sashay swarm (parent thread + six children) on a ~64s scripted
  loop drives all ten views, so every mockup shows the same story: staggered
  starts, a first-out error at ~12s (triage-failures), a needs-input at ~16s
  (update-docs), completions, a stalled-then-failed review-diff, recovery.
- Each view adds its own continuous motion: band morphs (1), recede-on-done
  fades (2), ISA-18.2-style flash + acknowledge + first-out flag (3), animated
  pipe dashes with rate-driven speed and valve states (4), rate-scaled
  ribbons (5), belt stop-on-error (6), filling tanks with bubbles (7),
  auto-panning main view with clickable always-legible mimic strip (8),
  scrolling event ticks with a stall detector (9), trains frozen by error
  signals (10).

## 2026-09-25 14:10 — Built v1

- What: full page, ten tabs, shared sim, pause/restart controls.
- Result: verified. Caught one real bug via a linkedom smoke test
  (`legend`/`C` used before definition — would have broken the page on
  load); fixed, plus viewport-rect alignment in the mimic strip, frozen
  trains cleared on finished tracks, seen-counter resets on scenario
  restart. Smoke test runs the page script in a DOM shim and fast-forwards
  2+ scenario loops: all ten views render live state, first-out flash
  appears at ~12s, stall tag flags review-diff at t=30, loop reset clean.
- Next: operator feedback. Musing shortlist was 1 + 2 + 8 as the short path.
