# Prototype Builder Handoff

A methodology and toolkit for building real, verified, click-through prototypes with a coding
agent — and handing each one off to a separate build system, team, or future agent as a
**self-contained git repository**, complete enough that `git clone` is the whole handoff.

Proven out over multiple real prototypes built with [Claude Code](https://claude.com/claude-code).
Written to be genuinely portable to other agentic coding harnesses first, and packaged as native
Claude Code Skills second — see [`docs/AGENT-HARNESS-ADAPTATION.md`](docs/AGENT-HARNESS-ADAPTATION.md).

## The idea, in one paragraph

Instead of writing a spec and then building to it, build a real, working prototype first —
against realistic synthetic data, with actual UX decisions made and verified in a browser — then
extract the specification from what you built and verified. The prototype is the spec. The
extracted handoff package (a fixed set of markdown docs living in the prototype's own repo) is
what lets someone else — a different build system, a different team, a future version of you —
rebuild it production-grade without re-deriving every decision from scratch.

See [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) for the full write-up.

## What's in this repo

```
docs/
  METHODOLOGY.md               The create → test → implement → record loop, and the principles behind it
  HANDOFF-FORMAT.md            The canonical docs/ package spec every prototype hands off with
  REPO-AS-SOURCE-OF-TRUTH.md   What "this repo is the source of truth" requires in practice
  AGENT-HARNESS-ADAPTATION.md  What's universal vs. Claude-Code-specific, and how to port this to other harnesses
  MCP-INTEGRATION.md           Documented (not yet implemented) design for exposing this via MCP
  STACK-DEFAULTS.md            The default tech stack this was proven on, and when to deviate

templates/
  docs-package/                Blank templates for HANDOVER.md, BACKEND.md, CONNECTIONS.md, ARCHITECTURE.md, KNOWLEDGE-BASE.md
  PROJECT-CLAUDE.md.template   Per-project instructions-file template
  PROJECT-README.md.template   Per-project README template

skills/
  proto-scaffold/              Bootstrap a new prototype: folder, git, service, docs skeleton
  proto-iterate/                The patch → check → restart → verify edit loop
  proto-verify/                 Headless-browser verification: screenshots, overflow, gestures, JS errors
  wcag-audit/                   Programmatic WCAG 2.2 AA contrast auditing
  demo-data-factory/            Deterministic, realistic, labeled synthetic data patterns
  prototype-handover/           Generating and maintaining the handoff package itself
```

Each `skills/*/SKILL.md` is written in [Claude Code's Skill format](https://docs.claude.com/en/docs/claude-code/skills)
(YAML frontmatter + markdown body) — but the body of every one is written to also read correctly
as a plain instructions document for a human, a different agent harness, or a custom agent loop.
See `docs/AGENT-HARNESS-ADAPTATION.md` for how to port them.

## Quickstart

1. Read `docs/METHODOLOGY.md` first — the loop and the principles.
2. Scaffold a new prototype following `skills/proto-scaffold/SKILL.md`, adapting the
   `<workspace-root>`/`<org>`/`<port>` placeholders to your own environment. `STACK-DEFAULTS.md`
   documents (and justifies) the default tech stack assumed by the skills; swap freely, note the
   swap in the project's own instructions file.
3. Build, iterating with `skills/proto-iterate/SKILL.md`, verifying with
   `skills/proto-verify/SKILL.md` and `skills/wcag-audit/SKILL.md`, seeding realistic data with
   `skills/demo-data-factory/SKILL.md`.
4. Once there's enough verified behavior to be worth extracting, write the handoff package
   following `docs/HANDOFF-FORMAT.md` and `skills/prototype-handover/SKILL.md`, using the
   templates in `templates/docs-package/`.
5. Before calling the handoff done, verify it the way `docs/REPO-AS-SOURCE-OF-TRUTH.md`
   describes: clone the pushed repo fresh into a scratch directory and boot it from nothing but
   the clone.

## Using this with Claude Code

Copy (or symlink) the contents of `skills/` into your project's or user's `.claude/skills/`
directory, and copy `templates/PROJECT-CLAUDE.md.template` into a new prototype as its starting
`CLAUDE.md`. Claude Code will surface the skills automatically when their descriptions match what
you're doing.

## Using this with another agent harness, or no agent at all

See `docs/AGENT-HARNESS-ADAPTATION.md` — the methodology, the handoff format, and the technique
described in each skill are all harness-agnostic by design. Only the packaging (Claude Code's
`SKILL.md` frontmatter/dispatch mechanism) is specific to Claude Code, and it's a thin wrapper
around plain-markdown content that reads correctly as a checklist for a human or a prompt
fragment for any other agent.

## Status

This is a living document of an actively-used, actively-evolving system — not a finished
framework. See `CHANGELOG.md` for what's changed and why. It's public and open-source
specifically so others can adapt it to their own prototyping workflows; see `CONTRIBUTING.md` if
you'd like to contribute an adaptation, a harness port, or an improvement back.

## License

MIT — see `LICENSE`.
