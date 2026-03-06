# Design: Dual-Branch GitOps Setup for ps-demo-all-in-one-branch

## Goal
Replicate the `ps-demo-all-in-one` setup in `ps-demo-all-in-one-branch` with a key modification:
- Values files (`envs/`) and Helm chart versioning should be managed in a separate `deployment` branch.
- PRs for environment promotion (Dev -> Stage -> Prod) should target the `deployment` branch.

## Architecture

### 1. Branching Strategy
- **`main`**: Contains application source code (`app/`), Helm chart templates (`helm/`), and the release automation workflows.
- **`deployment`**: Contains the environment-specific configurations (`envs/`). This branch tracks the current state of all environments.

### 2. File Organization
#### `main` branch:
- `.github/workflows/release.yaml`
- `app/` (Dockerfile, index.html)
- `helm/` (Chart.yaml, templates/, values.yaml)
- `package.json`, `package-lock.json`, `.releaserc.json`
- `.gitignore`, `README.md`

#### `deployment` branch:
- `envs/dev/values.yaml`
- `envs/stage/values.yaml`
- `envs/prod/values.yaml`

### 3. Automation Logic

#### Release Workflow (`main` branch)
1. Triggered on push to `main`.
2. Runs `semantic-release` for App and Helm chart.
3. Builds and pushes Docker images and Helm charts to GHCR.
4. **Environment Update**:
   - Checkouts the `deployment` branch.
   - Updates `envs/dev/values.yaml` with the new versions.
   - Commits directly to `deployment` branch.
   - Creates a PR from `deployment` to `deployment` (head: `promote-dev-to-stage`, base: `deployment`) to sync `dev` to `stage`.

#### Promotion Workflow (`deployment` branch)
1. Triggered when a PR is merged into `deployment` AND the PR modified `envs/stage/values.yaml`.
2. Syncs `envs/stage/values.yaml` to `envs/prod/values.yaml`.
3. Creates a PR from `deployment` to `deployment` (head: `promote-stage-to-prod`, base: `deployment`) to sync `stage` to `prod`.

## Implementation Steps
1. Initialize the `main` branch with the source code and automation from `ps-demo-all-in-one`.
2. Create the `deployment` branch.
3. Move `envs/` to the `deployment` branch and remove it from `main`.
4. Update GitHub Actions to handle the multi-branch logic (checkout `deployment` for updates, target `deployment` for PRs).
5. Configure repository permissions for the `deployment` branch.

## Verification
1. Push a change to `app/` in `main`.
2. Verify `envs/dev/values.yaml` is updated in the `deployment` branch.
3. Verify a PR is opened from `deployment` to `deployment` for Staging.
4. Merge the Staging PR and verify a Production PR is opened in the `deployment` branch.
