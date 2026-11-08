# cvecli-db

Builds and publishes prebuilt CVE databases for [cvecli](https://github.com/RomainRiv/cvecli).
Database production lives in the installable cvecli package; this repository owns
its schedule and GitHub release workflow.

## Build and release

The daily workflow installs the locked `cvecli[build]==0.2.1` release from PyPI,
then runs:

```bash
uv run --locked cvecli-build download-json --years 50 --data-dir data
uv run --locked cvecli-build extract-parquet --years 50 --data-dir data
uv run --locked cvecli-build enrich-nvd --years 50 --data-dir data
uv run --locked cvecli-build create-manifest --data-dir data --source github-actions --release-status official
```

Extraction preserves the nine normalized tables: CVEs, descriptions, metrics,
products, versions, CWEs, references, credits, and tags. The manifest contains the
schema version, checksums, sizes, statistics, and extraction coverage. License and
notice files are copied into the output directory and included in its checksums.
NVD enrichment adds `cve_cpe_configurations.parquet` and `cpe_matches.parquet`
for offline CPE lookup. It streams yearly NVD CVE feeds and the CPE Match archive
(roughly 1 GB for the latter), preserving NVD dictionary expansions and Boolean
configuration context. Their checksums and import coverage are added to the manifest.
There is no embedding generation or embedding release asset.

The workflow uploads the Parquet files, manifest, and the exact notice/license
files recorded in that manifest. The schema version remains compatible with cvecli.
Users install the data with `cvecli update`.

## Builder version

Local development and GitHub Actions use the same published cvecli 0.2.1 package
and dependency lockfile. No cvecli source checkout or `CVECLI_REF` variable is
required. A standalone clone can run:

```bash
uv sync --locked
uv run --locked cvecli-build --help
```

To upgrade the builder, change the exact version in `pyproject.toml`, then run
`uv lock --upgrade-package cvecli` and verify the build before committing both files.

To test an arbitrary local cvecli checkout without changing this repository's
dependency configuration:

```bash
uv run --no-project --with '/path/to/cvecli[build]' cvecli-build --help
```

## Manual releases

Run **Build CVE Database** through GitHub Actions with the desired year count and
release type: `official`, `prerelease`, or `draft`. Scheduled runs use 50 years and
an official release. Tags use the UTC date, with `-pre` or `-draft` suffixes where
applicable. No credentials are needed for local database production; publishing
uses the workflow's GitHub token.

## Licensing

Code is MIT licensed. Database content comes from `CVEProject/cvelistV5` and is
subject to the CVE Program Terms of Use included with every dataset.
