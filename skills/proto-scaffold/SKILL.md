---
name: proto-scaffold
description: Bootstrap a brand-new prototype project — folder, git repo, private (or public) GitHub backup, systemd unit, docs skeleton — and hand off the shared dev port to it. Use when starting a new prototype from scratch.
---

# Prototype scaffolding

One dev port, one active service, one git repo per prototype. This is the repeatable setup for
every new prototype in this workspace.

Adapt the specifics (workspace root, org name, port number, service manager) to your own
environment — the placeholders below (`<workspace-root>`, `<org>`, `<port>`) are exactly that:
placeholders. See `STACK-DEFAULTS.md` for the reasoning behind the default stack assumed here.

## Steps

1. **Get the name** — lowercase kebab-case slug (e.g. `widgetco`, `home-console`). This is the
   folder name, service name, and GitHub repo name. Also get a one-line description.

2. **Free up the shared dev port** — if your workspace convention is one prototype at a time on
   a shared port (see `STACK-DEFAULTS.md` "Deployment"):
   ```bash
   systemctl list-units --type=service --state=running | grep -v '@'  # find the active prototype
   systemctl stop <currently-active>.service
   systemctl disable <currently-active>.service
   ```
   Skip if nothing is running yet, or if this new prototype is getting its own dedicated port
   instead of joining the shared rotation (see "Standing exceptions" below).

3. **Create the project from a template directory** (`_template/` in your workspace, containing
   `deploy/service.template`, `package.json.template`, `README.md.template`, `CLAUDE.md.
   template`, `server.js.template` — see `templates/` in this repo for a starting point):
   ```bash
   mkdir -p <workspace-root>/<slug>
   cp -r <workspace-root>/_template/. <workspace-root>/<slug>/
   cd <workspace-root>/<slug>
   mv deploy/service.template deploy/<slug>.service
   mv package.json.template package.json
   mv README.md.template README.md
   mv CLAUDE.md.template CLAUDE.md
   mv server.js.template server.js
   mkdir -p public data
   ```
   Replace every `{{PROJECT_NAME}}`, `{{ONE_LINE_DESCRIPTION}}`, and `{{slug}}` placeholder
   across those files with real values — grep for `{{` afterward to confirm none were missed
   (but note a real server entrypoint may legitimately contain `${PORT}`-style template
   literals; only double-brace `{{...}}` markers are scaffold placeholders). Prefer scripted
   substitution (Python/Node string replace) over `sed`/`perl` one-liners once any value contains
   a `/` — shell-delimiter escaping bugs here are a common, easy-to-avoid time sink.

4. **Install dependencies**: `npm install`.

5. **git + GitHub backup**: confirm org and visibility with the user before creating — default
   private unless the project is explicitly meant to be public (a shared framework repo like
   this one is the exception, not the rule; most prototypes hold a specific product's IP and
   should stay private).
   ```bash
   git init && git branch -m main
   git add -A
   git commit -m "Initial commit: <slug> prototype scaffold"
   gh repo create <org>/<slug> --private --source=. --remote=origin --description "<one-line description>"
   git push -u origin main
   ```

6. **Install and start the service unit**:
   ```bash
   cp deploy/<slug>.service /etc/systemd/system/<slug>.service
   systemctl daemon-reload
   systemctl enable --now <slug>
   curl -s -o /dev/null -w "%{http_code}\n" http://localhost:<port>/
   ```

7. **Hand off to normal development**: the `proto-iterate` skill for edits, `proto-verify`/
   `wcag-audit` for UI changes, `demo-data-factory` once there's a schema worth seeding,
   `prototype-handover` once there's enough behavior to write the handoff package from. Fill in
   the new project's `CLAUDE.md` (working rules, load-bearing decisions, key files) as they
   emerge through the session — don't leave it at template-placeholder state once real work
   starts.

## Standing exceptions to the shared-port rotation

Most prototypes rotate through one shared dev port and get stopped/disabled when a different
prototype becomes active (step 2 above). A prototype can instead be promoted to a **standing,
always-on service on its own dedicated port** when there's an explicit reason for it to keep
running continuously — most commonly, the user wants a specific prototype to stay reachable
(other people using it, a real integration depending on it staying up, active real-world
testing) while other prototypes are developed independently. When this happens:
- Give it its own port, not the shared one.
- `systemctl enable` it permanently — it's not part of the stop/disable handoff other
  prototypes go through.
- Record the exception prominently in that project's own instructions file, including *why*, so
  a future session doesn't "fix" it back onto the shared port by mistake.

## Notes

- If asked to scaffold a prototype whose subject matter doesn't fit the default stack
  (`STACK-DEFAULTS.md`), that's a legitimate deviation — adjust accordingly and record the
  deviation prominently in the new project's instructions file so it isn't mistaken for an
  oversight later.
- Keep a top-level workspace instructions file (see this repo's own `/home/prototypes/CLAUDE.md`-
  style pattern) with the directory map of every prototype, updated when a new one is added.
