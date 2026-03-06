# GitOps All-In-One Dual-Branch Demo (ps-demo-all-in-one-branch)

This repository demonstrates a dual-branch GitOps workflow.

- **main**: Source code and Helm templates.
- **deployment**: Environment-specific configurations (`envs/`).

Automated releases on `main` update `envs/dev/values.yaml` in the `deployment` branch.
Environment promotions (Dev -> Stage -> Prod) occur via Pull Requests on the `deployment` branch.
