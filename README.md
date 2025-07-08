# 🚀 Release Process (GitLab Flow – Environment Branches)

This document describes our release and versioning strategy using GitLab Flow Variation 1, with dedicated environment branches (`main`, `test`, `uat`, `production`). It incorporates Upstream First principles, Semantic Versioning, and Docker image promotion through CI/CD.

---

## 📘 Branching Structure

| Branch        | Role                                 |
|---------------|---------------------------------------|
| `main`        | Default development & feature integration (DEV) |
| `test`        | QA testing (TEST env)                |
| `uat`         | Final validation before release      |
| `production`  | Stable code deployed to PROD         |

Feature, fix, and hotfix branches are named:

- `feature/<name>`
- `fix/<description>`
- `hotfix/<critical-issue>`

---

## 🧠 Upstream First Policy

For changes involving external dependencies:

- Submit a patch upstream first.
- Upon acceptance, update your local reference or version.
- Apply to relevant environment branches (`test`, `uat`, etc.).

Internal fixes do not require upstream contribution.

---

## 🧪 Semantic Versioning (SemVer)

We follow `MAJOR.MINOR.PATCH`:

| Segment  | Meaning                    | Example    |
|----------|----------------------------|------------|
| MAJOR    | Breaking changes            | `v2.0.0`   |
| MINOR    | New features, compatible    | `v1.3.0`   |
| PATCH    | Bugfixes and refinements    | `v1.3.1`   |

---

## 📦 Docker Image & Git Tag Strategy

- Docker images are built once on `main` and promoted through environments by retagging.
- Git tags (`v1.3.0`) define base version; injected dynamically via CI/CD.
- Version tags for DEV and TEST include:
    - Semantic version
    - Build date (`YYYYMMDD`)
    - Short commit SHA

### 🧪 Extended Version Format

| Environment | Format Example                          | Description                       |
|-------------|------------------------------------------|-----------------------------------|
| DEV         | `v1.3.0-dev-20240620-a3f1b2`             | Built from `main`                 |
| TEST        | `v1.3.0-test-20240620-c8d4e9`            | Retagged from DEV for QA          |
| UAT         | `v1.3.0-rc.1`, `v1.3.0-rc.2`             | RC version for staging            |
| PROD        | `v1.3.0`                                 | Final annotated tag               |
| HOTFIX      | `v1.3.1`, or `v1.3.0-hotfix.1-20240622-d9a0c1` | Patch or emergency fix       |

> ✅ Final tags (`vX.Y.Z`) remain semantic for compatibility; temporary images include build metadata.

---

## 🔧 CI/CD Version Automation

```yaml
before_script:
  - VERSION=$(git describe --tags --abbrev=0 || echo "v0.0.0")
  - COMMIT_SHA=$CI_COMMIT_SHORT_SHA
  - BUILD_DATE=$(date +'%Y%m%d')
  - IMAGE_TAG="${VERSION}-${CI_COMMIT_REF_NAME}-${BUILD_DATE}-${COMMIT_SHA}"
```

---

## 🏗️ Release Lifecycle

### ✅ DEV: Feature Integration

| Component      | Details                                |
|----------------|----------------------------------------|
| Branch         | `main`                                 |
| Trigger        | Merge of feature/fix branches          |
| Image Tag      | `dev-<latest-tag>-<date>-<commit-sha>` |
| Deployment     | Auto-deploy to DEV                     |
| Version Source | Latest Git tag on `production`         |

---

### 🧪 TEST: QA Validation

| Component      | Details                                   |
|----------------|-------------------------------------------|
| Branch         | `test` (fast-forward from `main`)         |
| Trigger        | QA request (e.g., start of test cycle)    |
| Image Tag      | `test-<latest-tag>-<date>-<commit-sha>`          |
| Deployment     | CI retags image from DEV → TEST           |
| Version Source | Same as DEV                               |

TEST is continuously updated from `main`, typically at the start of each test cycle.

---

### 📋 UAT: Release Stabilization

| Component      | Details                                   |
|----------------|-------------------------------------------|
| Branch         | `uat`                                     |
| Trigger        | After 2–3 sprints, when ready for release |
| Image Tag      | `vX.Y.Z-rc.<N>`                           |
| Deployment     | Retag image from TEST → UAT               |
| Bugfixes       | `fix/...` from `uat`, merged into `uat`   |

UAT is frozen for stabilization. Critical bugs discovered here are patched on `uat`.
After resolution, the fix is cherry-picked into `main`.

---

### 🚀 PROD: Official Release

| Component      | Details                                   |
|----------------|-------------------------------------------|
| Branch         | `production`                              |
| Trigger        | Merge from `uat`                          |
| Git Tag        | `vX.Y.Z` (annotated)                      |
| Image Tag      | `vX.Y.Z` / `prod`                         |
| Deployment     | CI deploys to PROD                        |

PROD uses final tag for traceability. Changes from `uat` are merged and released.

---

### 🔥 Hotfixes on PROD

| Component      | Details                                   |
|----------------|-------------------------------------------|
| Branch         | `hotfix/<desc>` from `production`         |
| Image Tag      | `vX.Y.Z+1`                                |
| Merge Back     | Into `main`, `test`, `uat` if applicable  |

Only critical fixes go here. After resolution, the fix is cherry-picked into upstream branches to maintain alignment.

---

## 🧩 UAT Bugfix Workflow & Upstream First

### Fixing Internal Bugs

- Create `fix/<desc>` from `uat`
- Merge into `uat`
- CI redeploys image: `vX.Y.Z-rc.<next>`
- After release, merge fix into `main`, `test` via cherry-pick or rebase

### Fixing External Bugs (Upstream First)

1. Submit patch to upstream repository
2. Wait for merge and release
3. Update dependency version in `uat`
4. Validate, tag, and deploy

Ensure these fixes are later applied to `main`, `test`, and `production`.

---

## 🔧 CI/CD Version Automation

```yaml
before_script:
  - VERSION=$(git describe --tags --abbrev=0 || echo "v0.0.0")
  - COMMIT_SHA=$CI_COMMIT_SHORT_SHA
  - IMAGE_TAG="${VERSION}-${CI_COMMIT_REF_NAME}-${COMMIT_SHA}"
