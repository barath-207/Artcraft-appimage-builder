# ArtCraft Multi-Repository AppImage Builder

This repository builds the latest upstream `main` commit for five ArtCraft applications and publishes each AppImage to its own GitHub repository and release. The central builder keeps one shared GitHub Actions workflow; each application has independent release history and update tracking.

## Applications

| App | Upstream repository | AppImage repository | Release asset |
|---|---|---|---|
| PdfCraft | `storytold/pdfcraft` | `barath-207/pdfcraft-appimage` | `PdfCraft-latest-x86_64.AppImage` |
| PhotoCraft | `storytold/photocraft` | `barath-207/photocraft-appimage` | `PhotoCraft-latest-x86_64.AppImage` |
| GridCraft | `storytold/gridcraft` | `barath-207/gridcraft-appimage` | `GridCraft-latest-x86_64.AppImage` |
| FilmCraft | `storytold/filmcraft` | `barath-207/filmcraft-appimage` | `FilmCraft-latest-x86_64.AppImage` |
| WordCraft | `storytold/wordcraft` | `barath-207/wordcraft-appimage` | `WordCraft-latest-x86_64.AppImage` |

## How it works

- Runs daily at 03:20 UTC (08:50 India time).
- Resolves the latest upstream `main` commit for each selected app.
- Compares that commit with the `upstream_commit` recorded in that app repository's latest GitHub release notes.
- Builds only apps whose upstream commit changed, unless `force=true`.
- Uses independent matrix jobs with `fail-fast: false`.
- Uses each upstream repository's `packaging/linux/package.sh --formats appimage`.
- Builds on Ubuntu 22.04.
- Uses Cargo caching per application.
- Uses pinned Craft fonts and PdfCraft translations.
- Publishes only changed apps to their corresponding AppImage repositories.
- Uses date-based release tags such as `v2026.11.20`.
- Uploads one AppImage and `SHA256SUMS` to each app's release.
- Does not require five separate build workflows.

## One-time GitHub setup

### 1. Create the five destination repositories

Create these repositories under the `barath-207` account:

- `pdfcraft-appimage`
- `photocraft-appimage`
- `gridcraft-appimage`
- `filmcraft-appimage`
- `wordcraft-appimage`

For the simplest AppGlider discovery and download experience, make them public.

### 2. Create a fine-grained Personal Access Token

Create a fine-grained GitHub token with access to exactly those five destination repositories.

Grant this repository permission:

- **Contents: Read and write**

No Issues, Pull requests, Actions, Administration, or other permissions are required by this workflow.

The token must be able to read releases and create/edit releases in all five destination repositories.

### 3. Add the token to the builder repository

In `barath-207/pdfcraft-appimage-builder`:

1. Open Settings.
2. Open Secrets and variables → Actions.
3. Select New repository secret.
4. Name it `ARTCRAFT_RELEASE_TOKEN`.
5. Paste the fine-grained token and save.

The builder repository itself can remain public. Never put the token directly in YAML or README.

### 4. Commit the workflow

Replace `.github/workflows/build.yml` with the supplied workflow and commit it to the default branch.

The builder repository's default workflow permissions can remain read-only. The cross-repository publishing step authenticates using `ARTCRAFT_RELEASE_TOKEN`.

## Manual runs

Open Actions → Build ArtCraft AppImages → Run workflow.

- `app=all`: check and build all five apps.
- Select one app: check/build only that app.
- `force=false`: build only when the upstream commit differs from that app's latest release.
- `force=true`: rebuild the selected app or all five apps even when upstream commits are unchanged.

A successful build publishes the changed application to its own repository. Unchanged applications are not republished.

## Release format

Each destination repository receives its own release:

`vYYYY.MM.DD`

For example:

`https://github.com/barath-207/pdfcraft-appimage/releases/latest`

The PdfCraft release contains:

- `PdfCraft-latest-x86_64.AppImage`
- `SHA256SUMS`

The other four repositories follow the same pattern with their own AppImage filename.

The release notes contain a machine-readable line:

`upstream_commit=<40-character-commit-sha>`

The workflow reads this value to decide whether a new build is needed.

## AppGlider

Add each destination repository separately in AppGlider using its repository URL or `owner/repository`:

- `barath-207/pdfcraft-appimage`
- `barath-207/photocraft-appimage`
- `barath-207/gridcraft-appimage`
- `barath-207/filmcraft-appimage`
- `barath-207/wordcraft-appimage`

Each repository has its own AppImage release stream, allowing AppGlider to manage each app independently. Use stable release tags in the `vYYYY.MM.DD` format.

## Build compatibility

The workflow retains these build choices from the existing builder:

- Upstream `main` SHA pinning.
- Ubuntu 22.04.
- Pinned Craft fonts revision `b076417b5880bd4154bd22f00695b1d28c6bea83`.
- Pinned Craft translations revision `575cfdef63227b14daf173b8cc87928bfdfa325e`.
- Stable Rust and per-app Cargo cache.
- Each upstream project's Linux AppImage packaging script.
- x86_64 AppImage asset naming.

The upstream projects have separate packaging scripts and evolve independently. Treat the first run for each application as an integration test and inspect its build log for app-specific dependency or build-input changes.

## Important behaviour and limitations

- A failed matrix build prevents the publishing job from running, avoiding a partial batch.
- A manual run for one app only checks and publishes that app.
- The workflow does not delete old releases.
- A new release on the same India-local date updates that day's release tag and replaces its AppImage/checksum assets.
- The workflow does not create `.zsync` files. AppImageUpdate's embedded update feed is separate and must not be assumed to point to these custom releases.
- AppGlider compatibility should be verified by adding each repository and checking that it detects the versioned AppImage release.
