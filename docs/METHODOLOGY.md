# Methodology

The core idea this whole system is built around: **the prototype is the spec.**

Instead of writing a specification document and then building software to match it, you build a
real, running, click-through prototype first — against realistic synthetic data, with real UI/UX
decisions made and verified in a browser — and then *extract* the specification from what you
built and verified. The extraction step (the "handoff package," see `HANDOFF-FORMAT.md`) is what
lets a separate build system, a different team, or a future version of yourself rebuild the thing
production-grade without re-deriving every UX decision from scratch or re-litigating decisions
that were already made and tested.

This document describes the loop and the principles behind it, independent of any specific
agent harness or tech stack. See `AGENT-HARNESS-ADAPTATION.md` for harness-specific notes and
`STACK-DEFAULTS.md` for the (swappable) default tech stack this system was proven on.

## The loop: create → test → implement → record

```
   ┌─────────┐     ┌────────┐     ┌───────────┐     ┌────────┐
   │ CREATE  │ ──▶ │  TEST  │ ──▶ │ IMPLEMENT │ ──▶ │ RECORD │ ──┐
   │ a slice │     │ in a   │     │ the real  │     │ decision│  │
   │ of the  │     │ real   │     │ feature   │     │ + spec  │  │
   │ product │     │ browser│     │           │     │         │  │
   └─────────┘     └────────┘     └───────────┘     └────────┘  │
        ▲                                                        │
        └────────────────────────────────────────────────────────┘
```

1. **Create** a small, coherent slice of the product — a feature, a panel, a flow. Small enough
   to verify in one pass, large enough to be a real user-facing unit (not "add a button").
2. **Test** it in a real browser (or real client, for non-web targets) against realistic data.
   Never declare a UI change done from reading code — render it and look, or drive it
   programmatically and assert on the rendered DOM/output. See `AGENT-HARNESS-ADAPTATION.md` for
   why this step is the one most agent harnesses skip, and why skipping it is the single highest-
   leverage mistake to avoid.
3. **Implement** the feature for real once the slice is validated — not a mockup of it, the
   actual working code, wired to actual (if synthetic) data, actually persisted.
4. **Record** what was decided and why — especially the parts that aren't obvious from reading
   the code: why a gesture behaves the way it does, why a permission model has the shape it has,
   why a color was chosen, what edge case a bug fix was actually protecting against. This is the
   raw material the handoff package gets extracted from later; capturing it *at decision time*
   (not reconstructed after the fact) is what makes the extraction cheap and accurate.

Repeat. The prototype accretes into a complete, verified reference implementation. The handoff
package is written once there's enough surface area to be worth extracting (see
`HANDOFF-FORMAT.md` for when and how), and is kept in lockstep with the code from then on — not
written once at the end as an afterthought.

## Principles

### 1. Fake but real data, always
Every feature is built and demonstrated against synthetic data that is realistic, deterministic
(reproducible from a seed, not random), clearly labeled as synthetic, and disposable without
touching anything real. This means a feature is never blocked on "we don't have real data/
credentials yet," and it means every demo, screenshot, and verification run is reproducible. See
the `demo-data-factory` skill for the concrete technique.

### 2. Verification is not optional, and it is not reading code
A change is not "done" because the code looks right. It's done when it's been driven in a real
client and observed to behave correctly — including the failure/edge cases, including both
light and dark themes if theming exists, including the smallest viewport if the product is
responsive, including the state with the *least* content (empty lists and short pages surface
layout bugs that content-rich pages hide — this is not a hypothetical, it's happened repeatedly
in practice). See the `proto-verify` skill.

### 3. Contracts are frozen, internals are not
Once a behavior is verified in the prototype, it becomes part of the contract a rebuild must
preserve: API shapes, permission semantics, gesture behavior, accessibility guarantees, visual
language. *How* that behavior is implemented is never frozen — a rebuild is free to change
languages, frameworks, database engines, anything internal, as long as the observable contract
holds. The handoff package's job is to make this distinction explicit, not to over-specify
implementation.

### 4. Every integration ships with a demo-mode twin
A feature that talks to a real external system (an API, a piece of hardware, a paid service)
should work in demo mode with zero real credentials, and switching to the real integration
should be a config change, not a rewrite. This keeps the prototype fully demoable/testable
without depending on real accounts, and it means the demo-mode path stays exercised (and
therefore working) even after the real integration exists — it doesn't rot into a fallback
nobody tests. See `CONNECTIONS` guidance in `HANDOFF-FORMAT.md`.

### 5. The repo is the handoff, not a conversation transcript
The output of this whole process is not "an agent explained the design to a human once." It's a
git repository — code, docs, and history together — that a separate build system, a different
agent, or a human engineer who wasn't in the room can clone and build from without any other
context. See `REPO-AS-SOURCE-OF-TRUTH.md` for what that requires in practice, including the
uncomfortable but important step of actually testing it: clone fresh, install fresh, boot fresh,
before believing the claim.

### 6. Record the *why*, not just the *what*
Code review can recover what a system does. It cannot recover why a decision was made a
particular way, what alternative was rejected and why, or what incident/bug prompted a
non-obvious safeguard. Capture those at the moment they happen — a "load-bearing decisions"
section in the project's own instructions file is the cheapest place to do this, updated
continuously rather than reconstructed at handoff time.

## What this methodology is not

- It is not "write a PRD, then build to it." The prototype comes first; the spec is extracted
  from what was actually built and verified, not predicted in advance.
- It is not "ship the prototype as the product." The prototype is deliberately allowed to make
  prototype-grade tradeoffs (simulated auth, disposable data, single-instance deployment) as
  long as those tradeoffs are explicitly recorded, not silently shipped as if they were
  production-grade decisions.
- It is not tied to any one tech stack, agent harness, or project type. See
  `AGENT-HARNESS-ADAPTATION.md` and `STACK-DEFAULTS.md` for what's actually specific to how this
  was first proven out, versus what's universal to the method.
