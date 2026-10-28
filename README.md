<!-- markdownlint-disable MD033 MD041 -->

<p align="center">
    <img
        src="./docs/assets/security-guardrails-mark.svg"
        alt="Security Guardrails shield logo"
        width="96"
        height="96"
    />
</p>

<h1 align="center">Security Guardrails</h1>

<p align="center">
    <strong>High-signal reusable GitHub Actions security gates for pull requests.</strong>
</p>

<p align="center">
    OpenGrep SAST, verified-only TruffleHog secrets, dependency review, Trivy
    vulnerability and license audits, and CycloneDX SBOM artifacts — tuned for
    low noise and actionable findings.
</p>

<p align="center">
    <a href="./LICENSE"
        ><img
            src="https://img.shields.io/github/license/garretpatten/security-guardrails?style=flat-square"
            alt="License: MIT"
    /></a>
    <img
        src="https://img.shields.io/badge/SAST-OpenGrep-2563eb?style=flat-square&logo=securityscorecard&logoColor=white"
        alt="SAST: OpenGrep"
    />
    <img
        src="https://img.shields.io/badge/secrets-TruffleHog%20verified-7c3aed?style=flat-square"
        alt="Secrets: TruffleHog verified only"
    />
    <img
        src="https://img.shields.io/badge/supply%20chain-Trivy%20%2B%20SBOM-0ea5e9?style=flat-square&logo=dependabot&logoColor=white"
        alt="Supply chain: Trivy and CycloneDX SBOM"
    />
</p>

<p align="center">
    <a href="https://github.com/garretpatten/security-guardrails/actions/workflows/test-workflow.yaml"
        ><img
            src="https://img.shields.io/github/actions/workflow/status/garretpatten/security-guardrails/test-workflow.yaml?branch=master&label=CI&logo=github&style=flat-square"
            alt="Test workflow status"
    /></a>
    <a href="https://github.com/garretpatten/security-guardrails/actions/workflows/quality-checks.yaml"
        ><img
            src="https://img.shields.io/github/actions/workflow/status/garretpatten/security-guardrails/quality-checks.yaml?branch=master&label=quality&logo=github&style=flat-square"
            alt="Quality checks workflow status"
    /></a>
</p>

<p align="center">
    ✓ PR-scoped SAST &nbsp;
    ✓ Verified secrets only &nbsp;
    ✓ CRITICAL/HIGH CVEs &nbsp;
    ✓ Copyleft license guardrails &nbsp;
    ✓ CycloneDX SBOM artifacts
</p>

<!-- markdownlint-enable MD033 MD041 -->

---

## Overview

**Security Guardrails** is a [reusable GitHub Actions
workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
that runs on pull requests in consumer repositories. It focuses on **changed
files and dependencies**, surfaces **high-confidence findings**, and uploads
machine-readable results (SARIF / JSON / CycloneDX) for downstream tooling.

| Job                      | Tool                                                               | What it catches                                                                                |
| ------------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| **OpenGrep – SAST**      | [OpenGrep](https://opengrep.dev/)                                  | ERROR-severity security patterns (`p/security-audit`, `p/owasp-top-ten`) in PR diffs           |
| **TruffleHog – Secrets** | [TruffleHog](https://github.com/trufflesecurity/trufflehog)        | **Verified** credentials only (`--results=verified`), SARIF output (no raw secret values)      |
| **Supply Chain**         | Dependency Review + [Trivy](https://github.com/aquasecurity/trivy) | New vulnerable deps, CRITICAL/HIGH CVEs, forbidden copyleft/high-risk licenses, CycloneDX SBOM |

## Quick start

Add a workflow in your repository (e.g. `.github/workflows/security-guardrails.yaml`):

```yaml
name: 'Security Guardrails'

on: pull_request

permissions:
  contents: read

jobs:
  security-guardrails:
    uses: garretpatten/security-guardrails/.github/workflows/security-guardrails.yaml@master
    with:
      opengrep_run: true
      trufflehog_run: true
      supply_chain_run: true
```

**Pin a commit SHA** instead of `@master` for supply-chain control:

```yaml
uses: garretpatten/security-guardrails/.github/workflows/security-guardrails.yaml@<full-commit-sha>
```

## Caller contract

- **Supported event: `pull_request` only.** The diff-based scans and
  dependency review need pull-request context. Calling the workflow from any
  other event (`push`, `workflow_run`, …) makes every job **fail with an
  `::error::` annotation** instead of silently passing.
- **Required permissions: `contents: read`** — the only permission this
  workflow needs. Grant it explicitly (workflow or job level) in the caller.
  Repositories with read-only default `GITHUB_TOKEN` permissions (the GitHub
  default since 2023) work fine as long as the caller grants
  `permissions: contents: read`; no `pull-requests` or `security-events`
  scopes are required.
- **No secrets are consumed.** Do not pass `secrets: inherit`; it only widens
  the blast radius.
- **Concurrency.** The workflow defines a
  `security-guardrails-${{ github.ref }}` concurrency group with
  `cancel-in-progress`, so superseded runs of the same PR are cancelled. Do
  not reuse this group name in your own workflows.

**Troubleshooting startup failures.** A caller that does not grant
`contents: read` fails before any job starts with an annotation like:

> The nested job 'supply-chain' is requesting 'contents: read', but is only
> allowed 'contents: none'.

Fix it by adding the `permissions` block shown in the quick start.

## Workflow inputs

| Input              | Type    | Default         | Description                                                    |
| ------------------ | ------- | --------------- | -------------------------------------------------------------- |
| `opengrep_run`     | boolean | `true`          | Run OpenGrep static analysis on changed files                  |
| `trufflehog_run`   | boolean | `true`          | Run TruffleHog secret scanning on the PR diff                  |
| `supply_chain_run` | boolean | `true`          | Dependency review, Trivy vuln/license scan, SBOM               |
| `opengrep_version` | string  | `v1.22.0`       | Pinned OpenGrep release tag                                    |
| `trivy_severity`   | string  | `CRITICAL,HIGH` | Minimum Trivy vulnerability severities                         |
| `trivy_skip_dirs`  | string  | _(empty)_       | Comma-separated directories excluded from Trivy scans (opt-in) |

Dependabot PRs are skipped automatically.

## Scanners

### OpenGrep (SAST)

[OpenGrep](https://github.com/opengrep/opengrep) is an open-source fork of
Semgrep CE with compatible rules and SARIF/JSON output. The workflow:

- Downloads a **pinned release binary** and verifies its **cosign signature**
  (identity + issuer pinned) before use
- Scans only **files changed in the PR**
- Uses rulesets `p/security-audit` and `p/owasp-top-ten`
- Fails only on **`ERROR` severity** findings to reduce noise
- Uploads `opengrep-results.json` as a workflow artifact

### TruffleHog (secrets)

- Scans the commit range from the pull request **base SHA** to `HEAD` with a
  digest-pinned TruffleHog image
- **`--results=verified`** — only secrets confirmed live by provider APIs fail
  the job; rotate exposed credentials immediately when this gate fails
- **SARIF output** uploaded as the `trufflehog-results-sarif` artifact — SARIF
  carries finding metadata (detector, file, line) but **no raw secret
  values**, so results can be archived safely
- Respects **`.truffleignore`** in the consumer repository; falls back to this
  repository's empty default if none is provided

**Known blind spot.** `--results=verified` intentionally drops findings that
are not confirmed live: `unverified` (detected but not verified — e.g.
credentials for internal systems whose verification endpoints TruffleHog
cannot reach) and `unknown` (verification errored). A credential for an
internally hosted service may therefore never be reported by this gate;
organizations with internal-secret concerns should supplement it with their
own TruffleHog run using `--results=verified,unknown`.

### Supply chain

1. **GitHub Dependency Review** — blocks PRs that introduce dependencies with
   **high-severity** advisories or **forbidden licenses**. The deny list lives
   in [`.github/dependency-review-config.yml`](./.github/dependency-review-config.yml)
   and covers strong/weak copyleft, source-available, and other high-risk
   licenses (GPL, AGPL, LGPL, MPL, CDDL, EPL, EUPL, CECILL, SSPL, BUSL, Elastic,
   and related identifiers). If the consumer repository does not define this
   file, the workflow falls back to the default in this repository.

   > **Deprecation note.** The `deny-licenses` key this policy relies on is
   > marked deprecated upstream
   > ([actions/dependency-review-action#938](https://github.com/actions/dependency-review-action/issues/938))
   > and may be removed in its next major release. The planned migration is
   > to an `allow-licenses` allow-list policy; it will ship as a tagged
   > release with a migration note in its release notes, coordinated with
   > upstream's timeline.

2. **Trivy filesystem scan** — CRITICAL/HIGH CVEs in lockfiles/manifests;
   **`ignore-unfixed: true`** avoids failing on issues with no patch.
3. **Trivy license audit** — flags **HIGH-severity (forbidden/copyleft)**
   licenses with full license text matching.
4. **CycloneDX SBOM** — `sbom.cyclonedx.json` uploaded per PR for audit and
   compliance pipelines.

**Full-tree scoping (deliberate).** Unlike the SAST and secrets gates, which
only look at the pull-request diff, the Trivy steps scan the **entire
repository tree**. A pre-existing CRITICAL/HIGH CVE or forbidden license
therefore fails every PR until it is fixed or explicitly waived — the gate
also enforces the state of the existing tree, not just new changes. Waive an
accepted finding by adding its **finding ID** (e.g. `CVE-2026-12345`) to
`.trivyignore` with a comment explaining why.

**`.trivyignore` holds finding IDs, not paths.** A plain `.trivyignore` lists
vulnerability/check IDs to waive. To exclude paths from Trivy scans, use the
`trivy_skip_dirs` workflow input or a Trivy config file in the consumer
repository. Without a consumer `.trivyignore`, the workflow falls back to this
repository's default.

## Philosophy: high signal, low noise

| Choice                         | Rationale                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| ERROR-only OpenGrep            | Warnings often reflect style or lower-confidence patterns                                  |
| Verified TruffleHog only       | Unverified entropy matches create alert fatigue                                            |
| CRITICAL/HIGH + ignore-unfixed | Focus on exploitable, patchable CVEs                                                       |
| Forbidden license list         | Surfaces copyleft / high-risk licenses in new deps                                         |
| PR-scoped SAST                 | Faster feedback; aligns with code under review                                             |
| Fail-loud preflights           | Unmet preconditions fail visibly — a gate that silently does not run is worse than no gate |
| Full-tree Trivy gates          | The existing tree is enforced too; waivers are explicit finding IDs in `.trivyignore`      |
| SARIF-only secret artifacts    | Result artifacts never contain raw secret values                                           |

## Versioning and upgrades

Releases are tagged (`v1`, `v2`, …) with short notes describing scanner
behavior changes. Pin the reusable workflow by **full commit SHA** (or tag)
and subscribe to
[releases](https://github.com/garretpatten/security-guardrails/releases) to
learn about updates — each release's notes call out breaking input or
artifact changes. This repository's action pins are kept fresh by Dependabot,
and the TruffleHog scan image is pinned by digest and updated deliberately
with each release.

## Community

| Resource                                | Use                                           |
| --------------------------------------- | --------------------------------------------- |
| [Code of Conduct](./CODE_OF_CONDUCT.md) | Expected behavior in issues and PRs           |
| [Contributing](./CONTRIBUTING.md)       | Branching, local checks, workflow conventions |
| [Security policy](./SECURITY.md)        | Vulnerability reporting (not public issues)   |

## Maintainers

[@garretpatten](https://github.com/garretpatten/)

Use the [issue templates](./.github/ISSUE_TEMPLATE/) for bugs and enhancements.

## License

This project is licensed under the [MIT License](./LICENSE).
