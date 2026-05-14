# gameloop-optimizer-releases

Public release repository for **GameLoop Optimizer (client edition)**. Only
purpose: host the packaged `.exe` and its SHA-256 manifest so the in-app
auto-updater can find newer versions.

The source code lives in a **separate private repo** — this repo is downloads-only.

---

## How the client finds updates

`client_app/client_ui/updater.py` hits:

```
https://api.github.com/repos/<OWNER>/gameloop-optimizer-releases/releases/latest
```

If the tag is newer than the embedded `CURRENT_VERSION`, the client:

1. Reads the `latest.json` asset to get the expected SHA-256.
2. Downloads the `.exe` asset.
3. Verifies SHA-256 — bails on mismatch.
4. Writes `_update_swap.bat` next to the running .exe; the `.bat` waits
   2 seconds, replaces the .exe, relaunches it, and self-deletes.

---

## Release layout

A release tagged `v0.2.0` ships TWO assets:

| Asset | Purpose |
|---|---|
| `GameLoopOptimizer-0.2.0.exe` | The actual binary the client downloads |
| `latest.json` | Manifest carrying version + SHA-256 |

Example `latest.json`:

```json
{
  "version": "0.2.0",
  "released_at": "2026-05-12T19:30:00Z",
  "asset_name": "GameLoopOptimizer-0.2.0.exe",
  "sha256": "abcd1234…0123456789abcdef0123456789abcdef0123456789abcdef01234567",
  "notes": "What's new in this release."
}
```

---

## Cutting a release (manual)

1. Build the .exe on the dev box:
   ```
   pwsh ./tools/release.ps1 -Version 0.2.0
   ```
   Produces `dist/GameLoopOptimizer-0.2.0.exe` and `dist/latest.json`.

2. On this repo's GitHub page:
   - Releases → Draft a new release
   - Tag: `v0.2.0`
   - Title: `v0.2.0`
   - Body: changelog
   - Drag-drop the `.exe` and `latest.json` from `dist/`
   - Publish

The next time a client app launches it'll auto-check and offer the update.

---

## Cutting a release (automated, via GitHub Actions)

The `.github/workflows/release.yml` workflow handles building + uploading.

1. Push a `v*` tag to the dev (private) repo. The workflow's `repository_dispatch`
   trigger calls into this public repo with the new version.
2. CI runs PyInstaller, computes SHA-256, writes manifest, creates a release
   in this repo with both assets attached.

You need to set:

- `RELEASE_PAT` — a Personal Access Token with `repo` scope on this repo,
  set as a secret in the dev repo.

See [`tools/release.ps1`](../tools/release.ps1) and
[`.github/workflows/release.yml`](.github/workflows/release.yml).

---

## Why a separate repo?

- The dev repo is private (action specs, registry tweaks, telemetry creds).
- The releases need to be public so the in-app updater can pull them
  without credentials.
- Splitting them keeps the public surface tiny — just `.exe` + manifest.
