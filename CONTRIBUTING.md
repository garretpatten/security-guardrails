# Contributing

Participants are expected to follow the [Code of Conduct](./CODE_OF_CONDUCT.md).

## Issues

Security vulnerabilities are **not** tracked in public issues until addressed; see **[SECURITY.md](./SECURITY.md)**.

Use [GitHub Issues](https://github.com/garretpatten/security-checks/issues) with the **Bug report** or **Feature request** form. Include the consumer workflow snippet, failing job logs (redacted), and the pinned workflow ref you use.

## Pull requests

- Branch from **`master`**, focused scope per PR.
- Pin third-party actions to full commit SHAs; let Dependabot propose bumps.
- Keep scanners **high-signal**: prefer verified secrets, ERROR-severity SAST, and CRITICAL/HIGH dependency findings.
- Document breaking workflow input changes in the README migration section.

### Checks (from repo root)

```bash
npm install

npx prettier --check .
npx markdownlint-cli2 "**/*.md" "#node_modules"
yamllint .github/workflows/*.yaml
```

Documentation-only changes still need **`prettier`** and **`markdownlint`** on touched Markdown files.
