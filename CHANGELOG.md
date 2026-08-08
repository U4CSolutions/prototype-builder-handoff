# Changelog

Notable changes to the methodology, handoff format, or skill techniques. Wording/typo fixes are
not tracked here — see git history for those. Dates are when the change was made, not necessarily
when it's read.

## 2026-08-08 — Initial extraction and publication

Extracted this repo from the accumulated, actively-used prototyping system on a private
prototyping workspace, where it had been proven out across multiple real prototypes (an
inventory-SaaS-style product with a live theme configurator; a multi-user home command center
with role-based access, real push notifications, and simulated third-party system integrations).

What shipped in this first cut:
- `docs/METHODOLOGY.md` — the create → test → implement → record loop and its six principles,
  written up from what had been informally practiced across several prototypes.
- `docs/HANDOFF-FORMAT.md` — the four-core-doc handoff package format (`HANDOVER.md`,
  `BACKEND.md`, `CONNECTIONS.md`, `ARCHITECTURE.md`) plus the knowledge-base-doc extension slot
  for product-specific complexity that deserves its own standalone reference.
- `docs/REPO-AS-SOURCE-OF-TRUTH.md` — codifying the "the repo is the handoff, verify it by
  actually cloning fresh" discipline, prompted directly by a real bug this practice caught: a
  gitignored runtime data directory that only existed on disk because it had been created once
  by hand during initial scaffolding, invisible to a genuinely fresh clone.
- `docs/AGENT-HARNESS-ADAPTATION.md` and `docs/MCP-INTEGRATION.md` — drawing an explicit line
  between what's universal to the methodology and what's specific to how it was first
  implemented (Claude Code Skills), plus a documented (not yet built) design for exposing it via
  MCP to any MCP-capable harness.
- `docs/STACK-DEFAULTS.md` — the default tech stack (Node/Express/SQLite/vanilla-JS/systemd) the
  skills assume, with the reasoning generalized so it's useful even for someone choosing
  different technology.
- Six skills ported and genericized from their private, workspace-specific originals:
  `proto-scaffold`, `proto-iterate`, `proto-verify`, `wcag-audit`, `demo-data-factory`,
  `prototype-handover` — stripped of workspace-specific paths/org names, gotcha lists kept
  intact since those are the highest-value, hardest-won content in each one.
- `templates/docs-package/` — blank versions of the four core handoff docs plus the
  knowledge-base template, and per-project `CLAUDE.md`/`README.md` starting templates.

## Future entries

Add an entry here for any change to the methodology, the handoff format, or a skill's actual
technique — see `CONTRIBUTING.md`.
