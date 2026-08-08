# Adapting This System to Different Agentic Harnesses

This system was proven out using Claude Code, and the `skills/` directory in this repo is
written in Claude Code's native "Skill" format. But almost nothing about the *methodology*
(`METHODOLOGY.md`), the *handoff format* (`HANDOFF-FORMAT.md`), or the *repo discipline*
(`REPO-AS-SOURCE-OF-TRUTH.md`) is actually Claude-Code-specific. This document draws that line
explicitly, so the system is genuinely portable — not portable in theory while secretly assuming
one harness's mechanics everywhere.

**Design priority, stated plainly**: write the universal layer so it works unmodified for *any*
sufficiently capable coding agent — Claude Code, Cursor, Aider, Windsurf, GitHub Copilot
Workspace, a bare LLM-plus-tool-use loop you built yourself. Claude Code's skill packaging is
the reference *implementation* of that universal layer, not a dependency of it.

## What's universal (works with any agent harness)

- **The create → test → implement → record loop** (`METHODOLOGY.md`) — any agent that can edit
  files, run a shell command, and observe output can follow this loop.
- **The handoff package format** (`HANDOFF-FORMAT.md`) — plain markdown files in a `docs/`
  directory. No tooling dependency at all; a human could write these by hand.
- **The repo-as-source-of-truth discipline** (`REPO-AS-SOURCE-OF-TRUTH.md`) — git is git,
  regardless of what's driving it.
- **The techniques described in each skill** — deterministic demo-data seeding, patch-verify-
  restart loops, browser-based verification sweeps, contrast auditing — are all just *practices*.
  They happen to be written up as Claude Code Skills in this repo because that's the harness this
  was proven on, but every one of them is describable as plain instructions for any agent that
  can run a headless browser and a shell.

## What's harness-specific (Claude Code today)

- **The `skills/*/SKILL.md` packaging format** — YAML frontmatter (`name`, `description`) plus a
  markdown body, invoked via Claude Code's `Skill` tool. This is Claude Code's mechanism for
  "load this set of instructions into context when relevant."
- **Skill auto-discovery/triggering** — Claude Code surfaces available skills to the model and
  the model (or the user) decides when to invoke one. Other harnesses have different mechanisms
  for this (custom slash commands, `.cursorrules`/`.windsurfrules`-style always-on context files,
  explicit tool definitions, or nothing at all — just a system prompt).
- **The `CLAUDE.md` convention** — Claude Code's convention of reading a `CLAUDE.md` file for
  project-specific instructions. `templates/PROJECT-CLAUDE.md.template` in this repo follows that
  convention because it's the reference implementation, but the *content* (working rules,
  load-bearing decisions, key files) is a useful pattern for literally any harness that can be
  pointed at a persistent instructions file.

## Porting the skills to another harness

For each skill in `skills/`, the `SKILL.md` body (everything after the frontmatter) is the
actual portable content — the frontmatter is just Claude Code's metadata wrapper. To adapt a
skill to a different harness:

1. **Cursor / Windsurf** (rules-file-driven): fold the relevant skill bodies into
   `.cursorrules`/`.windsurfrules` or project-specific rule files, either as always-on context or
   split by directory-scoped rules if the harness supports that. The patch→check→restart→verify
   loop in `proto-iterate` and the verification sweep in `proto-verify` translate directly into
   "rules the agent should follow when editing/verifying this project."
2. **Aider**: the conventions map onto Aider's own `.aider.conf.yml`/`CONVENTIONS.md`-style
   persistent-instructions mechanisms; the shell-command-driven skills (verify, wcag-audit) work
   unmodified since Aider can already run arbitrary shell commands.
3. **A custom agent loop you built**: treat each `SKILL.md` body as a prompt fragment to inject
   into context when its trigger condition is met (a file edit in a running prototype → inject
   `proto-iterate`; about to claim a UI change works → inject `proto-verify`; etc.) — this is
   the most literal translation, since it's exactly what Claude Code's own dispatch mechanism is
   doing under the hood.
4. **No agent at all (a human following the docs)**: every skill body is written as plain
   numbered/bulleted instructions specifically so it reads correctly as a human checklist, not
   just an LLM prompt. This was a deliberate authoring constraint, not an accident.

## Claude Code as an MCP option

Claude Code (and other MCP-capable harnesses) can also consume this system's capabilities via
the Model Context Protocol rather than (or alongside) the native Skills mechanism — see
`MCP-INTEGRATION.md` for the documented design of what an MCP server exposing this system's
tools/resources would look like. This is offered as an *additional* integration path, not a
replacement for the skills — MCP is the better fit when this system needs to be reachable from
a harness that has no native "skills" concept of its own but does speak MCP.

## If you adapt this system to a new harness

Please consider contributing the adaptation back (see `CONTRIBUTING.md`) — a `harnesses/<name>/`
directory alongside `skills/` for harness-specific packagings (Cursor rules, Aider config, etc.)
that stay in sync with the universal docs is the intended shape for this to grow into, once more
than one harness's adaptation exists to establish the pattern.
