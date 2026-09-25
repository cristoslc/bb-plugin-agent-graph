# Musing: legible swarms at overview zoom

## What sparked this

On sashays, a parent thread spawns a swarm of child threads. On the graph this
shows up as a bounded box: the parent card and its children inside. I like
seeing the whole swarm active at once, in the style of an industrial
flow-control monitor. But to fit the box on screen I zoom out so far that the
cards turn into unreadable smudge. I can see that *something* is happening. I
can't tell *what*.

## The mechanical why (found in the code today)

- Cards are fixed-size divs (`NODE_WIDTH` x `NODE_HEIGHT`) that scale with the
  viewport transform. Text scales with them. At 0.2x a card is ~90px wide; the
  label is noise.
- Zoom is clamped to 0.2..2.5.
- `fit()` in `graph.tsx` floors the scale at 0.4 (compact) / 0.6. So on a big
  swarm, Fit refuses to zoom out far enough, and I end up hand-rolling the
  wheel down to ~0.2. The floor is half the pain: the tool won't take me where
  I need to go, so I go somewhere worse manually.

## The ICS lens

Control-room HMIs solved "many processes, one screen" decades ago. Their
tricks, loosely:

- **Read state, not text, at distance.** Overview displays are lamp-first.
  Labels are for the zoomed-in display.
- **Color means abnormal.** ISA-101 gray-board style: the healthy plant is
  gray; color is spent only on attention states. The eye goes straight to the
  lit exception.
- **Semantic zoom.** Displays degrade gracefully: unit → instrument bubble →
  lamp on a line. Each zoom band is *designed*, not just scaled.
- **Overview beside detail.** An area mimic always visible next to the
  operating display; you navigate by clicking the mimic.
- **State without rate hides stalls.** DCS pairs every state display with a
  trend. A green lamp that stopped being interesting five minutes ago is a
  lie the trend exposes.

The graph today is the opposite bet: it scales text down instead of degrading
into designed states. Ten ways to flip that, ordered loosely: 1-2 make the
existing graph legible when small; 3-7 and 10 change what you're looking at;
8-9 add always-legible surfaces around whichever view wins.

## The ten

### 1. Semantic zoom: cards that morph instead of shrink

Keep the tree. Add scale bands: full card above ~70%; a "faceplate" (icon,
one-line title, status bar) around 40-70%; an instrument bubble (circle,
status ring, tiny glyph) below 40%; below ~20% a lamp on the edge line, 6px.
Edges thicken as nodes simplify. The browser stops scaling text; each band is
drawn for its size.

- ICS analog: unit → ISA instrument bubble → lamp.
- Why it fixes the pain: "what's going on" is status, and status stays legible
  at every zoom.
- Cost: low. Nodes are absolutely-positioned divs; swap a class per scale
  bucket in the render that already knows `view.scale`.

### 2. Gray-board discipline: color means abnormal

Add an "Operations" color mode. Everything neutral gray; running = blue pulse,
needs-input = amber, error = red, done = near-invisible dark. Done work
recedes instead of competing. At overview zoom you don't read labels because
you don't need to: the only lit things are the live ones.

- ICS analog: ISA-101 grayboard, alarm-color-only philosophy.
- Why: legibility at distance is achieved by subtracting noise, not by
  magnifying signal.
- Cost: low. It's a third color mode next to type/status; pairs naturally
  with 1.

### 3. Annunciator matrix mode

An alternate view that swaps the tree for a grid of lamp tiles, one per thread
in the swarm, nested containers for parent groups. Flash = running (ISA-18.2
flash codes), steady amber = waiting on you, red = error, dark = done. The
first failure in a burst gets a "first-out" flag, control-room style, so you
know which child to look at first.

- ICS analog: alarm annunciator panels; tile-per-point with flash codes.
- Why: annunciators are designed to be read from across a room. Zoom stops
  mattering entirely.
- Cost: medium. Same data, second layout mode in the same page.

### 4. Pipe-and-flow mimic (P&ID)

Threads become pipe runs; tool activity animates as flow arrows along the
pipe; state becomes valve symbols (open = running, throttled = waiting,
closed = done). Children branch off the parent header like a manifold. Lines
and arrows stay crisp at 0.2x where cards become mush; the reading becomes
"where is flow moving" instead of "what does this label say".

- ICS analog: P&ID process mimic, the classic SCADA overview.
- Cost: medium-high. Edges take on arrowhead/animation duty; nodes shrink to
  junctions. But it *is* the flow-control monitor fantasy.

### 5. Sankey flow budget

Replace the tree with ribbons: parent activity splits into one ribbon per
child, width = recent event rate (tool calls/min), done streams fade thin and
dark. The screen answers "where is the swarm's work going" at a glance, with
zero text needed.

- ICS analog: process flow balance / material balance displays.
- Lateral move: abandons spatial truth for rate truth. Biggest conceptual
  jump on this list.
- Cost: medium. New layout, but the timeline data is already there.

### 6. Andon line mode

Each child thread is a station on one horizontal line with a lamp stack above
it; the parent is the supervisor board at the left. Lamp stacks read from
across a factory floor, so this view has no zoom problem at all. Double-click
a station to chat beside it, as today.

- ICS analog: andon boards, line-balancer displays.
- Cost: low-medium. A third layout: one row of stacks. Good candidate for the
  "compact" case where cards already fight for space.

### 7. Vessel containment: make the bounded box mean something

The parent box already exists. Turn it into a tank: fill level = progress
(turns done against expected), a level gauge down the side, children docked
as smaller vessels with stub connections. Containment survives extreme
zoom-out as pure geometry; even as tiny shapes, "those small tanks belong to
that big tank" reads spatially.

- ICS analog: tank farm mimic with level gauges.
- Why: the box is already the right grouping symbol; it's just empty. Give it
  fill, and it reports at any zoom.
- Cost: low. Layout variant of the existing tree; container rects get fills
  and a gauge instead of being blank frames.

### 8. Overview mimic strip (fisheye minimap)

A permanently-100%-legible corner strip: the whole swarm as lamps on lines
(the 20% band from idea 1, but fixed-size, never scaled). A viewport rectangle
shows where the main canvas is looking; click a lamp to fly there. I never
zoom out to 0.2 again because the strip always shows the swarm.

- ICS analog: area overview mimic beside the operating display.
- Why: stops fighting zoom entirely; bifocal display, an old and proven
  trick.
- Cost: low. An overlay reading the same layout data. Arguably the best
  value-per-line-of-code on this list.

### 9. Trend strips under every lane

Under each thread's lane, a thin strip chart: time on x, event ticks for the
last 30 minutes. A child that has been firing steadily reads as dense ticks;
a stalled one reads as a flat line, even while its lamp is still green. State
without rate hides stalls; the trend is the honesty display.

- ICS analog: DCS trend groups under the process display.
- Cost: low-medium. Needs per-node event timestamps (the timeline has them)
  drawn as inline SVG.

### 10. Track diagram: threads as railway lines

The most lateral of the ten. Threads are lines on a track diagram, tool calls
are trains (moving dots) on the line, subagent spawns are junctions, signals
show state. Motion is visible from far away. A stalled line is simply one with
no moving trains. The "what's actually going on" becomes literal movement.

- ICS analog: railway interlocking panel / Underground line diagram.
- Cost: highest on the list (dots animating along edges), but it is the purest
  expression of the flow-control monitor.

## Where my hunch lands

Shortest path to "I can see the swarm and know what it's doing": 1 + 2 + 8.
Morphing bands kill the smudge, the gray board makes live work the only
colored thing, and the mimic strip removes the need to zoom out at all. They
compose and none needs a new layout engine.

The passion project, if the flow-monitor dream is the point: 4 (pipes) or 10
(trains), with 3's annunciator as the fallback view. 9's trend strips are
worth stealing regardless of which view wins, because every view on this list
lies about stalls.

## Open questions for later

- Do scale bands belong on the existing canvas, or is "Operations mode" its
  own view? (Suspect: same canvas, a render-mode flag.)
- Flash codes need an animation budget; the pulse already runs off
  `box-shadow` today, which the review flagged as slow. Any annunciator work
  should start by moving pulses off box-shadow.
- Rate math for 5/9 needs a definition of "event" (tool call? any timeline
  row?) and a window (5 min? 30?).

## Live mockups

All ten are built as animated mockups on one page — one tab per design,
driven by a shared simulated swarm (staggered starts, a first-out failure, a
waiting child, a stall, completions, recovery):
`docs/mockups/swarm-overview-mockups.html`

## The two inverse purposes (added after the mockups)

Watching the gray-board mockup made the purposes of the two surfaces click.
They are inverses, not variants.

**The thread board asks: what needs me?** bb is the operator's console. A
thread that is no longer running is the important one, because it cannot take
another turn until the operator does something. On that surface, emphasis
belongs to blocked-on-operator states, and finished work should get out of
the way.

**The flow view asks: what is the shape of the work?** I want to read the
swarm's action and status without understanding what any thread is doing in
detail. There, activity is the signal, finished work is history that recedes,
and nothing summons me.

Same state set, opposite emphasis policies. The trap is designing one view to
serve both. A gray board that dims "everything not running" would bury the
most operator-important state in the plugin: needs-input. The thread is
stalled, the plant is starved, and the display goes quiet exactly when it
should not.

The rule that reconciles them: **recede the successful, not the inactive.**
Done work is normal operation completing normally; gray it out. Waiting and
error are abnormal, the plant has stopped; they stay lit in every mode.
Running is the motion layer. ISA-101 agrees: color means abnormal, and a
stalled branch is abnormal even with no lamp flashing.

The two surfaces stay separate on purpose:

- Thread board / annunciator (3): sorted by operator actionability.
  Waiting-on-you is the top of the board. This is bb's native job.
- Flow surfaces (gray board 2, pipes 4, sankey 5, tracks 10): sorted by
  activity and shape. Waiting reads as a shut valve, a still train, a thin
  ribbon: absence of motion, still lit, but not a summons.

If the agent graph ever becomes the place I visit to find out what needs me,
it has stopped being a flow view. The mockups already split this way: tab 2
keeps needs-input dashed-amber while done work fades, and tab 3 is where the
attention sorting lives.
