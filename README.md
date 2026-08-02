# iaspis-ci

Build and test harness for the IASPIS engine.

Each workflow checks out an IASPIS commit over SSH with a read-only deploy key
and runs one gate against it. Nothing here is triggered by pushes or pull
requests — runs are started explicitly with a `ref` naming the commit to build.

| Workflow | Gate |
|---|---|
| `linux-headless.yml` | GCC Release build of `iaspis_engine` + `gamesdk`, doctest suite, `r2t_bake` |
| `us-ci.yml` | Standalone `iaspis_ultrasound` tests on a software Vulkan device (lavapipe) |
| `android-xr-debug.yml` | Meta Quest arm64-v8a debug APK |

## Running a build

One gate:

```sh
gh workflow run linux-headless.yml --repo Giannossk/iaspis-ci -f ref=<sha-or-branch>
```

All three at once:

```sh
printf '{"event_type":"build","client_payload":{"ref":"<sha-or-branch>"}}' \
  | gh api repos/Giannossk/iaspis-ci/dispatches --input -
```

From an IASPIS checkout, `tools/ci_dispatch.sh` wraps both forms and refuses to
dispatch a commit that has not been pushed.

`linux-headless.yml` also runs nightly against `main` at 03:00 UTC.

## Secrets

Each secret is the private half of an ed25519 **read-only deploy key** on one
repository. A deploy key grants read access to exactly that repository, cannot
write, and is revoked from the owning repository's Settings → Deploy keys.

| Secret | Repository |
|---|---|
| `IASPIS_DEPLOY_KEY` | `Giannossk/IASPIS` |
| `SHADERS_DEPLOY_KEY` | `Giannossk/IASPIS-Shaders` (submodule) |
| `PROFILE_BSS_DEPLOY_KEY` | `Giannossk/IASPIS-Basic_Skills` |
| `PROFILE_CHOLE_DEPLOY_KEY` | `Giannossk/IASPIS-Chole` |
| `PROFILE_APPEND_DEPLOY_KEY` | `Giannossk/IASPIS-Append` |

A key can only be a deploy key on one repository, so each private repository the
build touches gets its own. The Android job maps them onto per-repository SSH
host aliases and rewrites the corresponding HTTPS URLs with `url.insteadOf`, so
the submodule and the CMake-driven profile clone each pick the right key.

Only `IASPIS_DEPLOY_KEY` is needed by `linux-headless.yml` and `us-ci.yml`.
