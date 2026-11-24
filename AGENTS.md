# Agent guide — security-guardrails

Reusable GitHub Actions workflows for pull-request security gates: **OpenGrep**
SAST, **TruffleHog** verified secrets, and **supply-chain** scanning (Dependency
Review, Trivy CVE/license audit, CycloneDX SBOM). Keep changes **high-signal**,
**low-noise**, and **pinned to commit SHAs** for third-party actions.

## Repository standards

[garretpatten/github-standards](https://github.com/garretpatten/github-standards) is the
**canonical source of truth** for how every `garretpatten` repository must be structured
and maintained: license, security policy, contributing, community files, README
branding, CODEOWNERS, branch protection, pull requests, signed commits, Dependabot,
dependency hygiene, secret hygiene, workflow permissions, action pinning, repository
metadata, CI and quality, and exceptions.

- **Before starting any task**, read the current standards from the source of truth
  rather than from memory or a cached copy — they change over time. The standards index
  is at <https://github.com/garretpatten/github-standards#standards-files>; fetch raw
  files from
  `https://raw.githubusercontent.com/garretpatten/github-standards/master/<standard>.md`
  (via web fetch, `curl`, or `gh api`) and read every standard relevant to the task.
- **While working**, apply the standards to every change: workflows, dependency files,
  community files, repository metadata, and commit practices must conform to the
  standards as they exist at the time of the change.
- **When you detect drift** — this repository lagging behind a standard that changed —
  bring the repository back in line as part of normal work and call it out in your
  summary. If an update would be large or ambiguous, propose it instead of applying it
  silently.
- **Before finishing**, confirm the work satisfies the
  [verification](https://github.com/garretpatten/github-standards/blob/master/verification.md)
  standard and that local checks pass.
- For a full compliance review, use the `github-audit` skill, which audits a repository
  against these standards and reports numbered findings.

## Repository layout

| Path                                         | Purpose                                                     |
| -------------------------------------------- | ----------------------------------------------------------- |
| `.github/workflows/security-guardrails.yaml` | Reusable workflow consumed by other repos                   |
| `.github/workflows/test-workflow.yaml`       | Self-test on PRs (calls reusable workflow)                  |
| `.github/workflows/quality-checks.yaml`      | Calls `garretpatten/quality-checks` (see linters below)     |
| `.github/actionlint.yaml`                    | actionlint config (e.g., suppressing known false positives) |
| `docs/assets/`                               | Branding (shield mark SVG)                                  |
| `.trivyignore`                               | Example path exclusions for Trivy (consumers copy/adapt)    |

## Workflow conventions

1. **Pin actions** to full commit SHAs; update them manually — Dependabot
   does **not** open PRs here (`open-pull-requests-limit: 0` in
   `.github/dependabot.yaml`). The TruffleHog scan runs via a Docker image
   pinned by digest in the `trufflehog` job — Dependabot cannot bump digests
   inside `run:` blocks, so update `TRUFFLEHOG_DIGEST` manually per release.
2. **Reusable workflow inputs** must stay documented in **README.md**; breaking
   renames are called out in the tagged release's notes.
3. **Shell steps** in workflows: quote variables, use arrays for file lists
   (avoid unquoted `$(…)` — shellcheck SC2046).
4. **Noise controls**: OpenGrep `ERROR` only, TruffleHog `--results=verified`,
   Trivy `CRITICAL,HIGH` + `ignore-unfixed`, license audit `severity: HIGH`.
5. **Dependabot PRs** are skipped via `github.actor != 'dependabot[bot]'` on
   scan jobs.
6. **Line length**: `.yamllint` max 80 columns; use `# yamllint disable-line`
   sparingly for unpinnable long SHAs.

## Making changes

| Task                      | Edit                                                             |
| ------------------------- | ---------------------------------------------------------------- |
| Scanner behavior / inputs | `.github/workflows/security-guardrails.yaml` + README            |
| Self-test wiring          | `.github/workflows/test-workflow.yaml`                           |
| Docs / community files    | `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, …                 |
| Consumer path exclusions  | Document `.truffleignore` / `.trivyignore`; update examples here |

Do not commit unless the user asks.

## Verify before you finish

Run **all** checks that match what you changed before finalizing. CI’s
**Quality Checks** workflow (`.github/workflows/quality-checks.yaml`) delegates
to `garretpatten/quality-checks` and runs these tools on pull requests:

| CI linter    | Local equivalent (this repo)                        |
| ------------ | --------------------------------------------------- |
| actionlint   | `npm run lint:workflows` / `actionlint`             |
| eslint       | _(no JS sources — skipped locally)_                 |
| hadolint     | _(no Dockerfiles — skipped locally)_                |
| jq           | _(no `.json` scripts — skipped locally)_            |
| markdownlint | `npm run lint:md`                                   |
| prettier     | `npm run format:check`                              |
| ruff         | _(no Python — skipped locally)_                     |
| shellcheck   | Covered by **actionlint** on workflow `run:` blocks |
| taplo        | _(no TOML — skipped locally)_                       |
| yamllint     | `npm run lint:yaml`                                 |

**Default before you finish:**

```bash
npm install

npm run lint
```

Or step by step:

```bash
npx prettier --check .
npx markdownlint-cli2 "**/*.md" "#node_modules"
yamllint .github .yamllint .markdownlint.yaml
actionlint
```

Install tools locally if missing:

| Tool           | Example install           |
| -------------- | ------------------------- |
| **actionlint** | `brew install actionlint` |
| **yamllint**   | `pip install yamllint`    |

Local runs should pass before you finalize — especially **yamllint** on all
`.github/` YAML (workflows, **`ISSUE_TEMPLATE`**, **`dependabot.yaml`**, not
just `workflows/*.yaml`).

| If you edited                                                                                                                    | Run                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Any `*.md`                                                                                                                       | `npm run lint:md` and `npm run format:check` when layout/prose changed |
| Workflows, **`ISSUE_TEMPLATE`**, **`dependabot.yaml`**, **`.yamllint`**, **`.markdownlint.yaml`**, **`.github/actionlint.yaml`** | `npm run lint:yaml` and `npm run lint:workflows`                       |
| `package.json` / lockfile                                                                                                        | Full `npm run lint`                                                    |

## License

MIT — see [LICENSE](./LICENSE).
