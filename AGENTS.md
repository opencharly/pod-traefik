# AGENTS.md — pod-traefik

Standalone candy repo for the `traefik` candy — a reverse proxy with TLS
termination, HTTP-to-HTTPS redirect, and dynamic file-provider routing, run
under supervisord. The candy lives in `charly.yml` at the repo root plus its
static config.

Canonical files:

- `charly.yml` — the `traefik:` candy entity (description, `require`, `var`,
  `port`, `volume`, `service`, `plan`) and its `skill:` entity.
- `traefik.yml` — the static config copied to `/etc/traefik/traefik.yml`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:traefik` — the owning skill: the candy properties, the
  entrypoints, and the ACME resolver. Load before editing, building, deploying,
  or troubleshooting this candy.
- `/charly-infrastructure:supervisord` — the process-manager dependency.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `download:` / `copy:` / `run:` / `check:`, service
  declarations).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; ports, volumes, services).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert the installed binary reports `3.4.0`, the
  staged config with the `letsencrypt` resolver and resolved `/home/` ACME
  storage path, the dynamic-provider directory, and — at deploy scope — the API
  HTTP 200, the running service, and the reachable entrypoints.

## Modify this repo

- Edit the `traefik:` candy entity in `charly.yml`; the `skill:` entity in the
  same file is the owning skill's source — a candy change and its skill change
  land together.
- The `TRAEFIK_VERSION` var drives both the download URL and the version check;
  bump them together.
- Keep the `sed` step that resolves `${HOME}` in the ACME storage path — Traefik
  does not expand env vars in static config, and the `traefik-acme-storage-resolved`
  check asserts the `/home/` result.
- The `skill:` entity is the source for `/charly-infrastructure:traefik`; never
  edit the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
