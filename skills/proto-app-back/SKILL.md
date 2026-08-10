# App-style back button for PWA prototypes (modal history)

Make the Android hardware/gesture back button behave like a native app in a hash-routed SPA:
back closes the top overlay (drawer, sheet, dialog) and leaves the page beneath untouched;
between views it navigates history with scroll restored. Reference implementation:
`castle-console/public/app.js` (modal-history block near the top) + `configurator.js`
(Theme Lab registering via `window.ccModal`).

## The manager (one per app shell)

```js
const modalStack = [];      // overlay names in open order
const modalClosers = {};    // name -> hide-only close fn (no history effects)
let swallowPop = 0;         // self-triggered history.back() calls to ignore
const afterPopQueue = [];   // follow-up navigations, run once their swallow lands
function modalOpened(name, closer) {
  modalClosers[name] = closer;
  if (modalStack.includes(name)) return;          // content swap in an open overlay
  modalStack.push(name);
  history.pushState({ ccModal: name }, '', location.href);   // SAME hash on purpose
}
function modalClosed(name, then) {
  const i = modalStack.lastIndexOf(name);
  if (i < 0) { if (then) then(); return; }
  modalStack.splice(i, 1);
  if (history.state && history.state.ccModal === name) {
    swallowPop++; afterPopQueue.push(then || null); history.back();
  } else if (then) then();  // hash moved above our entry: don't risk a surprise nav
}
window.addEventListener('popstate', () => {
  if (swallowPop > 0) { swallowPop--; const fn = afterPopQueue.shift(); if (fn) fn(); return; }
  const name = modalStack.pop();
  if (name && modalClosers[name]) { modalClosers[name](); return; }
  popNav = true;                              // real back/forward between views
  setTimeout(() => { popNav = false; }, 0);   // outlives only the paired hashchange
});
// reload with an overlay open leaves its entry current — consume at boot
if (history.state && history.state.ccModal) { swallowPop++; history.back(); }
```

Each overlay splits its close into a **hide** (DOM only) and a **close** (hide +
`modalClosed(name)`); every UI close path — X button, scrim tap, Escape, swipe-down,
form submit — goes through close. Cross-module overlays (a ported configurator, a widget)
register through a `window.ccModal = { opened, closed }` handle so the shell owns history.

## Invariants (each traces to a shipped bug)

1. **Push the SAME hash.** The whole point: popping the entry fires `popstate` but no
   `hashchange`, so the router never runs and the page under the overlay keeps its complete
   state (scroll, in-place DOM, form fields). A naive `pushState`+`back()` layered on a hash
   router without this discipline causes real page navigations (this exact failure got the
   first attempt reverted).
2. **UI closes must CONSUME their entry** (`history.back()` + swallow counter). Skipping this
   leaves a stale same-hash entry behind the current one, and the user's next back press does
   visibly nothing — one dead press per dialog ever closed by X/scrim/Esc.
3. **`location.hash =` is synchronous; `history.back()` traversal is async.** Any
   close-then-navigate flow (menu item → other page, "created X" → its page, delete → parent
   list) must pass the navigation as a callback into `modalClosed`, run when the swallowed
   popstate lands. Navigating inline pushes the new entry ON TOP of the modal entry and the
   queued traversal then yanks the user straight back off the page they just reached. This
   ordering is invisible to synchronous inspection — QA it end-to-end (open overlay → tap the
   navigating control → assert final hash, then back → assert it skips the consumed entry).
4. **Guard against non-close calls.** If `closeDrawer` doubles as a direct event listener,
   its first argument is an Event — accept a follow-up only `typeof then === 'function'`.
   `modalClosed` on a not-open overlay must be a no-op (close calls come from global Escape
   handlers and defensive paths).
5. **Consume only when your entry is still on top** (`history.state.ccModal === name`); if the
   hash somehow moved above it, drop the stack entry and let a later back press resolve the
   stale one — a blind `back()` there is a surprise navigation.
6. **Boot cleanup**: reloading with an overlay open leaves its pushed entry as the current
   one; detect `history.state.ccModal` at startup and consume it, or the first back press
   after a reload is dead.
7. **Scroll memory rides along**: `history.scrollRestoration = 'manual'`; save `scrollY` per
   full hash in the `hashchange` listener (old URL's hash ← `e.oldURL`); the router restores
   the saved position only when the nav came from a traversal (`popNav` flag set in
   `popstate`, cleared in a 0-timeout so it can't leak into a later programmatic nav —
   `popstate` and its paired `hashchange` fire in the same task). Forward navs reset to top.

## QA checklist (puppeteer `page.goBack()` = hardware back)

- Back with overlay open: overlay closes, hash unchanged, a pre-overlay DOM node is still
  `isConnected` (proves no re-render).
- Each UI close path, then back: navigates immediately (no dead press).
- Every close-then-navigate flow: lands on target, then back skips the consumed entry.
- Chained overlays (menu → opens sheet): both entries resolve in order.
- Reload with overlay open, then back: single press navigates.
- Scroll: list → scroll → detail → back restores offset; forward nav starts at top.
- Zero pageerrors throughout.
