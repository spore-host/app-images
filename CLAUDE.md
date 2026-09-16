# CLAUDE.md — app-images

Container images for spore-host/spawn web apps, published to
`ghcr.io/spore-host/<app>`. Each app is a directory `images/<app>/` with a
`Dockerfile`. The README is also the "bring your own app to spawn" tutorial.

## Web-app image rules (why the Dockerfiles look the way they do)
spawn runs the container publishing `127.0.0.1:<port>:<port>` and fronts it with
spored's `:443` TLS reverse proxy, which gates access with a token. So an image
must: (1) bind `0.0.0.0` (not localhost), (2) run auth-less (the proxy token is
the gate), (3) serve on a known port. See the README.

## Versioning & changelog (required)
Follows **SemVer 2.0.0** + **Keep a Changelog** (`CHANGELOG.md`), the
spore.host-wide policy. Update `## [Unreleased]` in the same PR as any change.

**On release of an image:** promote `## [Unreleased]` → a dated section, then tag
`<app>-v<version>` (e.g. `openrefine-v3.10.1`) — the `Publish image` workflow
builds `linux/amd64` and pushes `ghcr.io/spore-host/<app>:<version>` + `:latest`.
First publish of a new app: **make the GHCR package public** (Package settings →
visibility) — spawn's catalog requires public, anonymously-pullable images.

## Build & test
- `docker build -t <app> images/<app>` then
  `docker run --rm -p 127.0.0.1:<port>:<port> <app>` and curl the port.
