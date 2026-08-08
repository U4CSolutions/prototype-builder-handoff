# Default Stack (Swappable)

This system was proven out using one specific, deliberately boring technology stack, chosen for
low operational overhead and fast iteration, not because the methodology requires it. Every
skill and template in this repo assumes these defaults *unless a project's own instructions say
otherwise* — swapping any of them is a normal, expected adaptation, not a deviation to apologize
for. Note any swap prominently in the project's own instructions file so it isn't mistaken for
an oversight later (see `templates/PROJECT-CLAUDE.md.template`).

## The defaults

| Layer | Default | Why |
|---|---|---|
| Runtime | Node.js (current LTS) | Ubiquitous, fast startup, no compile step for the common path |
| Web framework | Express | Minimal, well-understood, doesn't fight you |
| Database | SQLite (WAL mode, foreign keys on) via a synchronous driver (e.g. `better-sqlite3`) | Zero operational overhead (no separate DB server to run/back up during prototyping), synchronous API keeps prototype code simple, trivially portable (single file), and — critically — its constraints (single-writer, no network access) are *good* constraints for a prototype: they force simplicity that pays off when the schema is handed off for a real build to evolve |
| Frontend | Vanilla JS SPA, hash-based routing, **no build step** | The fastest possible edit-verify loop (`proto-iterate`'s patch→restart→verify cycle depends on there being no compile/bundle step between an edit and seeing it work) — a build step is a legitimate thing to add for a *production* rebuild, but actively hostile to prototype-speed iteration |
| Deployment | systemd service, one port per running prototype | Simplest possible "always running, restarts on crash, survives reboot" — no container orchestration overhead for a single-instance prototype |
| Verification | Puppeteer (headless Chrome) | Real browser, real rendering, real JS execution — the only way to actually verify a web UI rather than infer it from source |
| Accessibility | WCAG 2.2 AA, checked programmatically (see `wcag-audit`) | A fixed, external, testable bar — not a matter of taste |

## Why these specific choices, generalized

The pattern underneath the specific choices, useful when picking defaults for a different domain
(a CLI tool, a mobile app, a data pipeline) or a different ecosystem (Python, Go, Rust):

1. **Prefer zero-config, zero-server-process dependencies during prototyping.** SQLite over
   Postgres; a synchronous driver over connection pooling; a single running process over a
   service mesh. Every piece of infrastructure you don't have to run is iteration speed you keep.
2. **Prefer no build step over a build step**, until there's a concrete reason (bundle size,
   TypeScript, a component framework the team actually wants) to add one. A build step is a
   multiplier on the cost of every single edit-verify cycle you'll run hundreds of times.
3. **Prefer boring, well-documented technology** a coding agent (or a new team member) already
   has deep training-data familiarity with, over something novel that requires the agent to
   reason more carefully and make more mistakes per line.
4. **Match the deployment model to the actual product**, not to a generic "cloud native" default.
   A self-hosted single-instance household product (see the worked example in this repo's
   history) should target a Raspberry-Pi-class single box, not a Kubernetes cluster — over-
   engineering the target architecture is as much a mistake as under-engineering it.

## When to deviate

Deviate whenever the product genuinely needs something the defaults don't provide — real-time
bidirectional communication needs WebSockets the defaults don't preclude but don't assume either;
a product with heavy client-side interactivity might genuinely benefit from a component
framework; a product that's fundamentally a data-processing pipeline has almost nothing in
common with "a web app with a database." The point of naming defaults explicitly is so a
deviation is a *visible, intentional decision* recorded in the project's instructions file, not
a silent drift that looks like an oversight to whoever reads the project next.
