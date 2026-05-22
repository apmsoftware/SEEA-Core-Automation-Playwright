# SEEA-Core-Automation-Playwright

# Sync `conftest.py` Between Repositories Using GitHub Actions

This setup automatically syncs:

```txt
SEEA-Core-Automation-Playwright/conftest.py
```

to:

```txt
SEEA-Playwright-Consumer/tests/conftest.py
```

whenever `conftest.py` changes in the engine repository.

---

# Repository Details

## Source Repository
```txt
apmsoftware/SEEA-Core-Automation-Playwright
```

## Consumer Repository
```txt
apmsoftware/SEEA-Playwright-Consumer
```

---

# Architecture

Whenever code is pushed to:

```txt
conftest.py
```

inside:

```txt
SEEA-Core-Automation-Playwright
```

GitHub Actions will:

1. Trigger workflow
2. Clone consumer repo
3. Replace:
   ```txt
   tests/conftest.py
   ```
4. Commit changes
5. Push automatically

---

# Step 1 — Create GitHub Personal Access Token (PAT)

Open GitHub token settings:

https://github.com/settings/tokens

---

## Recommended Token Type

Use:
- Fine-grained token

---

## Repository Access

Grant access to:

```txt
SEEA-Playwright-Consumer
```

---

## Required Permissions

### Repository Permissions

| Permission | Access |
|---|---|
| Contents | Read and Write |
| Pull Requests | Read and Write (Optional) |

---

# Step 2 — Add GitHub Secret

Open:

```txt
https://github.com/apmsoftware/SEEA-Core-Automation-Playwright/settings/secrets/actions
```

---

## Create New Repository Secret

### Name
```txt
CROSS_REPO_TOKEN
```

### Value
Paste your GitHub PAT token.

---

# Step 3 — Create Workflow File

Inside repository:

```txt
SEEA-Core-Automation-Playwright
```

Create folders:

```txt
.github/workflows/
```

Create file:

```txt
sync-conftest.yml
```

---

# Step 4 — Add Workflow YAML

Paste the following YAML into:

```txt
.github/workflows/sync-conftest.yml
```

```yaml
name: Sync conftest.py to consumer repo

on:
  push:
    branches:
      - main
    paths:
      - 'conftest.py'

permissions:
  contents: write

concurrency:
  group: sync-conftest
  cancel-in-progress: true

jobs:
  sync:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout engine repo
        uses: actions/checkout@v4

      - name: Checkout consumer repo
        uses: actions/checkout@v4
        with:
          repository: apmsoftware/SEEA-Playwright-Consumer
          token: ${{ secrets.CROSS_REPO_TOKEN }}
          path: consumer

      - name: Copy conftest.py
        run: |
          cp conftest.py consumer/tests/conftest.py

      - name: Commit and push changes
        run: |
          cd consumer

          git config user.name "github-actions"
          git config user.email "github-actions@github.com"

          git add tests/conftest.py

          git diff --cached --quiet && echo "No changes detected" && exit 0

          git commit -m "Auto-sync conftest.py from engine repo"

          git push origin HEAD:main
```

---

# Step 5 — Commit Workflow

Run:

```bash
git add .github/workflows/sync-conftest.yml
git commit -m "Added conftest sync workflow"
git push
```

---

# Step 6 — Test Workflow

Modify:

```txt
conftest.py
```

Then push:

```bash
git add conftest.py
git commit -m "Updated conftest"
git push
```

---

# Expected Workflow Execution

GitHub Actions will:

1. Detect changes to:
   ```txt
   conftest.py
   ```

2. Clone:
   ```txt
   apmsoftware/SEEA-Playwright-Consumer
   ```

3. Replace:
   ```txt
   tests/conftest.py
   ```

4. Commit changes

5. Push automatically to:
   ```txt
   main branch
   ```

---

# Final Repository Structure

## Engine Repository

```txt
SEEA-Core-Automation-Playwright
 ├── conftest.py
 └── .github
      └── workflows
           └── sync-conftest.yml
```

---

## Consumer Repository

```txt
SEEA-Playwright-Consumer
 └── tests
      └── conftest.py
```

---

# Recommended Enterprise Improvement (Optional)

Instead of directly pushing to `main`, create an automated Pull Request.

Replace the final step with:

```yaml
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v7
        with:
          token: ${{ secrets.CROSS_REPO_TOKEN }}
          path: consumer
          commit-message: Sync conftest.py from engine
          title: Sync conftest.py from engine repo
          body: Automated synchronization from SEEA-Core-Automation-Playwright
          branch: auto-sync-conftest
```

Benefits:
- Safer deployments
- Review before merge
- Better audit trail
- Works with branch protection rules

---

# Troubleshooting

## Permission Denied During Push

Ensure PAT has:

```txt
Contents → Read and Write
```

---

## Workflow Not Triggering

Ensure:
- Branch is `main`
- File path is exactly:
  ```txt
  conftest.py
  ```

---

## Push Rejected

If branch protection exists on `main`:
- Use Pull Request approach instead

---

# Recommended Best Practice

Since consumer repo already installs engine package via `pip`, long-term best approach is:

- Package shared configs inside Python package
- Import config directly
- Avoid file duplication completely

This workflow is best when:
- Consumer requires physical file override
- Playwright/pytest expects local `conftest.py`
- Legacy structure must be preserved
