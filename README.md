# Dependabot Configuration Demo

This repository now keeps a minimal Dependabot setup: GitHub Actions updates stay enabled, while everything under `playground/*` is fully ignored.

## Project Structure

```
dependabot-demo/
├── .github/
│   └── dependabot.yml          ← Minimal configuration for GitHub Actions + playground ignore
├── package.json                ← Root: axios 1.7.0 (has CVEs) + lodash 4.17.20 (has CVEs)
├── app/
│   └── package.json            ← Main app: axios 1.7.0 + node-fetch 2.6.1 (has CVEs)
├── playground/
│   ├── frontend/
│   │   └── package.json        ← Frontend example: axios 1.7.0 + vue 3.3.0
│   └── backend/
│       └── package.json        ← Backend example: express 4.17.1 + lodash 4.17.20
└── vendor/
    └── legacy/
        └── package.json        ← Legacy code: moment 2.29.1 (has CVEs) + request 2.88.2 (deprecated)
```

## Active Rules

### GitHub Actions
- Config: `package-ecosystem: github-actions`, `directory: /`
- Expected: monitor action versions under `.github/workflows/`

### Ignore All Playground Dependencies
- Config: `directories: ["/playground/*"]` with `ignore: [{ dependency-name: "*" }]`
- Expected: no dependencies under `playground/*` receive any update PRs, including security updates
- Use case: directories you do not want Dependabot to manage at all

## How to Use

1. Push this project to GitHub
2. Enable Dependabot version updates and security updates in the repository Settings
3. Wait for Dependabot to run, usually within a few minutes after pushing
4. Observe the PRs you receive and confirm that only GitHub Actions updates are opened, while `playground/*` stays ignored

### Verification Steps

```bash
# Initialize and push to GitHub
git init
git add .
git commit -m "chore: init dependabot behavior test demo"
gh repo create dependabot-demo --public --source=. --push
```

After pushing, check the following on GitHub:

1. **Insights → Dependency graph → Dependabot** — review Dependabot run logs
2. **Security → Dependabot alerts** — review vulnerability alerts
3. **Pull requests** — review the PRs Dependabot created and confirm they match the active rules

## Notes

- `directories` supports glob patterns, so `"/playground/*"` covers all immediate subdirectories under `playground`
- `ignore: [{ dependency-name: "*" }]` is what fully blocks both version updates and security updates for those directories
