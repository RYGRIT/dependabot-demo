# Dependabot Behavior Test Demo

This repository tests the real behavior of various Dependabot configuration options, with a focus on the differences between **version updates** and **security updates**.

## Project Structure

```
dependabot-demo/
├── .github/
│   └── dependabot.yml          ← Configuration covering 5 test scenarios
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

## Key Findings

### `exclude-paths` vs `ignore`

| Option | Version Updates | Security Updates | Match Type |
|--------|:-:|:-:|--------|
| `exclude-paths` | ✅ Works | ❌ **Does not work** | File path (glob) |
| `ignore` | ✅ Works | ✅ Works | Dependency name + version type |
| `directories` | ✅ Allowlist | ❌ Does not affect security updates | Directory path |

### Main Conclusions

1. **`exclude-paths` cannot block security-fix PRs** — this is by design, not a bug
2. **`ignore` is the only config that can block security-fix PRs** — but it matches by dependency name, not by path
3. **Best practice**: avoid using old dependencies with known vulnerabilities in playground/test directories

## Test Scenarios

### Scenario 1: GitHub Actions Baseline
- Config: `package-ecosystem: github-actions`, `directory: /`
- Expected: monitor action versions under `.github/workflows/`

### Scenario 2: Exclude `playground` with `exclude-paths`
- Config: `exclude-paths: ["playground/**", "vendor/**"]`
- Expected for Version Updates: `playground/` and `vendor/` do not receive version-update PRs ✅
- Expected for Security Updates: vulnerable `axios` in `playground/` still receives a security-fix PR ❌
- **This is exactly the issue vscode-npmx ran into.**

### Scenario 3: `directories` Allowlist Mode (commented out, uncomment to test)
- Config: `directories: ["/app"]`
- Expected: only monitor version updates in the `app/` directory
- Advantage: explicitly declares which directories to monitor, which is clearer than exclusion

### Scenario 4: Ignore by Dependency Name
- Config: ignore all updates for `moment`, and ignore major updates for `request`
- Expected: `moment` receives no PRs at all, including security fixes; `request` only receives minor/patch PRs
- **This is the correct way to block security-update PRs**

### Scenario 5: Ignore All Dependencies (commented out, uncomment to test)
- Config: `ignore: [{ dependency-name: "*" }]`
- Expected: no dependencies in that directory receive any update PRs
- Use case: directories you do not want Dependabot to manage at all

## How to Use

1. Push this project to GitHub
2. Enable Dependabot version updates and security updates in the repository Settings
3. Wait for Dependabot to run, usually within a few minutes after pushing
4. Observe the PRs you receive and compare them with the expected behavior

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
3. **Pull requests** — review the PRs Dependabot created and note the scenario labels

## Recommended Fix for `vscode-npmx`

`vscode-npmx` uses `axios: "1.7.0"` in `playground/yarn/package.json`, which has a security vulnerability, and `exclude-paths: playground/**` cannot block security-fix PRs.

Recommended fixes, in priority order:

### Option A: Update the playground dependency version (recommended)
```bash
# Use the latest version directly in playground to avoid security updates
cd playground/yarn && npm install axios@latest
```

### Option B: Add a dedicated `ignore` rule for the playground directory
```yaml
# Add this to dependabot.yml
- package-ecosystem: npm
  directory: /playground/yarn
  schedule:
    interval: weekly
  ignore:
    - dependency-name: "*"
```

### Option C: Manually dismiss the alert in Security Alerts
Find the corresponding alert under **Security → Dependabot alerts** in the GitHub repository and dismiss it manually.

## References

- [Dependabot configuration options](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file)
- [exclude-paths documentation](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#exclude-paths) — note that it is marked only for "Version updates"
- [ignore documentation](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#ignore) — note that it is marked for both "Version updates" and "Security updates"
