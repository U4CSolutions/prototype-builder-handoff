# The Handoff Package Format

The concrete deliverable this system produces alongside every prototype: a `docs/` directory in
the prototype's own repository, containing a small, fixed set of documents that together let a
separate build system (or a human, or a different agent) rebuild the product production-grade
without reading the prototype's source code line by line. Blank templates for every file below
live in `templates/docs-package/` in this repo.

## Why a fixed format

A build system consuming handoff packages from many different prototypes benefits enormously
from those packages having a **consistent shape** — the same four-or-five files, in the same
place, answering the same questions, every time. Ad hoc documentation (a README that grew
organically, scattered comments, a wiki nobody kept current) doesn't compose across projects the
way a fixed format does. This is the same reasoning that makes OpenAPI/JSON Schema valuable for
APIs, applied to the *product-and-architecture* level instead of just the API-shape level.

## The core four (write these for every prototype worth handing off)

### 1. `HANDOVER.md` — the master build document
The entry point. Contains:
- **Product definition**: what the thing is, in a paragraph, plus any load-bearing philosophy
  (a tier/permission model, a core UX principle) that shapes everything else.
- **Module specifications as user stories + acceptance criteria**, harvested from what the
  prototype *actually does* — including the edge cases that got fixed during building, since
  those are the highest-value ones to capture (they represent a mistake already made and
  corrected; a rebuild that doesn't know about them will make the same mistake again).
- **Non-functional requirements**: accessibility bar, performance/responsiveness expectations,
  the security posture actually verified (not aspirational).
- **Test matrix**: every acceptance criterion, with how it was verified in the prototype
  (E2E/integration/programmatic-assertion/manual) — a rebuild should turn each row into an
  automated CI check.
- **Build-suite requirements**: what tooling the target harness should set up (linter, types,
  test frameworks, CI stages) — prescriptive, not just descriptive of what the prototype has.
- **Parity checklist** and **known gaps/open questions** — an honest list, not a marketing one.

### 2. `BACKEND.md` — the contract reference
Every API endpoint (method, path, params, response shape, auth/permission gating), the full data
schema, and the architectural notes a backend engineer needs that don't fit neatly as an
endpoint-by-endpoint fact: what's an audit trail and what isn't, what's deliberately not
paginated yet, what limitations were accepted at prototype scale and why. If the product isn't a
web API, this is still the right place for "the contract" in whatever form it takes (CLI
interface, library API, message schema).

### 3. `CONNECTIONS.md` — real vs. simulated, and how to flip the switch
Every external integration point, explicitly marked **real** or **simulated**, with:
- what the simulated/demo-mode version does (see `METHODOLOGY.md` principle 4),
- exactly what a real integration requires (credentials, endpoints, translation layer between
  the app's internal model and the external system's model),
- any hard platform constraint that isn't a bug and can't be "fixed" (e.g. a browser API that
  requires a secure context) — documented so nobody wastes time trying to fix the unfixable.

### 4. `ARCHITECTURE.md` — the target build architecture
The gap between "what the prototype is" and "what a production build needs to be": measured
current-state metrics, target architecture diagram, frontend/backend/hosting requirements,
a security hardening checklist, a CI/CD plan, and an explicit list of what the spec-driven
builder must **not** change (the frozen contracts, cross-referenced from the other docs).

## The extension slot: knowledge-base docs

Some products have one piece of logic that's disproportionately easy to get subtly wrong in a
rebuild — a permission model with more than one independent axis, a pricing/tier engine, a state
machine with non-obvious transitions. When a prototype has one of these, give it its own
standalone knowledge-base doc (`ROLES-AND-PERMISSIONS.md`, `PRICING-ENGINE.md`, whatever fits)
rather than burying it as a subsection of `BACKEND.md`. The signal that something deserves this
treatment: you'd want a new contributor to be able to read *just this one file* and correctly
extend the logic without also reading the other four documents first.

`templates/docs-package/KNOWLEDGE-BASE.md.template` is a generic starting point for this pattern
— it's deliberately not prescriptive about content since what goes in it is, by definition, the
one thing that's unique to this particular product.

## When to write the package

Not on day one. Write it once there's enough real, verified behavior to extract from — usually
once the first few core features are built and tested, not before. Writing a handoff package
against a prototype that's still churning wastes effort re-writing it every session. But once
it exists, treat it as living documentation: update it in the *same session* as any behavior
change, the same discipline as updating tests alongside code. A stale handoff package is worse
than no handoff package, because it's trusted.

## Where it lives, and why that matters

`docs/` inside the prototype's own git repository — not a separate wiki, not a shared drive, not
a chat transcript. See `REPO-AS-SOURCE-OF-TRUTH.md` for the full reasoning: the repo (code +
docs + commit history together) is the actual handoff artifact. A build system should be able to
`git clone` the prototype's repository and have everything it needs.
