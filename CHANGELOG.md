# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- JavaScript dependency analysis script (`scan_js_deps.py`) to identify repositories with JavaScript dependencies and check for known vulnerabilities using `npm audit`.
- Enhanced OSS scanner to better handle JavaScript/Node.js projects and their dependency trees.
- Added support for analyzing repositories with `package.json` files to identify outdated or vulnerable dependencies.
- New reporting format for JavaScript vulnerabilities, including severity levels and recommended fixes.
- Integration with npm audit for comprehensive vulnerability scanning of JavaScript dependencies.

### Fixed
- Resolved issues with OSS report generation for repositories with complex dependency structures.
- Improved error handling for npm audit command execution.
- Fixed path resolution for JavaScript projects in nested directories.

### Changed
- Updated documentation to include JavaScript dependency scanning capabilities.
- Improved logging for JavaScript dependency analysis to provide more detailed output.
- Optimized the scanning process for better performance with large JavaScript codebases.
- `README.md` with setup and usage instructions.
- `requirements.txt` for Python dependencies.
- `.gitignore` to exclude sensitive files and development artifacts.
- `--repo` CLI argument to scan a single repository (e.g., `--repo owner/repo` or `--repo repo` defaulting to `--org`). If omitted, all repositories in `--org` are scanned.
- Optional Dagda integration for container image analysis:
  - Flags: `--dagda-url`, `--docker-image`
  - Outputs: `{repo}_dagda.json`, `{repo}_dagda.md`
- Optional Syft integration for SBOM generation:
  - Runs on cloned repo directory; if `--docker-image` is provided, also runs on the image
  - Flag: `--syft-format` (default: `cyclonedx-json`)
  - Outputs: `{repo}_syft_repo.json`/`.md` and (if image) `{repo}_syft_image.json`/`.md`

- OSS scanner: integrated Semgrep Struts2 rules (`semgrep-rules/java-struts2.yaml`, `java-struts2-heuristics.yaml`) to detect RCE patterns in Java code.
- OSS scanner: added naive POM-based detector for known Struts2 CVEs (e.g., CVE-2017-5638), including property resolution from the same pom `<properties>`.
- OSS scanner: unified JSON parsing across `pip-audit`, `osv-scanner`, `npm audit`, and `semgrep` into a consolidated vulnerability table in per-repo markdown reports.
- OSS scanner: helper to optionally generate `package-lock.json` in temp clones (`npm install --ignore-scripts --package-lock-only`) when only `package.json` exists to improve OSV coverage.
- OSS scanner: optional flag `--parse-osv-cvss` to compute CVSS base scores from OSV severity vectors (uses `cvss`/`cvsslib` if available).
- Dependencies: added `cvss` to `requirements.txt` for CVSS vector parsing.

- CodeQL scanner: revamped orchestration modeled after `scan_oss.py`.
  - Retries on GitHub API, org→user fallback, and improved logging.
  - Concurrency with `--max-workers` for repo-level parallelism.
  - Enhanced CLI: `--fail-fast`, `--fail-on-severity`, `--sarif-only`, `--json-only`, `--top-n`, `--timeout-seconds`, `--skip-autobuild`, `--build-command`.
  - Language detection improvements (Java/Kotlin, JS/TS, Python, Go, C/C++, C#, Ruby, Swift).
  - CodeQL DB creation supports custom build or autobuild; step timeouts supported.
  - SARIF parsing enriched with rule tags, precision, CWE extraction; severity normalized from security-severity.
  - Rule-specific mitigation text extracted from CodeQL rule metadata (`help`, `fullDescription`) and surfaced in Markdown reports; `helpUri` captured as `rule_doc_url` in JSON.
  - Deduplication and ranking (CVSS then severity rank).
  - Per-repo JSON/Markdown outputs and org-level `codeql_summary.md`.
 - Orchestrator script `orchestrate_scans.py` to run all scanners with profiles (fast/balanced/deep) and generate `markdown/orchestration_summary.md`.
- README: new Orchestrator section with usage examples and summary/output locations.
- Docker: comprehensive toolchain in image (Semgrep, Gitleaks, Trivy, Syft, Grype, OSV-Scanner, CodeQL CLI, govulncheck, bundler-audit, Dependency-Check) with per-scanner report volumes.
- Docker Compose: macOS-friendly defaults (platform linux/amd64), report/cache volumes, and balanced-profile entrypoint via orchestrator.
- Orchestrator: preflight `logs/versions.log` capturing installed tool versions for diagnostics.
- Compose: fixed invalid `.git-credentials` file mount; all mounts now target directories only. Added guidance to ensure host bind paths exist.
- Docs: added `Docker.md` with comprehensive Docker/Compose usage, single/multi-scanner runs, arguments, and troubleshooting.

### Changed
- OSS scanner: corrected `pip-audit` usage to `-r <requirements*.txt> -f json`, tolerate non-zero exits when vulns are found, and fallback to `python -m pip_audit` when the CLI is not in PATH.
- OSS scanner: for Node/Python, OSV scanning now targets lockfiles only; for Java manifests (`pom.xml`, Gradle), OSV scans the repository recursively (`osv-scanner -r`) for better resolution.
- OSS scanner: when only `package.json` is present, fallback to `npm audit --json` with robust parsing and aggregation.
- OSS scanner: improved error handling, logging, and multi-JSON chunk parsing across all tool integrations.
- OSS scanner: deduplication now preserves `cvss_score` from any source when the current record lacks it, improving severity ranking consistency.

- CodeQL scanner: reporting now shows top findings sorted by CVSS/severity and includes SARIF artifact paths (unless `--json-only`).
  - Added `Mitigation` column to per-repo Markdown Top Findings tables.

- Dockerfile: Made `bundler-audit` installation resilient to corporate SSL interception.
  - Option B: automatic HTTP RubyGems fallback if HTTPS install fails (last resort, insecure).
  - Option A (alternative): support `--build-arg CORP_CA_B64=<base64 PEM>` to trust a corporate root CA during build.

### Planned
- Add structured, parseable output (e.g., SARIF/JSON) and a consolidated summary report.
- Add CI workflow and containerized execution (Docker) with pinned tool versions.
- Add `--dry-run` mode and additional dependency discovery (Pipenv, setup.cfg/py, requirements.in).
- Add a minimal fixtures test suite covering JSON parsing normalization and the Struts2 POM detection helper (including property resolution).

### Fixed
- Resolved Semgrep rules path to use absolute `semgrep-rules/` directory relative to the project, avoiding CWD issues.
- Prevented OSV extractor errors by avoiding direct scans of `package.json`; scanning is restricted to lockfiles, with `npm audit` as fallback.
- Eliminated false "pip-audit not installed" failures by correcting flags and adding a module fallback when PATH resolution fails.

## [0.3.1] - 2025-09-12

### Added
- Hardcoded IPs/Hostnames report now includes proof fields:
  - JSON: adds `key` and `value` per finding.
  - Markdown: new columns Key and Value in Detailed Findings.
- Logging controls added to `scan_hardcoded_ips.py` (mirrors gitleaks script):
  - `-v/--verbose` (repeatable), `-q/--quiet`, `--loglevel {DEBUG,INFO,WARNING,ERROR,CRITICAL}`.

### Changed
- `scan_gitleaks_fixed.py` and `scan_hardcoded_ips.py` now fall back from org to user on 404 for repo listing.
- `install_dependencies.sh` expanded with optional flags and post-install sanity check (documented in 0.3.0), minor refinements.

### Fixed
- Replaced deprecated `datetime.utcnow()` with `datetime.now(timezone.utc)` in `scan_hardcoded_ips.py`.
- Summary aggregation for hardcoded IPs: robust regex parsing prevents `ValueError` when reading markdown counts.
- Created missing Semgrep rules file `semgrep-rules/hardcoded-ips-hostnames.yaml` used by hardcoded IP scanner.

### Notes
- Consider adding a timeout to the Semgrep subprocess and SIGINT handling if long scans are interrupted.

## [0.3.0] - 2025-09-11

### Added
- Bandit and Trivy FS scanners integrated:
  - Bandit: writes `<repo>_bandit.json` and `<repo>_bandit.md` with summaries and sample findings.
  - Trivy (fs): writes `<repo>_trivy_fs.json` and `<repo>_trivy_fs.md` with severity summaries.
- Semgrep taint-mode optional pass (`--semgrep-taint`) and exploitable flows section.
- Gitleaks integration with secrets findings section.
- Policy gate and `policy.yaml` link surfaced in summary.
- Threat intel enrichment (KEV/EPSS) for Grype and Top 5 ranking:
  - Added KEV/EPSS badges and an explicit `Exploitability` column to "Top 5 Vulnerabilities".
  - New "Threat Intel" counts block and a "Threat Intel Diagnostics" subsection.
- VEX support passthrough for Grype (`--vex <file>`) and sample VEX file `struts2_exploitable.vex.json` (CVE-2017-5638 set to exploitable for Struts2).
- Summary now lists Bandit and Trivy report links in "Detailed Reports" when present.
- `install_dependencies.sh` expanded to install all scanners on macOS/Linux/Windows.
  - Optional flags: `--no-java`, `--no-go`, `--sanity-only`.
  - Post-install sanity check prints versions for all tools.

### Changed
- Summary table status for Bandit/Trivy is derived from JSON results rather than exit codes (avoids "success" when findings exist).
- Top 5 now considers Trivy FS vulnerabilities in addition to Grype to avoid empty results when VEX suppresses Grype findings.
- Improved repository information with contributors, languages, activity, and enhanced Top 5 ranking (KEV > EPSS > severity).

### Fixed
- Resolved `IndentationError` in `run_pip_audit_scan()` and removed placeholder lines.
- Hardened summary generation with additional try/except blocks around sections to ensure the summary always writes.

### Notes
- For Grype + VEX, best results occur when scanning an SBOM with bom-refs matching the VEX `affects` entries.

## [0.2.0] - 2025-09-10

### Added
- CLI options via `argparse` for org, API base, report directory, inclusion flags, worker count, and verbosity (`-v`, `-vv`).
- Logging with levels and timestamps; scanner outputs are captured and written to files.
- Parallel processing using `ThreadPoolExecutor` with configurable `--max-workers`.

### Changed
- GitHub authentication now uses `Authorization: Bearer <token>` and validates token presence at startup.
- HTTP calls use a `requests.Session` with retries/backoff and timeouts.
- Repository cloning is shallow (`--depth=1 --filter=blob:none`) and prefers `ssh_url` when available.
- Repositories that are forked or archived are skipped by default (override with `--include-forks` and `--include-archived`).
- Dependency discovery expanded to PEP 621 (`project.dependencies`) and Poetry (`tool.poetry.dependencies`).
- Temporary requirement files created from `pyproject.toml` are cleaned up after scanning.
- `safety` and `pip-audit` invocations now capture stdout/stderr and return codes; results saved to per-repo files.

## [0.1.0] - 2025-09-10

### Added
- Initial Python script `scan_repos.py` that:
  - Fetches repositories from a GitHub organization via the REST v3 API with pagination.
  - Clones repositories into a temporary working directory.
  - Detects Python dependencies via `requirements.txt` or PEP 621 `pyproject.toml` (`project.dependencies`).
  - Runs `safety` and `pip-audit` against discovered dependencies.
  - Writes per-repo vulnerability reports into `vulnerability_reports/`.
  - Cleans up temporary clone directories on completion.

### Known Issues
- `GITHUB_TOKEN` may be `None`, resulting in an invalid `Authorization` header. Token should be required and validated at startup.
- `clone_url` often requires credentials for private repos; support for `ssh_url` or embedded token over HTTPS is needed.
- `requests.get` calls lack timeouts and retry/backoff; risk of hangs and rate-limit failures.
- `safety` flags may not be correct for current versions (`--save-html` with `--output text` mismatch). Pin `safety` version and adjust CLI accordingly.
- Only PEP 621 dependencies are parsed from `pyproject.toml`; Poetry (`tool.poetry.dependencies`) and other ecosystems are not handled.
- Temporary requirements file created from `pyproject.toml` is not explicitly cleaned per repo.
- No aggregated summary across repos; reports are individual text/markdown files.
- Limited error handling: subprocess failures for scanners are ignored (`check=False`) without surfacing status.

### Security Notes
- Avoid echoing tokens. Consider using `Bearer` scheme for the header and validate presence.
- Prefer least-privilege tokens; document required scopes.

[Unreleased]: https://example.com/compare/v0.3.1...HEAD
[0.3.1]: https://example.com/compare/v0.3.0...v0.3.1
[0.3.0]: https://example.com/compare/v0.2.0...v0.3.0
[0.2.0]: https://example.com/compare/v0.1.0...v0.2.0
[0.1.0]: https://example.com/releases/tag/v0.1.0
