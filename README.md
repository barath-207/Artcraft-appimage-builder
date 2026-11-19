# PdfCraft AppImage Builder

This repository builds the latest `main` commit from `storytold/pdfcraft` on GitHub-hosted Ubuntu 22.04 runners and publishes the resulting x86_64 AppImage as a public GitHub Release.

## Setup

1. Create a **public** GitHub repository, for example `Barath702/pdfcraft-appimage-builder`.
2. Upload this repository's contents, preserving `.github/workflows/build.yml`.
3. In the new repository, open **Settings → Actions → General → Workflow permissions**. The workflow declares its own `contents: write` permission for the publishing job; the default read-only setting is fine because the job-level permission overrides it.
4. Open **Actions → Build PdfCraft AppImage** and enable workflows if GitHub asks.
5. Choose **Run workflow** for the first build. Leave `force` unchecked unless you deliberately want to rebuild a commit already published.
6. Wait for both `build` and `publish` jobs to finish successfully. The first build can take a while; Cargo caching helps later builds.

## Public download URLs

After the first successful release, the stable asset URL is:

`https://github.com/OWNER/REPO/releases/latest/download/PdfCraft-latest-x86_64.AppImage`

The release also contains the versioned upstream-named AppImage, its `.zsync` file, `UPSTREAM-COMMIT.txt`, and `SHA256SUMS.txt`.

## Automation

The workflow checks upstream `main` daily at 03:20 UTC (08:50 India time). It creates one release per upstream commit and skips commits already published. `workflow_dispatch` supports a `force` option to make a fresh release for the same upstream commit.

## Important notes

- The build runs on Ubuntu 22.04 to target glibc 2.35 compatibility, following PdfCraft's packaging guidance.
- It uses PdfCraft's own `packaging/linux/package.sh --formats appimage`, not a hand-built AppDir.
- It checks out the font and translation repositories at the pinned revisions used by the current upstream PdfCraft release workflow.
- The AppImage's embedded zsync update metadata points to **this builder repository**, because the packaging script reads `GITHUB_REPOSITORY`. Keep the versioned `.AppImage.zsync` asset attached to each release.
- Builds are unofficial and should be labelled as such. Review upstream licensing and branding rules before redistributing. Do not imply endorsement by the PdfCraft/ArtCraft maintainers.
- The workflow separates compilation from release publishing. The build job has read-only permissions; only the separate publishing job receives `contents: write`.
- GitHub-hosted Actions usage limits and repository policies apply. Public repositories generally receive free standard-runner minutes, subject to GitHub's current terms.

## Troubleshooting

- If `CRAFT_FONTS_REQUIRED` or `CRAFT_I18N_REQUIRED` fails, check whether upstream changed the pinned companion repositories or their required revisions.
- If packaging says no `.zsync` was produced, check that the `zsync` package installed successfully and that `zsyncmake` is on PATH.
- If upstream changes the package script or required Ubuntu libraries, update this workflow to match the upstream Linux release job.
- If a build fails, no new release is published; the previous `latest` release remains available.
