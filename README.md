# app-images

Container images for [spore-host/spawn](https://github.com/spore-host/spawn) web
apps — and a tutorial for **bringing your own app to spawn when it has no official
container image**.

Each app lives under `images/<app>/` and is published to
`ghcr.io/spore-host/<app>` by [`.github/workflows/publish.yml`](.github/workflows/publish.yml).

| App | Image | Port | Notes |
|-----|-------|------|-------|
| [OpenRefine](images/openrefine/) | `ghcr.io/spore-host/openrefine` | 3333 | Data cleaning/wrangling. No official image — built from the upstream release. |

---

## Tutorial: bring your own app to spawn

`spawn app launch` streams two kinds of app: **DCV** GUI apps and **web** apps
(ones that serve their own HTTP UI — Jupyter, code-server, OpenRefine…). If your
web app already has a good public image, you can launch it directly:

```sh
spawn app launch myapp --image someorg/myapp --web-port 8080
```

But some apps — like **OpenRefine** — are actively maintained yet ship **no
official container image**, and the community images are stale. This repo shows
the fix: build a small, trustworthy image from the app's official release, publish
it publicly, and (optionally) add it to the catalog. OpenRefine is the worked
example.

### The three rules for a spawn web-app image

spawn runs your container on an ephemeral instance, publishes its port on
localhost, and fronts it with **spored's `:443` TLS reverse proxy**. The proxy
gates access with a one-time token (`https://<host>/?spore_token=…` → a
`Secure; HttpOnly` cookie), so **the token is the access control** — you don't
need the app's own auth. That leads to three rules:

1. **Bind `0.0.0.0`, not localhost.** The instance publishes `127.0.0.1:<port>:<port>`,
   so the app inside the container must listen on `0.0.0.0:<port>` or the
   published port is unreachable.
2. **Run auth-less.** Turn the app's own login off — the spored proxy token
   already gates it. (Leaving app auth on just means two prompts.)
3. **Serve on a known port**, which you'll put in the catalog entry.

### Worked example: OpenRefine

[`images/openrefine/Dockerfile`](images/openrefine/Dockerfile) installs the
official OpenRefine release on a JRE base and runs it per the rules above:

```dockerfile
FROM eclipse-temurin:21-jre
ARG OPENREFINE_VERSION=3.10.1
# … download + extract the official release …
EXPOSE 3333
# -i 0.0.0.0  → bind all interfaces (rule 1); OpenRefine has no auth (rule 2)
CMD ["./refine","-i","0.0.0.0","-p","3333","-d","/data","-m","4g"]
```

Build and test it locally before publishing:

```sh
docker build -t openrefine images/openrefine
docker run --rm -p 127.0.0.1:3333:3333 openrefine
curl -s http://127.0.0.1:3333/ | grep -i '<title>'   # → <title>OpenRefine</title>
```

### Publish it

Tag the repo `<app>-v<version>` (or run the **Publish image** workflow manually);
CI builds `linux/amd64` and pushes to GHCR:

```sh
git tag openrefine-v3.10.1 && git push origin openrefine-v3.10.1
# → ghcr.io/spore-host/openrefine:3.10.1  (+ :latest)
```

**Make the package public** the first time (GHCR packages start private): repo →
*Packages* → the package → *Package settings* → *Change visibility → Public*.
spawn's catalog requires public, anonymously-pullable images.

### Launch it

Directly, from anywhere:

```sh
spawn app launch openrefine --image ghcr.io/spore-host/openrefine --web-port 3333
```

Or, if it's in the catalog (`spawn app list` shows it), just:

```sh
spawn app launch openrefine
```

spawn resolves an instance, pulls the image, waits for `:3333` to answer, brings
up the TLS proxy, and opens `https://<host>/?spore_token=…` in your browser.

### Add it to the shipped catalog (optional)

To make an app a built-in (`spawn app list` for everyone), add a `kind: web`
entry to [`libs/catalog/catalog.yaml`](https://github.com/spore-host/libs). The
image must be **public**. See spawn's
[`docs/catalog-schema.md`](https://github.com/spore-host/spawn/blob/main/docs/catalog-schema.md)
for every field. OpenRefine's entry:

```yaml
- name: openrefine
  kind: web
  port: 3333
  image: ghcr.io/spore-host/openrefine
  tag_default: "3.10.1"
  visibility: public
  gpu: false
  instance_families: [c7i, m7i]
```

No `args:` — the image's `CMD` already binds `0.0.0.0`. (An app whose *stock*
image binds localhost or requires a token uses `args:` to override, e.g.
code-server's `[--bind-addr, 0.0.0.0:8080, --auth, none]`.)

---

## Adding another app

1. `images/<app>/Dockerfile` following the three rules.
2. Build + test locally.
3. Tag `<app>-v<version>` → CI publishes `ghcr.io/spore-host/<app>` → make it public.
4. (Optional) add a `kind: web` catalog entry in libs.

## License

The build recipes in this repo are under [MIT-0](LICENSE). Each image bundles
upstream software under its own license (OpenRefine: BSD-3-Clause).
