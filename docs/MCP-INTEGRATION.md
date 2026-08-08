# MCP Integration (Documented Option, Not Yet Implemented)

This document specifies how this system's capabilities *could* be exposed via the [Model
Context Protocol](https://modelcontextprotocol.io) so that any MCP-capable agent harness —
Claude Code included, via MCP rather than (or alongside) its native Skills mechanism — can reach
them without a harness-specific integration. **No server code exists in this repo yet.** This is
a design document written to make building that server a well-scoped, low-ambiguity task
whenever it's actually needed, per this repo's own methodology of not building infrastructure
ahead of a real, current requirement (see `METHODOLOGY.md`).

## Why MCP is the right fit here (when it's needed)

The six skills in `skills/` are already harness-agnostic *instructions* (see
`AGENT-HARNESS-ADAPTATION.md`). What they're not, today, is programmatically *invokable* by a
harness that doesn't have Claude Code's native Skill-loading mechanism. MCP's tool/resource model
is a natural fit for exposing:
- **the instructional content itself** as resources (so any MCP client can read a skill's body
  the same way Claude Code does today), and
- **the mechanical, deterministic parts** as tools (scaffolding a new prototype's directory
  structure, running the verification sweep, generating a handoff-package skeleton) — the parts
  that don't require model judgment to execute correctly, only to decide *when* to invoke.

## Proposed resource surface

Read-only, maps directly onto this repo's own file layout:

| Resource URI pattern | Content |
|---|---|
| `prototype-builder://methodology` | `docs/METHODOLOGY.md` |
| `prototype-builder://handoff-format` | `docs/HANDOFF-FORMAT.md` |
| `prototype-builder://skills/{skill-name}` | The body of `skills/{skill-name}/SKILL.md` |
| `prototype-builder://templates/{template-name}` | A blank template from `templates/` |
| `prototype-builder://stack-defaults` | `docs/STACK-DEFAULTS.md` |

A client that only implements resource-reading (no tool-calling) already gets meaningful value:
it can pull the methodology and skill instructions into its own context the same way a human
would read this repo.

## Proposed tool surface

Mechanical operations that don't require model judgment about *what* to do, only *whether* to do
them right now:

| Tool | Input | Effect |
|---|---|---|
| `scaffold_prototype` | `{name, description, stack_overrides?}` | Materializes a new project directory from the templates in this repo (the mechanical portion of `proto-scaffold` — directory structure, template files with placeholders filled in; does **not** perform the judgment calls in that skill, like confirming org/visibility or deciding on stack deviations, which stay the calling agent's responsibility) |
| `render_handoff_skeleton` | `{project_name, has_kb_doc?, kb_doc_name?}` | Emits the four-or-five-file `docs/` skeleton from `templates/docs-package/` with headers pre-filled, ready for the agent to populate from the real prototype's verified behavior |
| `run_verification_sweep` | `{base_url, routes[], viewports[], themes[]}` | Drives the `proto-verify` puppeteer pattern against a running instance, returns structured pass/fail + screenshots — the mechanical execution of that skill's procedure, not a replacement for reading the screenshots and judging them |
| `run_contrast_audit` | `{base_url, selectors[], themes[]}` | Drives the `wcag-audit` in-page contrast-ratio script, returns structured pass/fail per selector/theme |

**Explicitly not proposed as tools**: anything requiring product judgment (deciding what a
feature's acceptance criteria should be, writing the prose sections of `HANDOVER.md`, deciding
whether a bug is "the app's fault" vs. a platform constraint). Those stay squarely in the calling
agent's own reasoning — the tool surface here is intentionally narrow, mirroring
`METHODOLOGY.md`'s point that verification and judgment are not things to automate away.

## Auth / transport notes for a future implementation

- **Local-first**: this system was built for a single operator's prototyping workflow (a
  personal or small-team dev box), so a stdio-transport local MCP server (no auth, runs as the
  same user as the agent) is the right default — not a hosted multi-tenant service.
- **A hosted/shared variant** (a team's shared prototype-builder MCP server) is a reasonable
  future extension but introduces real questions — per-user scaffolding permissions, which repos
  a given caller can write to — that are out of scope for this document until there's an actual
  team workflow driving the requirement.

## Building this

When there's a real, current need for MCP access to this system (not before — see
`METHODOLOGY.md`), the concrete next step is a small Node or Python MCP server implementing the
resource and tool surfaces above, living in a `mcp-server/` directory in this repo, with its own
`README.md` covering install/run instructions for whatever MCP-capable client is consuming it.
Contributions implementing this are welcome — see `CONTRIBUTING.md`.
