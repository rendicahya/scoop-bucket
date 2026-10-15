# CLAUDE.md - personal Scoop bucket

This repository is the `personal` Scoop bucket of `rendicahya`
(<https://github.com/rendicahya/scoop-bucket>). It holds Scoop manifests and
their documentation only. Each application keeps its source, releases, and
release notes in its own repository.

Read `CONTRIBUTING.md` for the full policies, manifest conventions, and
validation steps. This file is the short version for maintenance work.

## Layout

- `bucket/<app>.json`: one manifest per application. Currently only
  `resik.json` (<https://github.com/rendicahya/resik>).
- `bin/`: thin wrappers around Scoop's maintenance scripts (`test.ps1`,
  `checkver.ps1`, `checkurls.ps1`, `checkhashes.ps1`, `formatjson.ps1`).
- `Scoop-Bucket.Tests.ps1`: loads Scoop's bucket test suite.
- `.github/workflows/validate-manifests.yml`: runs that suite on Windows
  PowerShell 5.1 and PowerShell 7. Actions and Scoop are pinned to commit SHAs.

## Rules

- Never invent a URL, version, or hash. A manifest's `hash` is the SHA-256 of
  the asset downloaded from the published release, never of a local build
  (zip archives embed timestamps, so rebuilds differ).
- `version` must match the release that `url` points to.
- Every file must be UTF-8 without BOM, CRLF line endings, 4-space indentation
  (2 for YAML), no trailing whitespace, and a final newline. The test suite
  checks every file in the repository, not only manifests.
- Manifests stay declarative (`bin`, `persist`, `checkver`, `autoupdate`); no
  installer scripts that delete user data or touch files outside the app.
- Do not add manifests for applications without a published release.

## Updating a manifest after an upstream release

1. Confirm the release exists and download its asset into an empty folder:
   `gh release download v<version> -R rendicahya/<app> --pattern '<asset>' --dir <dir>`.
2. Set `version`, `url`, and `hash` (`(Get-FileHash <file> -Algorithm SHA256).Hash.ToLower()`).
   `.\bin\checkver.ps1 <app> -Update` can do this through `autoupdate`; still
   compare its hash with the downloaded asset.
3. Check: `.\bin\checkver.ps1 <app>` (reports the new version) and
   `.\bin\checkurls.ps1 <app>`.
4. Test the install. Resik is usually installed on this machine from the
   bucket, so uninstall it first and reinstall from the bucket afterwards:
   `scoop uninstall <app>`, `scoop install .\bucket\<app>.json`, run the app
   (for Resik: `resik --version`, `resik.cmd --version`, and a `--dry-run`),
   then `scoop uninstall <app>`.
5. Commit with the message `<app>: Update to version <version>` (the style
   used in the history), push, and check that the "Validate manifests"
   workflow passes for the pushed commit.
6. Reinstall from the bucket: `scoop update`, then
   `scoop install rendicahya_scoop-bucket/<app>` (this machine registers the
   bucket under that name; users add it as `personal`).

## Git workflow

Commit directly to `main`; there are no feature branches or pull requests.
Ask for explicit approval before every commit and push. Never force-push or
rewrite history without explicit authorization, and never claim a check passed
unless it ran and its output was read.
