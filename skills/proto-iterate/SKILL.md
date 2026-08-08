---
name: proto-iterate
description: Safe patch→check→restart→verify iteration loop for prototype web apps. Use whenever editing a running prototype service (Node/Express or similar) so changes land verified, never blind.
---

# Prototype iteration loop

The reliable cycle: patch with pattern-miss detection, syntax-check, restart-to-deploy, verify in
a real browser. Every step exists because skipping it has caused a real bug to ship silently at
some point — this isn't process for its own sake.

1. **Patch via a scripted replace with pattern-miss detection** — never blind-append when
   replacing existing code, and never trust that a find/replace matched without checking:

```bash
node << 'PATCH'
const fs = require('fs');
const f = 'public/app.js';
let s = fs.readFileSync(f, 'utf8');
const old = `exact existing code`;
const neu = `replacement code`;
if (!s.includes(old)) { console.error('PATTERN MISS'); process.exit(1); }  // NEVER skip this
s = s.replace(old, neu);
fs.writeFileSync(f, s);
console.log('patched');
PATCH
```

2. **Syntax gate**: `node --check <file>` (or your language's equivalent — `python -m py_compile`,
   `go build`, etc.) for every touched file, before restarting anything.
3. **Deploy**: `systemctl restart <service> && sleep 1` (restart-to-deploy; no build step, per
   `STACK-DEFAULTS.md`).
4. **Verify in a real browser** — use the `proto-verify` skill. Never declare a UI change done
   from reading code alone; render it and look, or drive it programmatically and assert on the
   rendered result.
5. CSS is append-heavy by convention in a no-build-step prototype: later rules win at equal
   specificity. When a change "doesn't take," suspect an earlier higher-specificity rule (grep
   the selector) rather than re-appending blindly.

## Hard-won gotchas (cost real debugging time — respect them)

- A patch that silently no-ops is worse than one that fails: **always** exit non-zero on pattern
  miss, and check the tool's output actually confirms the patch landed.
- Command output containing terminal color codes (ANSI escapes) can corrupt anything you capture
  it into for further use (a URL, a path) — strip color output when capturing programmatically,
  don't just eyeball that it "looks fine" in the terminal.
- Shell variable/glob expansion inside a quoted script string can silently corrupt template
  literals or path strings — prefer single-quoted heredocs (`<< 'PATCH'`) so the shell doesn't
  touch the script body at all.
- Inside a single-quoted heredoc, characters with special meaning in *other* quoting contexts
  (backticks, `$`) are literal — don't escape them defensively; escaping can itself introduce a
  syntax error.
- When generating code that itself contains string-interpolation syntax (JS template literals,
  Python f-strings, etc.) as literal text rather than something to evaluate, double-check the
  interpolation markers survived the generation step exactly as intended — after patching, grep
  the target file for the expected literal text rather than assuming it came through correctly.
- Media-query / more-specific-selector overrides must appear *after* the base rules they override,
  at equal specificity, in an append-heavy stylesheet.
- Platform-specific mobile-browser quirks are real and easy to lose to a well-intentioned but
  wrong "obvious" CSS property — e.g. `overflow-x: hidden` on the root elements can silently
  break `position: sticky` on some mobile browsers where `overflow-x: clip` doesn't; horizontal
  touch-drag gestures can trigger unwanted browser-level history navigation unless explicitly
  suppressed. Treat any such fix you discover as a load-bearing decision worth recording (see
  `HANDOFF-FORMAT.md` / the project's own instructions file), not just a one-off patch.
