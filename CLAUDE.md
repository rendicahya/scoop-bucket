# Task: Set Up My Personal Scoop Bucket

You are working inside my existing local Git repository for a personal Scoop bucket.

## Project Information

- GitHub username: `rendicahya`
- Repository name: `scoop-bucket`
- Repository URL: `https://github.com/rendicahya/scoop-bucket`
- Bucket name used by consumers: `personal`
- Purpose: Distribute multiple Windows applications through Scoop.
- Target platform: Windows 10 and Windows 11.
- Package manager: Scoop.

Each application will have its own source-code repository and release process. This repository is exclusively for Scoop manifests and bucket-maintenance documentation.

## Objectives

Set up a clean, maintainable, standards-compliant personal Scoop bucket that can host multiple applications in the future.

### 1. Inspect the existing repository

Before making changes:

- Inspect the current directory, files, and Git status.
- Read existing documentation and configuration.
- Preserve existing user work.
- Do not overwrite or delete files unnecessarily.
- Do not initialize another Git repository.
- Do not commit or push changes automatically.

### 2. Create the repository structure

Use the following structure where appropriate:

    scoop-bucket/
    ├── bucket/
    │   └── .gitkeep
    ├── .github/
    │   └── workflows/
    │       └── validate-manifests.yml
    ├── .gitignore
    ├── README.md
    ├── CONTRIBUTING.md
    └── LICENSE

Adapt the structure if appropriate files already exist.

The `bucket/` directory must contain Scoop manifests in JSON format, with one manifest per application.

Do not create fake application manifests, placeholder download URLs, invented hashes, or manifests for applications that do not have verified release artifacts.

### 3. Create README.md

Write a polished, clear README in English.

Explain:

- The purpose of this personal Scoop bucket.
- That it is maintained by `rendicahya`.
- That it can distribute multiple independently maintained Windows applications.
- How to add the bucket to Scoop.
- How to search for, install, and update applications.
- How to list configured buckets.
- How manifests relate to upstream application repositories and releases.
- How users can report issues or contribute.

Include these commands:

Add the bucket:

    scoop bucket add personal https://github.com/rendicahya/scoop-bucket

List configured buckets:

    scoop bucket list

Search for an application:

    scoop search <app-name>

Install an application:

    scoop install <app-name>

Update Scoop and bucket metadata:

    scoop update

Update an installed application:

    scoop update <app-name>

Explain that an application can only be installed after its valid manifest has been added to the bucket.

Do not imply that any particular application is already available unless its manifest actually exists.

### 4. Create CONTRIBUTING.md

Document a simple workflow of committing directly to `main` (no feature branches or pull requests):

1. Start from an up-to-date `main`.
2. Add or update a manifest in `bucket/`.
3. Validate the JSON and Scoop manifest conventions.
4. Verify that release URLs are accessible and SHA-256 hashes match the actual artifacts.
5. Test installation, execution, updating, and uninstallation when feasible.
6. Review the diff, then commit and push directly to `main`.
7. Ensure the validation workflow passes for the pushed commit.

Document the following manifest conventions:

- One manifest per application.
- Use lowercase manifest filenames where practical.
- Follow official Scoop manifest conventions.
- Keep version numbers synchronized with upstream releases.
- Use actual, accessible release URLs.
- Use verified SHA-256 hashes.
- Include suitable metadata such as description, homepage, license, architecture, and persistence settings when applicable.
- Avoid unnecessary installer scripts and side effects.
- Never include credentials, tokens, or private URLs.
- Document unusual installation or uninstallation behavior.

### 5. Add GitHub Actions for validation

Create `.github/workflows/validate-manifests.yml`.

Requirements:

- Run on pull requests and pushes to the default branch.
- Validate every `*.json` manifest under `bucket/`.
- Fail when a manifest contains malformed JSON.
- Where practical, use established tooling to validate Scoop-specific manifest conventions.
- Prefer official Scoop documentation and maintained validation tools.
- Pin third-party GitHub Actions to reviewed versions or full commit SHAs where appropriate.
- Apply least-privilege workflow permissions.
- Do not require secrets or personal access tokens for ordinary validation.
- Do not execute arbitrary installation scripts from manifests as part of validation.
- Report limitations honestly if full Scoop-specific validation cannot run in the selected environment.

Keep the workflow simple, reliable, and maintainable. Do not introduce an elaborate CI/CD framework.

### 6. Document local validation on Windows

Explain how to validate manifests locally on Windows.

Prefer official Scoop tooling or a maintained validation utility. Verify the current documentation before selecting commands.

If a development-only tool is needed, document how to install and run it. Do not add Python, Node.js, or another runtime dependency merely for basic JSON validation.

The repository must remain usable even when optional validation tooling is unavailable locally.

### 7. Repository and manifest policies

Establish these policies:

- This repository contains Scoop manifests and bucket documentation, not application source code.
- Each application maintains its own source repository and release process.
- Each manifest must reference a real upstream release artifact.
- SHA-256 hashes must be verified against the corresponding artifact.
- Do not duplicate manifests unnecessarily.
- Do not include application binaries directly in the bucket repository unless there is a compelling, documented reason.
- Avoid destructive installation or uninstallation behavior.
- Keep manifests straightforward and maintainable.
- Do not add placeholder manifests for hypothetical future applications.

### 8. Official documentation

Use official Scoop documentation as the primary reference:

- Scoop repository: https://github.com/ScoopInstaller/Scoop
- App Manifests: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests
- Buckets: https://github.com/ScoopInstaller/Scoop/wiki/Buckets
- Creating an App Manifest: https://github.com/ScoopInstaller/Scoop/wiki/Creating-an-app-manifest

Verify the references and follow current documented conventions.

### 9. Quality requirements

- Keep the repository minimal and easy to maintain.
- Prefer clear documentation over unnecessary abstraction.
- Do not implement any applications in this repository.
- Do not create a manifest generator unless there is a compelling reason.
- Do not create application release workflows; upstream application repositories own their releases.
- Preserve existing files and changes.
- Do not commit or push automatically.
- Do not modify GitHub repository settings.
- Never claim a check passed unless it was actually executed and its result inspected.

### 10. Execution process

Work directly in the current repository.

1. Inspect the repository and briefly summarize the findings.
2. Implement the setup.
3. Run available local validation and checks.
4. Review the final Git diff for accidental changes, incorrect instructions, and invalid configuration.
5. Summarize the files created or modified, validation results, and remaining manual steps.

Proceed with implementation rather than stopping after proposing a plan. Ask me only if an essential ambiguity cannot be resolved safely from the repository or official documentation.
