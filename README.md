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
machine-readable results (JSON / CycloneDX) for downstream tooling.

| Job                      | Tool                                                               | What it catches                                                                      |
| ------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| **OpenGrep – SAST**      | [OpenGrep](https://opengrep.dev/)                                  | ERROR-severity security patterns (`p/security-audit`, `p/owasp-top-ten`) in PR diffs |
| **TruffleHog – Secrets** | [TruffleHog](https://github.com/trufflesecurity/trufflehog)        | **Verified** credentials only (`--results=verified`), JSON output                    |
| **Supply Chain**         | Dependency Review + [Trivy](https://github.com/aquasecurity/trivy) | New vulnerable deps, CRITICAL/HIGH CVEs, forbidden copyleft licenses, CycloneDX SBOM |

## Quick start

Add a workflow in your repository (e.g. `.github/workflows/security-guardrails.yaml`):

```yaml
name: 'Security Guardrails'

on: pull_request

jobs:
  security-guardrails:
    uses: garretpatten/security-guardrails/.github/workflows/security-guardrails.yaml@master
    with:
      opengrep_run: true
      trufflehog_run: true
      supply_chain_run: true
    secrets: inherit
```

**Pin a commit SHA** instead of `@master` for supply-chain control:

```yaml
uses: garretpatten/security-guardrails/.github/workflows/security-guardrails.yaml@<full-commit-sha>
```

## Workflow inputs

| Input              | Type    | Default         | Description                                      |
| ------------------ | ------- | --------------- | ------------------------------------------------ |
| `opengrep_run`     | boolean | `true`          | Run OpenGrep static analysis on changed files    |
| `trufflehog_run`   | boolean | `true`          | Run TruffleHog secret scanning on the PR diff    |
| `supply_chain_run` | boolean | `true`          | Dependency review, Trivy vuln/license scan, SBOM |
| `opengrep_version` | string  | `v1.22.0`       | Pinned OpenGrep release tag                      |
| `trivy_severity`   | string  | `CRITICAL,HIGH` | Minimum Trivy vulnerability severities           |

Dependabot PRs are skipped automatically.

## Scanners

### OpenGrep (SAST)

[OpenGrep](https://github.com/opengrep/opengrep) is an open-source fork of
Semgrep CE with compatible rules and SARIF/JSON output. The workflow:

- Installs a **pinned release** via the official install script
- Scans only **files changed in the PR**
- Uses rulesets `p/security-audit` and `p/owasp-top-ten`
- Fails only on **`ERROR` severity** findings to reduce noise
- Uploads `opengrep-results.json` as a workflow artifact

### TruffleHog (secrets)

- Scans the PR diff (`base` → `HEAD`)
- **`--results=verified`** — only secrets confirmed live by provider APIs
- **`--json`** — structured output uploaded as an artifact
- Respects **`.truffleignore`** in the consumer repository

### Supply chain

1. **GitHub Dependency Review** — blocks PRs that introduce dependencies with
   **high-severity** advisories or **forbidden licenses** (GPL, AGPL, SSPL,
   BUSL, and related identifiers).
2. **Trivy filesystem scan** — CRITICAL/HIGH CVEs in lockfiles/manifests;
   **`ignore-unfixed: true`** avoids failing on issues with no patch.
3. **Trivy license audit** — flags **HIGH-severity (forbidden/copyleft)**
   licenses with full license text matching.
4. **CycloneDX SBOM** — `sbom.cyclonedx.json` uploaded per PR for audit and
   compliance pipelines.

Add a **`.trivyignore`** in your repo to exclude paths (see this repo’s
example).

## Migration

### Renamed from `security-checks`

This project was renamed **Security Guardrails** (`security-guardrails`). Update
consumer workflows:

```yaml
# Before
uses: garretpatten/security-checks/.github/workflows/security-checks.yaml@master

# After
uses: garretpatten/security-guardrails/.github/workflows/security-guardrails.yaml@master
```

### From Semgrep

The **`semgrep_run`** input was replaced by **`opengrep_run`**. OpenGrep uses
the same registry rulesets and is a drop-in engine swap for Semgrep CE scans.

```yaml
# Before
with:
  semgrep_run: true

# After
with:
  opengrep_run: true
  supply_chain_run: true   # disable if you only want SAST + secrets
```

## Philosophy: high signal, low noise

| Choice                         | Rationale                                                 |
| ------------------------------ | --------------------------------------------------------- |
| ERROR-only OpenGrep            | Warnings often reflect style or lower-confidence patterns |
| Verified TruffleHog only       | Unverified entropy matches create alert fatigue           |
| CRITICAL/HIGH + ignore-unfixed | Focus on exploitable, patchable CVEs                      |
| Forbidden license list         | Surfaces copyleft / business-risk licenses in new deps    |
| PR-scoped SAST                 | Faster feedback; aligns with code under review            |

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
