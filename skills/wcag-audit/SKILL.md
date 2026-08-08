---
name: wcag-audit
description: Programmatic WCAG 2.2 AA contrast auditing for web UIs, run in-page across every theme. Use after any styling/theming change — never eyeball contrast.
---

# WCAG 2.2 AA contrast audit

Thresholds: **4.5:1** for normal text, **3:1** for large text (≥24px, or ≥18.66px bold) and UI
component boundaries.

In-page audit function (evaluate via a headless browser, once per theme):

```js
const lum = (r, g, b) => {
  const f = (c) => { c /= 255; return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4); };
  return 0.2126 * f(r) + 0.7152 * f(g) + 0.0722 * f(b);
};
const ratio = (fg, bg) => {
  const l1 = lum(...fg), l2 = lum(...bg);
  return (Math.max(l1, l2) + 0.05) / (Math.min(l1, l2) + 0.05);
};
// effective solid background: walk up ancestors until a non-transparent backgroundColor
const bgOf = (el) => {
  for (let n = el; n && n !== document.documentElement; n = n.parentElement) {
    const m = getComputedStyle(n).backgroundColor.match(/[\d.]+/g)?.map(Number) || [];
    if (m.length === 3 || (m.length === 4 && m[3] >= 0.99)) return m.slice(0, 3);
  }
  return getComputedStyle(document.body).backgroundColor.match(/[\d.]+/g).map(Number).slice(0, 3);
};
```

## Procedure

1. List the selectors added/changed this session (pills, badges, buttons, labels, links, chart
   text).
2. For each visible match: `fg` = computed `color`, `bg` = `bgOf(el)`, size/weight → threshold;
   report PASS/FAIL ratios.
3. Repeat for **every** theme the product supports (drive the theme engine programmatically;
   reload first if palettes are only set at boot).
4. Fix failures by adjusting the *component's own* colors (theme-conditional overrides scoped to
   that component), not by weakening a shared/global color token that other, currently-passing
   components also depend on.
5. Elements over a gradient: check against the gradient's midpoint color at minimum.

## Two traps that produce false readings

**A design-system token that's contrast-safe against one background isn't automatically safe
against a different one.** If a theme engine computes a "muted text" or "status text" color
enforced to be AA-compliant against the page's base background, and a component then places that
same text against a *different* surface tone (a slightly recessed card background, for
instance), the pre-computed color can fall just short of the threshold even though the token
itself is "AA-enforced." The fix is choosing a background for that component that the token was
actually validated against (or was validated to a token with enough margin to survive the
shift), not touching the token. This has happened in practice — margins as small as a few
hundredths of a contrast-ratio point below threshold, easy to miss without a programmatic check.

**Decorative color-swatch buttons (no visible text) are not text-contrast failures.** A custom
color picker's palette swatches are legitimately buttons with no text content, only an
`aria-label` — an automated sweep that blindly checks `color` vs `background-color` on every
`<button>` will flag these. Check the matched element's actual visible text content before
treating a flagged result as real; an empty/decorative match is a false positive from an
overly-broad selector, not a bug to fix.

## Known trap: UA default button text

Default browser button text is black — any `<button>` restyled as a tile/card needs an explicit
`color: inherit` (or equivalent), or dark-mode contrast fails instantly and silently.

## Non-contrast AA checklist that rides along

Visible `:focus-visible` rings on every interactive element; `aria-label` on icon-only buttons;
`aria-pressed`/`aria-expanded` state kept in sync with actual state; underlined (not
color-only-distinguished) body links; `prefers-reduced-motion` fallbacks for every animation; hit
targets at least ~40px on touch interfaces.
