# emsdk mirror

Mirrors [`emscripten/emsdk`](https://hub.docker.com/r/emscripten/emsdk) tags
from Docker Hub into GHCR so CI does not break when Docker Hub returns
`502 Bad Gateway` (a recurring problem in the core repo):

```
Error response from daemon: Head "https://registry-1.docker.io/v2/emscripten/emsdk/manifests/5.0.4": received unexpected HTTP status: 502 Bad Gateway
```

## What it produces

```
ghcr.io/euro-office/emsdk:<tag>
```

The [`emsdk-mirror.yml`](../.github/workflows/emsdk-mirror.yml) workflow copies
the upstream manifest with `docker buildx imagetools create`. It is a pure copy,
not a rebuild: the mirrored image keeps the same digests and platforms as
upstream.

## Using the mirror in CI

Replace the upstream reference, e.g.:

```diff
- emscripten/emsdk:5.0.4
+ ghcr.io/euro-office/emsdk:5.0.4
```

## Adding / changing mirrored tags

Edit [`tags.txt`](./tags.txt) (one tag per line) and push to `main`. The
workflow re-runs automatically on changes to this directory.

## When it runs

- **Weekly** (Monday 04:00 UTC) to refresh moving tags like `latest` and
  re-copy pinned tags.
- **On push to `main`** that touches this directory or the workflow.
- **Manually** via *Run workflow*, optionally passing a single tag to mirror
  on demand without editing `tags.txt`.
