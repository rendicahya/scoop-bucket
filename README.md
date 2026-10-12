# Personal Scoop Bucket

A personal [Scoop](https://scoop.sh) bucket maintained by [rendicahya](https://github.com/rendicahya), used to distribute Windows applications on Windows 10 and Windows 11.

The bucket can host several independently maintained applications. Each application lives in its own source repository and has its own release process. This repository holds only the Scoop manifests that tell Scoop where to download each release and how to install it, plus the documentation for maintaining them.

## Available applications

Every application in this bucket has a manifest file in the [`bucket/`](bucket/) directory, named `<app-name>.json`. If `bucket/` contains no manifests, no applications are available yet.

You can only install an application after its valid manifest has been added to this bucket. Seeing an application's source repository on GitHub doesn't mean you can install it through Scoop.

## Prerequisites

- Windows 10 or Windows 11
- Scoop installed ([installation instructions](https://scoop.sh))
- Git, which Scoop uses to clone buckets (`scoop install git`)

## Usage

Add this bucket to Scoop under the name `personal`:

```powershell
scoop bucket add personal https://github.com/rendicahya/scoop-bucket
```

List the buckets you have configured:

```powershell
scoop bucket list
```

Search for an application:

```powershell
scoop search <app-name>
```

Install an application:

```powershell
scoop install <app-name>
```

If another bucket has an application with the same name, add the bucket name to pick this one:

```powershell
scoop install personal/<app-name>
```

Update Scoop and the metadata of every bucket:

```powershell
scoop update
```

Update an installed application:

```powershell
scoop update <app-name>
```

Other useful commands:

```powershell
scoop info <app-name>        # Show details about an application
scoop cat personal/<app-name> # Print the manifest Scoop will use
scoop uninstall <app-name>   # Uninstall an application
scoop bucket rm personal     # Remove this bucket from Scoop
```

## How manifests relate to upstream releases

- Each application keeps its own source code, issue tracker, and release process in its own repository.
- Each manifest in `bucket/` points to a real release artifact published by that upstream repository. For example, it might point to a ZIP or EXE file attached to a GitHub release.
- Each manifest pins the artifact's SHA-256 hash. Scoop checks the hash before installing, so a changed or corrupted download won't install.
- A manifest's `version` matches the upstream release it points to. When an application publishes a new release, its manifest here is updated to the new version, URL, and hash.
- This repository doesn't contain application binaries or source code.

## Reporting issues

- **Problems installing, updating, or uninstalling an application through Scoop** (broken download links, hash mismatches, missing shortcuts, and so on): open an issue in this repository at <https://github.com/rendicahya/scoop-bucket/issues>.
- **Bugs or feature requests in the application itself:** report them in that application's own repository. The manifest's `homepage` field links to it.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the following:

- The branching and pull-request workflow
- Manifest conventions and repository policies
- How to validate manifests locally on Windows
- What the GitHub Actions validation checks, and what it doesn't

## Repository layout

```text
scoop-bucket/
├── bucket/                  # Scoop manifests, one <app-name>.json per application
├── bin/                     # Thin wrappers around Scoop's maintenance scripts
├── .github/workflows/       # Manifest validation on pushes and pull requests
├── Scoop-Bucket.Tests.ps1   # Entry point for Scoop's bucket test suite
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## License

The manifests and documentation in this repository are released under the [MIT License](LICENSE). Each application installed through this bucket has its own license, which its manifest's `license` field identifies.
