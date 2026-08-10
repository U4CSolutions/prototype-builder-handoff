# Card-deck UI component (reusable stack pattern)

The horizontal card-stack component used across Castle Console (rooms, dashboard stats,
chat directory, scene selector). Use THIS pattern for any "one thing at a time, swipe to
browse" surface — don't reinvent carousels. Reference implementation:
`castle-console/public/app.js` (`itemsViewHtml` stacked branch, `deckClassFor`, `layoutDeck`,
`stepStack`, `wireStacked`, `measureDeckBodies`) + `style.css` (`.stacked`/`.deck-*`).

## Markup contract

```html
<div class="stacked deck-short|deck-mid|deck-tall" data-stack="<key>">
  <div class="deck-area">
    <article class="deck-card dk0|dkL1|dkR1|dkL2|dkR2|dkH" [inert]> …content… </article>
    ×N — ALL cards render at once; position classes place them
  </div>
</div>
```

- `stackedIndex[key]` (module map) holds the active index; `deckClassFor(i, active, n)` maps
  signed circular distance → position class; `layoutDeck` re-classes in place (no re-render,
  CSS transitions animate the shuffle; interior scroll positions survive).
- Fixed-height `.deck-area` per size class; only `dk0` is interactive — every other card is
  `inert` so its controls can't be hit and taps fall through to the area (left half = prev,
  right half = next). Swipes are looped.

## Invariants (each traces to a shipped bug — keep all of them)

1. **Center by layout** (`left:0; right:0; margin-inline:auto`), never `left:50% +
   translateX(-50%)` — the untransformed box adds real scrollWidth (phantom page overflow that
   grows with text size).
2. **`.deck-area { overflow: clip; overflow-clip-margin: ~3rem }`** — a dragged card
   transformed past the RIGHT edge adds scrollable overflow (Android renders it as the whole
   page stretching); the clip margin keeps shadows and the drag painted.
3. **Rubber-band the finger-follow** (`40 * tanh(dx/80)`, ≤40px) so the drag stays inside the
   clip headroom.
4. **Gesture safety**: `touch-action: pan-y pinch-zoom` + `overscroll-behavior-x: none`
   (horizontal drags must not trigger history nav); multi-touch aborts the drag (never break
   pinch-zoom); `|dx| > 2|dy|` intent test; ghost-click suppression for ~450ms after a
   recognized swipe (browsers synthesize a click after touch sequences).
5. **Card bodies are scroll containers ONLY when they measurably overflow** (`.scrollable` via
   `measureDeckBodies`) — an empty scroll container is a chaining boundary that traps page
   scroll. Re-measure on resize AND on theme metric changes (root font-size), not just once.
6. **`layoutDeck` resets each card's className** — any custom state class (selection, status)
   must live on an INNER element of the card, or it vanishes on the first step.
7. **In-place refresh**: to re-render contents, swap the whole `[data-stack]` element and
   re-run the wiring — attaching listeners twice to a surviving element double-fires.

## Attaching custom actions (e.g. tap-to-run, hold-to-save)

- Give the CARD the `data-action`/data-* attributes and handle them in the app's delegated
  click handler; only `dk0` can receive the tap (others are inert).
- Hold gestures follow the strict-hold rules (see `proto-crud-states`): primary pointer only,
  second press cancels, >8px movement cancels. **Ghost-click swallow state must live at module
  scope**, not in a closure over the deck element — the hold's own refresh swaps the element
  and a closure-scoped swallow dies with it, letting the synthesized click fire the tap action.
- Keyboard: cards are divs — wire Enter/Space explicitly for `[role=button]`/`[role=link]`.

## Pair mode (`deck-pair`)

Add the `deck-pair` class to the `.stacked` container to show TWO front cards side by side
(`dk0 dk0a` left, `dk0 dk0b` right — both interactive/non-inert), with the under-stack peeks
MIRRORED at both edges (same transform magnitudes each side; with 4 cards the split is
perfectly symmetric). `pairDeckClassFor(i, active, n)` maps circular distance → position
(`d=0/1` fronts, `d=2`→right peek, `d=n-1`→left peek, deeper pairs behind); `layoutDeck`
switches on the container class. Keep peek offsets SMALL (±1.55/2.6rem like the single deck) —
oversized offsets push transformed boxes past the area and reintroduce page scrollWidth even
under `overflow: clip`. Stepping advances the pair by one; tap either front card to
fire its action.

### Pair-mode extras (scene-deck pattern)

- **Permanent stack depth**: four decorative `.deck-underlay` layers (two per side, mirrored
  transforms, `pointer-events: none`) guarantee the stacked look regardless of card count —
  real peeked cards render above them at matching offsets.
- **Create slot**: when only ONE real card exists, render a synthetic front-right card (dashed
  face, its own `data-action`) so the pair is always full — e.g. a "New scene" card that opens
  the create flow. It participates in the loop like any card; hold/tap handlers must filter by
  their own `data-action`.

### Looping

Both deck modes LOOP (swipe cycles endlessly in either direction; pair mode wraps through the
[last, first] pair). If a product ever wants clamped ends instead, clamp the index in BOTH
`stepStack` and `layoutDeck` (stale indices survive card add/remove) and switch the class
mapping to linear distances — but looping is the default spec.

### Directional overlap (pair mode)

During (and after) a step, the INCOMING side rides on top: sliding left (forward) puts the
right front card above the left (`.step-fwd .dk0b { z-index: 6 }`); sliding right (back) puts
the left above the right (`.step-back .dk0a`). Set the direction class on the container in
`stepStack` BEFORE `layoutDeck` runs so the crossing animation carries the right z-order —
container classes survive layoutDeck's card-level className resets. ALSO toggle the class
live in the drag handler from the finger's current `dx` sign (updating if the finger
reverses mid-gesture) — commit-time-only classes leave the overlap stale from the previous
swipe while the user is still dragging.

### Drag offsets COMPOSE with slot transforms — and QA must measure mid-drag

If a card's resting position comes from a base transform (pair-mode slots), the finger-follow
must COMPOSE with it — set a container CSS variable that every slot transform includes
(`translateX(calc(<slot> + var(--dragx, 0px)))`) and drive `--dragx` from JS with a
`.dragging` class disabling transitions. Writing an inline `transform` REPLACES the base and
teleports the card to center the instant the drag starts — an on-device disaster that
class/final-state assertions never catch. QA rule: assert MID-DRAG GEOMETRY (card deltas from
their resting rects, all fronts moving together) with held CDP touch sequences, not just the
post-release state. Full deck QA also covers: every card through the whole loop both
directions, taps at every slot, holds at every slot with full-table isolation, edge taps,
ghost-click suppression, vertical-scroll pass-through, two-finger no-ops, and rapid-burst
stepping.
