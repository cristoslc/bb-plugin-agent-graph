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

## 2026-09-25 14:40 — Observed in a real browser, fixed rendering

- What: operator reported the andon misrendering and several views not
  scaling with the viewport. No desktop browser is attached to this session,
  so drove headless Chromium via the cached Playwright install and screenshotted
  every tab (added a `__seek(t)` hook to jump the sim to exact moments instead
  of waiting real time).
- Found: andon's six fixed-width stations + board overflowed the panel
  (clipped off the right edge); fixed 640px canvases (trees 1/2, pipes 4,
  sankey 5, tracks 10, vessels 7) sat in a horizontal scroll strip instead of
  shrinking; mimic strip 8 overflowed below ~650px.
- Fixed: andon is fluid flex (stations share width, labels ellipsize); SVG
  views got viewBox + fluid width; DOM canvases got a ResizeObserver scaler
  (`responsive()`); m7/m8 wrap their fixed panes; annunciator names ellipsize.
  Re-screenshotted narrow (620px), wide (1600px), andon normal + alarm: all
  correct, zero page errors.
- Also captured: operator's point that parent/child has two nests
  (thread→turns→actions vs parent thread→child threads) — added to the musing.

## 2026-09-25 14:55 — Centered layout pass

- What: operator reported sections hug left with no max-width; instruments
  not centered.
- Fixed: sections capped at 960px and centered; stage is now a centering
  flex column; header/tab bar/footer share the same column; every inner
  element (slider bar, legend, annunciator grid + note, trend lanes) gets
  explicit width:100% + max-width:640px so it centers with its canvas.
- Verified: screenshots at 1600px (tabs 1/3/6/7/8/9) and 620px (1/6/9),
  zero page errors, smoke suite green.

## 2026-09-25 15:20 — Not there yet: stage geometry + replay break

- What: operator still saw off layout (screenshot at ~1000px), and tab
  switching broke the replay (manual restart needed).
- Found (two real bugs):
  1. The old `.stage{overflow-x:auto}` rule still existed below the new flex
     rule, and the 960px section let a 640px instrument float with ~160px
     dead panel each side — read as "not centered". Stage is now
     `width:fit-content; max-width:100%; margin:0 auto`: the panel hugs its
     instrument and the pair centers together. Tabs/footer caps re-applied
     (a prior edit hadn't survived into the file).
  2. `responsive()` measured canvas height with getBoundingClientRect on
     first observation. For a section first observed while `display:none`,
     that's 0 → it applied scale(0) and cached natH=0; the stage stayed
     blank and the replay appeared dead. Fixed: use offsetHeight
     (transform-independent), skip observation when clientWidth < 2, never
     apply a ~0 scale.
- Andon rework per operator: LINE STOPPED is global (andon-cord semantics —
  any station stops the line) and now sits as a status strip directly above
  the conveyor, naming the culprit station(s): "● LINE STOPPED —
  triage-failures", with "· acknowledged" appended after ACK. The belt row
  (status strip, stations, belt) is a flex column; stations align to the
  belt.
- Verified with journey tests, not just stills: fresh-visit all 10 tabs
  (no blank/scaled-to-zero stages), replay crossing the 64s rollover while
  switching tabs (clock wraps to t+1.6s, first-out cleared), culprit strip
  text correct, screenshots at 1000px and 1600px. Zero page errors.
