# Agent guide — security-guardrails

Reusable GitHub Actions workflows for pull-request security gates: **OpenGrep**
SAST, **TruffleHog** verified secrets, and **supply-chain** scanning (Dependency
Review, Trivy CVE/license audit, CycloneDX SBOM). Keep changes **high-signal**,
**low-noise**, and **pinned to commit SHAs** for third-party actions.

## Repository layout

| Path                                         | Purpose                                                       |
| -------------------------------------------- | ------------------------------------------------------------- |
| `.github/workflows/security-guardrails.yaml` | Reusable workflow consumed by other repos                     |
| `.github/workflows/test-workflow.yaml`       | Self-test on PRs (calls reusable workflow)                    |
| `.github/workflows/quality-checks.yaml`      | Calls `garretpatten/quality-checks` (actionlint, yamllint, …) |
| `docs/assets/`                               | Branding (shield mark SVG)                                    |
| `.trivyignore`                               | Example path exclusions for Trivy (consumers copy/adapt)      |

## Workflow conventions

1. **Pin actions** to full commit SHAs; let Dependabot open bump PRs.
2. **Reusable workflow inputs** must stay documented in **README.md**; breaking
   renames need a migration section.
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

Run **all** checks that apply to your edits before finalizing. **actionlint is
required** for any workflow YAML change — it catches invalid syntax, expression
errors, and embedded shell issues (including shellcheck) before CI.

```bash
npm install

npm run lint
```

Or run individually:

```bash
npx prettier --check .
npx markdownlint-cli2 "**/*.md" "#node_modules"
yamllint .github/workflows/*.yaml
actionlint
```

Install tools locally if missing:

| Tool           | Example install           |
| -------------- | ------------------------- |
| **actionlint** | `brew install actionlint` |
| **yamllint**   | `pip install yamllint`    |

CI’s **Quality Checks** workflow also runs **actionlint**, **yamllint**, and
**markdownlint** on pull requests — local runs should pass first.

| If you edited              | Run                                                            |
| -------------------------- | -------------------------------------------------------------- |
| Any `*.md`                 | `npm run lint:md` (and `format:check` if prose/layout changed) |
| `.github/workflows/*.yaml` | **`actionlint`** and **`yamllint`** (mandatory)                |
| `package.json` / lockfile  | Full `npm run lint`                                            |

## License

MIT — see [LICENSE](./LICENSE).
