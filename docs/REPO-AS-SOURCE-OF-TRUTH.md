# The Repo Is the Handoff

The handoff artifact is a git repository, not a conversation, a document sent separately, or a
verbal explanation. If a build system, a teammate, or a future agent can't get everything they
need from `git clone`-ing the prototype's repo, the handoff isn't finished — regardless of how
much was explained elsewhere.

## What "source of truth" requires in practice

### 1. The docs package lives in the repo (see `HANDOFF-FORMAT.md`)
Not linked from the repo, not in a wiki the repo references — *in* the repo, versioned alongside
the code it describes, so a specific commit and a specific spec state are always the same thing.

### 2. Commit history is real and kept
Don't squash a prototype's history down to one commit before publishing, and don't force-push
over it. The history itself is evidence: what order things were built in, what got reverted and
why (a revert commit with a real message is more informative than a silently-disappeared
feature), how the design evolved. A build system — or a future you — benefits from being able to
answer "when did this behavior change, and what was it before" from `git log`/`git blame`
directly, not from asking someone to remember.

Practically: commit at meaningful checkpoints (a working feature, a fixed bug, a documented
decision), write commit messages that explain *why* a change was made when that's not obvious
from the diff, and never treat history rewriting as a cleanup step before "real" publishing —
the messy-but-honest history is more valuable than a tidy fabricated one.

### 3. There is an audit trail for anything a rebuild will need to reason about
Application-level audit logging (who did what, when — see the permission-model KB-doc guidance
in `HANDOFF-FORMAT.md`) is a *product* requirement to document, not just a repo-hygiene one: if
the product will eventually need to answer "who changed this and when" for its own users, that
capability should exist (or its absence should be an explicitly documented gap) before handoff,
not discovered as a missing requirement during the rebuild.

### 4. It's verified by actually doing the thing a build system would do
This is the step that's easiest to skip and the one that catches real bugs. Before believing a
repo is handoff-ready:

```bash
git clone <repo-url> <scratch-dir>
cd <scratch-dir>
<install dependencies fresh>
<boot the application fresh>
<exercise the core paths: health check, seed demo data if applicable, log in / core action>
```

Do this in an empty scratch directory, not the long-running dev instance — the dev instance has
accumulated local state (directories created ad hoc during initial setup, config files hand-
edited once and forgotten) that a genuinely fresh clone won't have. This exact failure mode has
happened in practice: a database file's parent directory existed on disk only because it was
manually created once during scaffolding, was correctly gitignored (it holds real/generated
data, never committed), and the application code never re-created it — so the long-running
service worked fine while a fresh clone crashed immediately on first boot. The dev instance gave
false confidence; only the fresh-clone test caught it.

### 5. CI is where "source of truth" becomes enforceable, not just aspirational
A handoff claim ("this repo has everything you need") is a promise. CI is what keeps the promise
true over time instead of decaying the first time someone changes code without updating docs.
At minimum: a syntax/build check, the verification sweep (see the `proto-verify` skill) as an
automated gate, and an accessibility/contrast check if the product has one (see `wcag-audit`) —
all blocking on every push, not just run manually before a release. See `MCP-INTEGRATION.md` and
the individual skills for how these checks are performed manually during prototyping; promoting
them into CI is the natural next step once a prototype graduates toward production.

## What this is not

This is not a call to over-engineer prototype-stage repos with production CI/CD from day one —
see `METHODOLOGY.md` on when to write the handoff package. It's a statement of what "this repo
is the source of truth" has to actually mean once that claim is made: verifiable by a stranger
(or a build system with no other context) starting from nothing but the clone URL.
