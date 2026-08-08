# Contributing

This repo documents a real, actively-used prototyping methodology — it's public specifically so
others can adopt and adapt it, and so improvements discovered anywhere flow back. Contributions
are welcome, with a bias toward changes that come from actually having used the system, not
speculative process improvements.

## What's especially welcome

- **Harness ports**: an adaptation of the skills to a different agentic coding harness (Cursor,
  Aider, Windsurf, a custom agent loop) — see `docs/AGENT-HARNESS-ADAPTATION.md`. The suggested
  home for these is a `harnesses/<name>/` directory alongside `skills/`, once more than one
  exists to establish the pattern.
- **A real MCP server implementation** of the design in `docs/MCP-INTEGRATION.md`, once someone
  has an actual need for it (see that doc's framing on why it's documented-not-built today).
- **Corrections and additions to the gotcha lists** in `skills/proto-iterate/SKILL.md`,
  `skills/proto-verify/SKILL.md`, and `skills/wcag-audit/SKILL.md` — these are meant to
  accumulate real, hard-won lessons over time, not stay static.
- **A worked example** — a genericized, sanitized walkthrough of applying the full methodology
  end-to-end on a toy project, if one doesn't already exist by the time you're reading this.

## What to think twice about

- **Adding process for its own sake.** Every rule in `docs/METHODOLOGY.md` exists because
  skipping it caused a real problem at some point. If you're proposing a new rule, it should be
  because you hit a real problem it would have prevented — say so in the PR description.
- **Coupling the universal docs to one harness's mechanics.** `docs/METHODOLOGY.md`,
  `docs/HANDOFF-FORMAT.md`, and `docs/REPO-AS-SOURCE-OF-TRUTH.md` should stay readable and
  actionable by someone using a completely different toolchain. Harness-specific detail belongs
  in `skills/` or a `harnesses/` port, not the core docs.
- **Baking in one organization's specifics.** This repo intentionally uses placeholders
  (`<workspace-root>`, `<org>`, `{{PROJECT_NAME}}`, etc.) instead of any one team's real paths or
  org names — keep it that way so it stays adoptable by anyone.

## Process

Open a PR. For anything beyond a small fix, a short issue or discussion first is appreciated so
the change can be scoped before the writing effort goes in — this is a documentation-heavy repo,
and the cost of a contribution is mostly in getting the content right, not the diff size.

## Changelog discipline

Any change that affects the methodology, the handoff format, or a skill's actual technique
(not just wording/typo fixes) should add an entry to `CHANGELOG.md` describing what changed and,
briefly, why — the same "record the why" discipline this repo asks of every prototype it
documents (`docs/METHODOLOGY.md` principle 6) applies to itself.
