# Contributing

This guide covers how to add or update an application in this bucket and how to validate the change. Read the [README](README.md) first for an overview of how the bucket works.

Official Scoop references:

- [App Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests): every manifest property and its format
- [Creating an App Manifest](https://github.com/ScoopInstaller/Scoop/wiki/Creating-an-app-manifest)
- [Buckets](https://github.com/ScoopInstaller/Scoop/wiki/Buckets)
- [Scoop repository](https://github.com/ScoopInstaller/Scoop), including the [manifest JSON Schema](https://github.com/ScoopInstaller/Scoop/blob/master/schema.json)

## Repository policies

- This repository holds Scoop manifests and bucket documentation only. It doesn't hold application source code.
- Each application keeps its own source repository and release process. Release workflows belong in the application's repository, not here.
- Each manifest must reference a real, publicly accessible upstream release artifact.
- SHA-256 hashes must be checked against the artifact they describe.
- Each application has one manifest. Don't duplicate manifests.
- Don't commit application binaries to this repository. If a binary ever needs to be hosted here, document the reason in the commit message and in the manifest's `##` comment.
- Manifests must not install or uninstall destructively. Don't delete user data, change unrelated system settings, or touch files outside the application's own directory without a documented reason.
- Keep manifests straightforward and easy to maintain.
- Don't add placeholder manifests for applications that might exist in the future. Add a manifest only after the application has a published release artifact.

## Workflow

This repository uses a simple workflow: the maintainer commits directly to `main`, without feature branches or pull requests.

1. **Start from an up-to-date `main`.**

   ```powershell
   git switch main
   git pull
   ```

2. **Add or update a manifest in `bucket/`.** Create or edit `bucket/<app-name>.json` following the [manifest conventions](#manifest-conventions) below.

3. **Validate the JSON and Scoop manifest conventions.** Run the checks in [Local validation on Windows](#local-validation-on-windows).

4. **Verify that release URLs are accessible and that SHA-256 hashes match the actual artifacts.** See [URLs and hashes](#3-urls-and-hashes).

5. **Test installation, execution, updating, and uninstallation, when feasible.** See [Installation testing](#5-installation-testing).

6. **Commit and push to `main`.** Review the diff first. In the commit message, name the application and version (for example, `Add resik 0.1.0` or `Update resik to 0.2.0`), and mention any unusual installation behavior.

   ```powershell
   git diff
   git add bucket/<app-name>.json
   git commit -m "Add <app-name> <version>"
   git push
   ```

7. **Check CI.** Confirm that the **Validate manifests** workflow passes for the pushed commit. If it fails, fix the manifest in a follow-up commit.

External contributors who can't push to `main` can open a pull request instead; the same checks apply.

## Manifest conventions

- **One manifest per application.** Name it `bucket/<app-name>.json`. Users type the file name (minus `.json`) when they run `scoop install`.
- **Use lowercase file names where practical**, with hyphens between words (for example, `my-tool.json`).
- **Follow the official Scoop manifest conventions** in [App Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests). Manifests must validate against Scoop's JSON Schema, which CI checks.
- **Keep version numbers in sync with upstream releases.** `version` must match the release that `url` points to.
- **Use real, accessible release URLs.** Prefer stable, versioned URLs, such as GitHub release assets, over "latest" links that change over time.
- **Use verified SHA-256 hashes.** Compute the hash from the artifact itself, never by copying it from somewhere unverified.
- **Include suitable metadata:**
  - `description`: a short, single-sentence description of the app
  - `homepage`: the application's upstream repository or website
  - `license`: an [SPDX identifier](https://spdx.org/licenses/) such as `MIT`, or an object with `identifier` and `url`
  - `architecture`: use this when there are separate builds such as `64bit`, `32bit`, or `arm64`
  - `bin` or `shortcuts`: the commands or Start menu entries to expose
  - `persist`: settings or data that must survive an update
  - `checkver` and `autoupdate`: recommended so that `bin\checkver.ps1` can detect new upstream releases
- **Avoid unnecessary installer scripts and side effects.** Prefer declarative properties (`bin`, `shortcuts`, `persist`, `extract_dir`, `env_add_path`) over `pre_install`, `installer`, `post_install`, or `uninstaller` scripts.
- **Never include credentials, tokens, or private URLs.** Every URL must be publicly downloadable without authentication.
- **Document unusual installation or uninstallation behavior.** Put user-facing information in `notes`, explain maintainer-facing reasoning in a `##` comment, and mention it in the commit message.
- **Formatting:** UTF-8 without a byte order mark (BOM), CRLF line endings, 4-space indentation, no trailing whitespace, and a final newline. `.editorconfig` and `.gitattributes` handle most of this, and the test suite enforces it. Running `.\bin\formatjson.ps1` applies Scoop's canonical JSON formatting.

### Manifest skeleton

The structure below is a common one for an application released as a ZIP on GitHub. It isn't a working manifest. Replace every `<...>` value with real data, and remove the properties that don't apply.

```json
{
    "version": "<version>",
    "description": "<One-sentence description.>",
    "homepage": "https://github.com/rendicahya/<repository>",
    "license": "<SPDX identifier>",
    "architecture": {
        "64bit": {
            "url": "https://github.com/rendicahya/<repository>/releases/download/v<version>/<artifact>.zip",
            "hash": "<sha256 of the artifact>"
        }
    },
    "bin": "<executable>.exe",
    "shortcuts": [
        [
            "<executable>.exe",
            "<Display Name>"
        ]
    ],
    "checkver": {
        "github": "https://github.com/rendicahya/<repository>"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/rendicahya/<repository>/releases/download/v$version/<artifact>.zip"
            }
        }
    }
}
```

## Local validation on Windows

Validation tooling is optional on your machine. The repository works without it, and CI runs the full schema validation on every push to `main`. Running the checks locally catches problems sooner.

All commands below run in PowerShell from the repository root.

### 1. JSON syntax (no extra tools)

PowerShell's built-in `ConvertFrom-Json` catches malformed JSON without installing anything:

```powershell
Get-ChildItem .\bucket -Filter *.json | ForEach-Object {
    $file = $_
    try {
        $null = Get-Content -Raw -LiteralPath $file.FullName | ConvertFrom-Json -ErrorAction Stop
        Write-Host "OK      $($file.Name)"
    } catch {
        Write-Host "INVALID $($file.Name): $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

This only checks JSON syntax. It doesn't check Scoop-specific rules.

### 2. Scoop manifest validation (official test suite)

`bin\test.ps1` runs Scoop's official bucket test suite, the same one CI uses. It validates every manifest in `bucket/` against Scoop's [JSON Schema](https://github.com/ScoopInstaller/Scoop/blob/master/schema.json) and checks file style (BOM, line endings, trailing whitespace, indentation) for every file in the repository.

It requires the following:

- Scoop installed. The suite is loaded from your Scoop installation through `scoop prefix scoop`, or from `$env:SCOOP_HOME` if you set it.
- The PowerShell modules Pester 5 and BuildHelpers. These are development-only tools. Install them once for your user:

  ```powershell
  Install-Module -Name Pester -MinimumVersion 5.2.0 -MaximumVersion 5.99 -Scope CurrentUser -Force -SkipPublisherCheck
  Install-Module -Name BuildHelpers -MinimumVersion 2.0.1 -Scope CurrentUser -Force
  ```

  Windows PowerShell 5.1 includes an old Pester 3.x. You need `-SkipPublisherCheck` to install Pester 5 next to it. Scoop's suite needs Pester 5, not Pester 6, which is why the commands set `-MaximumVersion 5.99`. If PowerShell asks to install the NuGet provider, accept the prompt. Run the commands in an interactive window, because the prompt can't appear in a non-interactive session.

Run the suite from a PowerShell prompt:

```powershell
.\bin\test.ps1
```

Check the summary line at the end of the output. `Failed: 0` means every check passed. Don't rely only on the exit code: if you launch the script with `powershell.exe -File`, Windows PowerShell 5.1 reports exit code `0` even when tests fail, because the script uses `#Requires -Modules`.

### 3. URLs and hashes

These scripts download the artifacts each manifest references and compare them with the manifest. They don't run any installer.

```powershell
.\bin\checkurls.ps1                  # Check that every URL in every manifest is reachable
.\bin\checkhashes.ps1 <app-name>     # Download the artifacts and compare their SHA-256 hashes
```

To compute a hash by hand from a downloaded artifact:

```powershell
(Get-FileHash -Algorithm SHA256 -LiteralPath .\<artifact>.zip).Hash.ToLower()
```

### 4. Upstream version and formatting

```powershell
.\bin\checkver.ps1 <app-name>            # Compare the manifest with the latest upstream release (needs "checkver")
.\bin\checkver.ps1 <app-name> -Update    # Update version, URL, and hash (needs "autoupdate"); review the diff afterwards
.\bin\formatjson.ps1                     # Apply Scoop's canonical JSON formatting to all manifests
```

`checkver.ps1 -Update` rewrites the manifest. Review the result with `git diff` and still run the hash check.

### 5. Installation testing

Unlike the other checks, this one installs the application and runs any scripts in the manifest. Only do it for manifests you have reviewed.

```powershell
scoop install .\bucket\<app-name>.json   # Install straight from the local manifest
<command>                                # Confirm the application starts and works
scoop uninstall <app-name>               # Confirm uninstallation is clean
```

To test updating, install the previous version (for example, `scoop install personal/<app-name>` before you push your change). After pushing, run `scoop update` followed by `scoop update <app-name>`. Also check that any `persist` data survives the update.

## Continuous integration

The [`.github/workflows/validate-manifests.yml`](.github/workflows/validate-manifests.yml) workflow runs on pushes to `main`, on pull requests, and on demand. It runs on `windows-latest` under both Windows PowerShell 5.1 and PowerShell 7, and does the following:

1. Checks out this repository and Scoop's source at a pinned release commit.
2. Installs Pester 5 and BuildHelpers from the PowerShell Gallery.
3. Runs `bin\test.ps1`, which validates **every** `*.json` file in `bucket/` against Scoop's JSON Schema and checks the style of every file. Malformed JSON or schema violations fail the build.

The workflow has read-only repository permissions, needs no secrets, and doesn't save Git credentials.

**Limitations:**

- CI doesn't download artifacts, check URLs, verify hashes, or install applications. This is deliberate, so that CI never runs installer scripts from manifests. Run steps 3 to 5 of the local validation above before pushing.
- Schema validation checks structure, not correctness. For example, it can't tell whether `version` matches the release that `url` points to.
- Scoop's validator uses Newtonsoft.Json.Schema, which allows 1,000 free schema validations per hour. If that limit is reached, the affected tests are reported as **skipped**, not failed. Check the test output if you see skipped tests.

**Maintaining the workflow:** GitHub Actions are pinned to full commit SHAs, and the Scoop test suite is pinned to a release commit. To update them, look up the commit for the new release tag, review the release notes, and update the SHA and the version comment together.
