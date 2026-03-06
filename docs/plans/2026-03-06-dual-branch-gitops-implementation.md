# Dual-Branch GitOps Implementation Plan

> **For Gemini:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replicate the `ps-demo-all-in-one` setup with a separate `deployment` branch for environment values and chart versioning.

**Architecture:** The `main` branch holds source code and Helm templates. The `deployment` branch holds `envs/`. GitHub Actions bridge the two by updating `deployment` from `main` and handling promotions within `deployment`.

**Tech Stack:** GitHub Actions, Helm, Docker, semantic-release, Shell scripting.

---

### Task 1: Initialize Core Files in `main`

**Files:**
- Create: `package.json`
- Create: `.releaserc.json`
- Create: `.gitignore`
- Create: `README.md`

**Step 1: Create `package.json`**
```json
{
  "name": "ps-demo-all-in-one-branch",
  "version": "0.0.0",
  "private": true,
  "devDependencies": {
    "semantic-release": "^21.0.0",
    "@semantic-release/git": "^10.0.1",
    "@semantic-release/github": "^9.0.0",
    "@semantic-release/commit-analyzer": "^10.0.0",
    "@semantic-release/release-notes-generator": "^11.0.0"
  }
}
```

**Step 2: Create `.releaserc.json`**
```json
{
  "branches": ["main"],
  "plugins": [
    [
      "@semantic-release/commit-analyzer",
      {
        "preset": "angular"
      }
    ],
    "@semantic-release/release-notes-generator",
    "@semantic-release/github"
  ]
}
```

**Step 3: Create `.gitignore`**
```text
node_modules/
*.tgz
.DS_Store
.env
```

**Step 4: Create `README.md`**
Use a simplified version of the source README, highlighting the dual-branch setup.

**Step 5: Commit**
```bash
git add package.json .releaserc.json .gitignore README.md
git commit -m "chore: initial repository setup"
```

---

### Task 2: Implement Application and Helm Templates

**Files:**
- Create: `app/Dockerfile`
- Create: `app/index.html`
- Create: `helm/Chart.yaml`
- Create: `helm/values.yaml`
- Create: `helm/templates/_helpers.tpl`
- Create: `helm/templates/deployment.yaml`
- Create: `helm/templates/service.yaml`

**Step 1: Create Application files**
Copy `app/Dockerfile` and `app/index.html` from source.

**Step 2: Create Helm Chart files**
Copy all files from `helm/` directory from source.

**Step 3: Commit**
```bash
git add app/ helm/
git commit -m "feat: add application and helm chart templates"
```

---

### Task 3: Set up `deployment` branch

**Files:**
- Create: `envs/dev/values.yaml` (on `deployment` branch)
- Create: `envs/stage/values.yaml` (on `deployment` branch)
- Create: `envs/prod/values.yaml` (on `deployment` branch)

**Step 1: Create and switch to `deployment` branch**
```bash
git checkout -b deployment
```

**Step 2: Create environment values files**
Copy `envs/` from source.

**Step 3: Commit and return to `main`**
```bash
git add envs/
git commit -m "chore: initialize environment values"
git checkout main
```

---

### Task 4: Configure Multi-Branch Release Workflow

**Files:**
- Modify: `.github/workflows/release.yaml`

**Step 1: Update the update-versions step**
Modify the `deploy-dev-and-promote-stage` job to:
1. Checkout `deployment` branch.
2. Update `envs/dev/values.yaml`.
3. Commit to `deployment`.
4. Create PR on `deployment` branch.

**Step 2: Commit**
```bash
git add .github/workflows/release.yaml
git commit -m "ci: update release workflow for dual-branch setup"
```

---

### Task 5: Configure Multi-Branch Promotion Workflow

**Files:**
- Modify: `.github/workflows/promote.yaml`

**Step 1: Update triggers and targets**
Ensure the workflow triggers on `deployment` branch and opens PRs against `deployment`.

**Step 2: Commit**
```bash
git add .github/workflows/promote.yaml
git commit -m "ci: update promote workflow for deployment branch"
```

---

### Task 7: Implement Build Safety Checks

**Files:**
- Modify: `.github/workflows/release.yml`

**Step 1: Update job conditions**
Ensure `deploy-dev-and-promote-stage` only runs if builds are successful or skipped, and never if they fail.

```yaml
    if: |
      (needs.build-app.result == 'success' || needs.build-app.result == 'skipped') &&
      (needs.build-helm.result == 'success' || needs.build-helm.result == 'skipped') &&
      (needs.build-app.result == 'success' || needs.build-helm.result == 'success')
```

---

### Task 8: End-to-End Verification Scenarios

**Step 1: Scenario 1 - App Only**
- Push `feat(app)` change.
- Verify App built, version bumped in `envs/dev`, and Staging PR created with App-only change.

**Step 2: Scenario 2 - App and Helm**
- Push `feat(app)` and `feat(helm)` changes.
- Verify both built and both versions updated in PR.

**Step 3: Scenario 3 - Helm Only**
- Push `feat(helm)` change.
- Verify App build skipped, Chart built and updated in Staging PR.

---

### Task 9: Final Documentation

**Step 1: Update README.md**
Document the dual-branch architecture, branching rules, and promotion triggers.
