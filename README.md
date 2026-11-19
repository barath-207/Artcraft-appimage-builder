# ArtCraft Single-Release AppImage Builder

This repository builds the latest upstream `main` commit for five ArtCraft apps and publishes **exactly five stable x86_64 AppImage assets** in one rolling GitHub Release.

## Applications

| App | Upstream repository | Stable release asset |
|---|---|---|
| PdfCraft | `storytold/pdfcraft` | `PdfCraft-latest-x86_64.AppImage` |
| PhotoCraft | `storytold/photocraft` | `PhotoCraft-latest-x86_64.AppImage` |
| GridCraft | `storytold/gridcraft` | `GridCraft-latest-x86_64.AppImage` |
| FilmCraft | `storytold/filmcraft` | `FilmCraft-latest-x86_64.AppImage` |
| WordCraft | `storytold/wordcraft` | `WordCraft-latest-x86_64.AppImage` |

## How it works

- Runs daily at 03:20 UTC (08:50 India time).
- Resolves the latest upstream `main` commit for each app.
- Reads the last-built commit map from the rolling release notes.
- Builds only apps whose upstream commit changed.
- Uses independent matrix jobs with `fail-fast: false`.
- Uses each upstream repository's own `packaging/linux/package.sh --formats appimage`.
- Builds on Ubuntu 22.04 for the glibc 2.35 compatibility floor.
- Uses Cargo caching per application.
- Smoke-tests the produced AppImage.
- Updates only changed assets in one release tagged `artcraft-appimages`.
- Leaves the other four assets untouched.
- Uses the repository's built-in `GITHUB_TOKEN`; no PAT or cross-repository secret is needed.

## One-time setup

1. Keep this repository public.
2. Back up the current PdfCraft workflow and README.
3. Replace `.github/workflows/build.yml` with this package's workflow.
4. Update the README.
5. Commit and push.
6. Open Actions → Build ArtCraft AppImages → Run workflow.
7. Select `all`, leave `force` unchecked, and run.

The publisher job declares `contents: write`; the build jobs have read-only permissions. The repository's default workflow permission can remain read-only because the publish job explicitly elevates its own permission.

## Manual runs

- `app=all`: check and build all five.
- Select one app: check/build only that app.
- `force=true`: rebuild selected apps even if the upstream SHA has not changed.

## Release

The workflow creates or updates one release:

`https://github.com/OWNER/REPO/releases/tag/artcraft-appimages`

Stable download URLs:

```text
https://github.com/OWNER/REPO/releases/latest/download/PdfCraft-latest-x86_64.AppImage
https://github.com/OWNER/REPO/releases/latest/download/PhotoCraft-latest-x86_64.AppImage
https://github.com/OWNER/REPO/releases/latest/download/GridCraft-latest-x86_64.AppImage
https://github.com/OWNER/REPO/releases/latest/download/FilmCraft-latest-x86_64.AppImage
https://github.com/OWNER/REPO/releases/latest/download/WordCraft-latest-x86_64.AppImage
```

The release notes retain a machine-readable upstream commit map. This avoids adding metadata files as release assets: the five AppImages are the only uploaded binary assets.

GitHub itself also displays its automatic source-code ZIP and TAR.GZ links; those are not uploaded application assets.

## Important update behaviour

This design is for ArtCraft Launcher or another updater that downloads the stable URLs above. It does **not** publish `.zsync` files. AppImageUpdate's embedded zsync feed is separate and may still point to the upstream project. Do not use AppImageUpdate as the update mechanism for these custom builds unless its embedded update information is deliberately changed and separately tested.

## PdfCraft compatibility

The original PdfCraft workflow's core build choices are retained:
- upstream `main` SHA pinning;
- Ubuntu 22.04;
- pinned Craft fonts revision `b076417b5880bd4154bd22f00695b1d28c6bea83`;
- pinned Craft translations revision `575cfdef63227b14daf173b8cc87928bfdfa325e`;
- stable Rust and Cargo cache;
- upstream Linux packaging script;
- AppImage runtime smoke test.

The workflow does not delete the old PdfCraft release or any old assets. It creates a new rolling release on the first successful run. Review the current upstream packaging changes if their build inputs or environment variable names change.

## Important validation note

The five upstream projects have separate packaging scripts and evolve independently. This workflow uses the same established Linux packaging entry point for each. The first run is the integration test: inspect each app's build log and correct any app-specific dependency or build-input change reported by upstream. A failed matrix job does not cancel successful app builds, but the publish job is gated on the matrix completing successfully to avoid publishing a partial batch accidentally.
