---
name: proto-verify
description: Headless-browser verification harness for prototype UIs — screenshots, mobile overflow audits, touch-gesture simulation, JS-error capture, theme sweeps. Use after every UI change and before telling anyone something works.
---

# Prototype UI verification (Puppeteer)

Puppeteer is a dev dependency of every prototype. Run verification scripts with the working
directory set to the project root so a plain `require('puppeteer')` resolves from that
project's own dependencies — never hardcode another project's path. Boilerplate:

```js
const puppeteer = require('puppeteer');
const b = await puppeteer.launch({ headless: 'new', args: ['--no-sandbox', '--disable-dev-shm-usage'] });
const p = await b.newPage();
const errs = []; p.on('pageerror', e => errs.push(e.message));   // ALWAYS collect
await p.setViewport({ width: 390, height: 844, deviceScaleFactor: 2, hasTouch: true }); // mobile-class
```

## Standard assertions

- **No horizontal overflow**, on every page, in every theme, at every content-density state
  (including the *shortest* content state for the *most* restricted user/role — see "the
  smallest-content-state trap" below): `document.documentElement.scrollWidth === window.innerWidth`
- **Zero pageerrors** across the full interaction sweep.
- **Both themes**, if the product has theming — drive the theme engine programmatically and
  re-assert, don't just check the default.
- **Geometry ground truth is `getBoundingClientRect()`** — assert alignment/visibility/size
  numerically, don't eyeball a screenshot for something a script can measure exactly.

## Screenshots

- `page.screenshot({ clip })` uses page coordinates, not viewport coordinates — for below-the-
  fold targets, compute `rect.top + window.scrollY`.
- **Read every screenshot before claiming success.** A passing programmatic assertion alongside
  a broken-looking render is still a failure — the assertions catch what you thought to check
  for; the screenshot catches what you didn't.

## Touch gestures (CDP — requires a `hasTouch` viewport)

```js
const cdp = await p.target().createCDPSession();
const swipe = async (x1, y1, x2, y2) => {
  await cdp.send('Input.dispatchTouchEvent', { type: 'touchStart', touchPoints: [{ x: x1, y: y1 }] });
  for (let i = 1; i <= 5; i++) {
    await cdp.send('Input.dispatchTouchEvent', { type: 'touchMove',
      touchPoints: [{ x: x1 + (x2 - x1) * i / 5, y: y1 + (y2 - y1) * i / 5 }] });
    await new Promise(r => setTimeout(r, 20));
  }
  await cdp.send('Input.dispatchTouchEvent', { type: 'touchEnd', touchPoints: [] });
};
```

Gesture caveats discovered the hard way:
- A browser may fire **touchcancel** (not touchend) when native scroll consumes a vertical drag.
- Browsers synthesize a **ghost click** after a touch sequence — an app-side fix is a
  capture-phase click suppressor armed by recognized swipes.
- Horizontal drags can trigger **history back/forward navigation** unless the app sets
  `overscroll-behavior-x: none`.
- Debug gesture handlers by logging decisions from a parallel in-page listener, and trace hash/
  navigation issues via `hashchange` (log `e.oldURL → e.newURL`, since same-task double
  assignments make direct reads of the current location lie) plus `popstate` (a fingerprint of
  browser-level gesture navigation vs. app-level routing).

## The viewport-toggle reload trap (a test-harness gotcha, not usually an app bug)

Toggling `hasTouch`/`deviceScaleFactor` via `page.setViewport()` **mid-session** — e.g. switching
from a mobile-emulation profile to a desktop one within the same verify script — can trigger a
real full page reload in some headless-Chrome/Puppeteer version combinations. This looks exactly
like an app bug (in-memory state wiped, a "logged out" state appearing mid-test, null-reference
crashes) and can cost real debugging time chasing the wrong cause. If you need both a mobile and
a desktop viewport profile in one script, either hold `hasTouch`/`deviceScaleFactor` constant
across the switch (vary only width/height) or use a fresh `page`/`browser.newPage()` per profile.
If an unexplained error appears immediately after a viewport switch, suspect this before assuming
an app bug — check for `page.on('load')` firing an extra time.

## The smallest-content-state trap

Don't only verify the page/state with the most content (usually a default dashboard). Also
verify whichever state has the *least* content for the *most* restricted role/permission level
(an empty list, a scoped-down user account, a first-run empty state). Layout bugs involving
containers that are supposed to scroll internally but instead stretch to fill available space
reliably hide behind content-rich pages and surface only on short ones — this has happened in
practice, not hypothetically.

## Full-app smoke sweep

Iterate every route × {every theme} × {mobile width, desktop width} (× every distinct role/
permission level, if the product has more than one): assert the page renders, no horizontal
overflow, zero JS errors. Run this before any "ready for testing" or "this works" claim.
