# vert-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Vert](https://github.com/vert-sh/vert).

## What this is

A four-file TOS Docker application archive (`vertdocker.tar.gz`) that installs
Vert on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `vertdocker`)
- `vertdocker.lang` — 23-language superset (TOS requires 14)
- `vertdocker.svg` — application icon (274 bytes, 2 SVG elements)
- `docker-compose.yml` — one service, image `moechz/vert:2026.11.14`

## How it works

| | |
|---|---|
| Container image | `moechz/vert:2026.11.14` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8150/` (container port 80) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/app/.data` → `/Volume1/DockerAppData/vertdocker/data`, `/var/cache/nginx` → `/Volume1/DockerAppData/vertdocker/cache`, `/var/log/nginx` → `/Volume1/DockerAppData/vertdocker/log`, `/run` → `/Volume1/DockerAppData/vertdocker/run` |
| Logs | `docker logs vertdocker` (stdout/stderr) |



## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 80.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/vertdocker/data` | Vert application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/vertdocker/cache` | Vert application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/vertdocker/log` | Vert application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/vertdocker/run` | Vert application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/vertdocker.tar.gz + .sha256
scripts/build.sh 2026.11.14-2   # packaging iteration bump
scripts/build.sh 2026.11.14-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/vertdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v2026.11.14-1`**, assets `vertdocker.tar.gz` +
   `vertdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `vertdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `2026.11.14-1` → automated validation → review.

## License / branding

Vert is distributed by its upstream authors under **AGPL-3.0**. The container image is the official image from Docker Hub (`moechz/vert:2026.11.14`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The application runs inside a container; data lives in the application data directory (`/Volume1/DockerAppData/vertdocker/`) as described above.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
