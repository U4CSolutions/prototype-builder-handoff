---
name: prototype-handover
description: Generate and maintain a spec-driven handover package from a working prototype so another code-generation harness (or team) can re-architect it production-grade. Use when asked to document, hand over, or "productionize" a prototype.
---

# Prototype → build-handover package

The prototype is the living spec (see `METHODOLOGY.md`). This skill extracts it into the
documents specified in `HANDOFF-FORMAT.md` — written so a fresh engineering harness (human or
agent) can build from them without reading the prototype's source code line by line — while
keeping the prototype itself runnable as the reference implementation.

## Package layout

Write these into the project's own `docs/` directory (see `HANDOFF-FORMAT.md` for full detail on
each):

| File | Contents |
|---|---|
| `HANDOVER.md` | Master build document: scope, target architecture summary, module specs as user stories + acceptance criteria, NFRs, test matrix, build-suite requirements, parity checklist |
| `CONNECTIONS.md` | Every external integration: real-integration requirements + demo-mode mock behavior for each |
| `BACKEND.md` | API/interface reference, data schema, architecture notes, documented limitations |
| `ARCHITECTURE.md` | Target production architecture, hosting/ops requirements, CI/CD plan, security checklist |
| *(optional)* a knowledge-base doc | For any one piece of logic disproportionately easy to get subtly wrong in a rebuild (see `HANDOFF-FORMAT.md` "the extension slot") |

## Authoring rules

1. **Specs are behavioral, not implementational**: write each feature as user stories with
   Given/When/Then acceptance criteria harvested from what the prototype *actually does*
   (including edge cases fixed during prototyping — those are the highest-value ones, since they
   represent a mistake already made and corrected).
2. **The prototype's verified behaviors ARE the regression suite**: every verification assertion
   run during development becomes a test-matrix row, classified by how it was verified
   (unit/integration/E2E/programmatic-assertion).
3. **Contract-first**: document interface shapes exactly as the prototype actually serves them
   (status codes, error shapes, edge-case parameter handling included). A rebuild may change
   internals; it should not change contracts without an explicit spec update.
4. **Record the whys**: product philosophy, UX decisions and the reasoning behind them,
   accessibility bar, interaction semantics (gestures, keyboard behavior). A rebuild that loses
   these regresses the product even if every acceptance criterion technically still passes.
5. **Prescribe the build suite** the target harness should set up: linter + config, type layer,
   unit test framework + coverage floor, E2E runner, CI pipeline stages, a migration path for
   prototype data if applicable.
6. **Keep docs in lockstep with the prototype**: any feature change updates the relevant handoff
   doc in the *same session* — the same discipline as updating tests alongside code, not a
   separate documentation pass done later (and often not done at all, if deferred).
7. **Verify the handoff claim itself**: before considering a package complete, actually clone the
   repository fresh into a scratch directory and boot it from nothing but the clone — see
   `REPO-AS-SOURCE-OF-TRUTH.md`. This step has caught real bugs (a missing runtime-created
   directory, an undocumented manual setup step) that the long-running dev instance masked.

## Regenerating from scratch

Walk: routes/views → API endpoints/interface surface → data schema → access-control/permission
logic → any theming/accessibility system → interaction/gesture semantics → the demo/mock layer
for each integration. For each, write spec + acceptance criteria + open questions. Then diff
against the actually-running prototype using the `proto-verify` skill before publishing — a
handoff doc that describes intended behavior rather than verified behavior is a liability, not
an asset.
