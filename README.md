# pod-traefik

The `traefik` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It provides a reverse proxy
with TLS termination, HTTP-to-HTTPS redirect, and dynamic file-provider routing.

## What it provides

Downloads the pinned Traefik `v3.4.0` release binary to `/usr/local/bin/traefik`
and stages a static config at `/etc/traefik/traefik.yml`: a `web` entrypoint on
`:8000` that redirects to HTTPS, a `websecure` entrypoint on `:8443` backed by
the letsencrypt ACME resolver, an insecure API/dashboard on `:8080`, and a
watched dynamic-provider directory at `/etc/traefik/dynamic`. It runs as a
supervisord service.

| Property | Value |
|---|---|
| Requires | `layer-supervisord` |
| Ports | `8000`, `8080`, `8443` |
| Volume | `certs` → `~/.traefik/acme` |
| Service | `traefik` (`/usr/local/bin/traefik --configFile=/etc/traefik/traefik.yml`, `restart: always`) |
| Pinned version | `v3.4.0` |

Traefik does not expand env vars in static config, so the build resolves the
ACME storage `${HOME}` to an absolute `/home/` path before the config is written.

## How to use it

```yaml
my-image:
  candy:
    - '@github.com/opencharly/pod-traefik:<tag>'
```

## Verification

The candy's `check:` plan asserts the installed binary reporting `3.4.0`, the
staged config with the `letsencrypt` resolver and the resolved `/home/` ACME
storage path, the dynamic-provider directory, and — at deploy scope — the API
answering HTTP 200 on the API port, the running `traefik` service, and the
reachable `:8000` and `:8443` entrypoints.

## Layout

- `charly.yml` — the `traefik:` candy entity (description, `require`, `var`,
  `port`, `volume`, `service`, `plan`) plus its `skill:` entity.
- `traefik.yml` — the static config copied to `/etc/traefik/traefik.yml`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:traefik` — the candy properties, the
  entrypoints, and the ACME resolver.
- `/charly-infrastructure:supervisord` — the process-manager dependency.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
