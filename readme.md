# GitLab to GitHub Migration Pipeline

## Overview

This workflow automates GitLab to GitHub repository migrations using GitHub Enterprise Importer (GEI) and the `gh-gl2gh` extension.

The pipeline provides:

- Environment validation
- Migration readiness assessment
- Manual approval gate
- Parallel repository migration
- Post-migration validation
- Migration artifacts and logs

Supported migration targets:

- GitHub Enterprise Cloud (GHEC)
- GitHub Enterprise Cloud Data Residency (DR)

Supported storage types:

- GitHub Storage
- Azure Storage
- AWS S3

---

# Workflow Stages

## Stage 1 - Environment Validation

Validates:

- Required GitHub Environment secrets
- GitLab source configuration
- Target GitHub API URL
- Inventory CSV file
- Required migration scripts
- Storage configuration

### Required Secrets

| Secret | Required |
|----------|----------|
| GITLAB_PAT | Yes |
| GH_PAT | Yes |

### Required Variables

| Variable | Required |
|----------|----------|
| SOURCE_GL_SERVER_URL | Yes |
| STORAGE_TYPE | No (Defaults to GITHUB) |

### Storage Validation

#### GitHub Storage

For GitHub Enterprise Cloud:

```text
TARGET_GITHUB_API_URL=https://api.github.com
```

No upload URL is required.

For GitHub Data Residency:

```text
TARGET_GITHUB_API_URL=https://api.<SUBDOMAIN>.ghe.com
```

Required:

```text
TARGET_UPLOAD_URL=https://uploads.<SUBDOMAIN>.ghe.com
```

---

#### Azure Storage

Required secret:

```text
AZURE_STORAGE_CONNECTION_STRING
```

---

#### AWS Storage

Required:

```text
AWS_BUCKET_NAME
AWS_REGION
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Optional:

```text
AWS_SESSION_TOKEN
```

`AWS_SESSION_TOKEN` is only required when using temporary AWS credentials (STS / AssumeRole).

---

## Stage 2 - Migration Readiness Check

Performs migration readiness assessment against all repositories listed in the inventory CSV.

Activities:

- Validates repository accessibility
- Validates GitLab API access
- Reviews migration prerequisites
- Produces readiness reports

Generated artifacts:

```text
1_migration_readiness_check.out
logs/
output_files/
```

---

## Stage 3 - Approval Stage

Manual approval checkpoint.

Approvers review readiness results before migration begins.

Artifact available for review:

```text
1_migration_readiness_check.out
```

---

## Stage 4 - Repository Migration

Performs parallel repository migrations using:

```bash
gh gl2gh migrate-repo
```

Features:

- Parallel execution
- Real-time migration status
- Live log streaming
- CSV-based migration tracking
- Migration artifact generation

Generated artifacts:

```text
2_migration.out
output_files/
logs/
```

Generated CSV files:

```text
gl2gh_migration_output-<timestamp>.csv
repos_with_status.csv
```

---

## Stage 5 - Post Migration Validation

Validates migrated GitHub repositories.

Activities:

- Confirms migration completion
- Reviews migration results
- Verifies repositories created successfully
- Generates validation reports

Generated artifacts:

```text
3_post_migration_validation.out
output_files/
logs/
```

---

# Workflow Inputs

| Input | Description |
|---------|-------------|
| ENVIRONMENT_NAME | GitHub Environment containing migration secrets |
| APPROVAL_ENVIRONMENT_NAME | GitHub Environment used for manual approval |
| TARGET_GITHUB_API_URL | Target GitHub API URL |
| INVENTORY_FILE | Inventory CSV generated using `gh gl2gh inventory-report` |
| RUNNER_LABEL | GitHub runner label |

---

# Target GitHub API URL Examples

## GitHub Enterprise Cloud

```text
https://api.github.com
```

## GitHub Enterprise Cloud Data Residency

```text
https://api.<SUBDOMAIN>.ghe.com
```

Example:

```text
https://api.cmf-factory.ghe.com
```

---

# Inventory CSV Requirements

The following columns must exist in the inventory file:

```text
group-path
project
url
github_org
github_repo
gh_repo_visibility
```

Example:

```csv
group-path,project,url,github_org,github_repo,gh_repo_visibility
mygroup,myrepo,https://gitlab.example.com/mygroup/myrepo,target-org,target-repo,private
```

---

# Required Scripts

The workflow expects the following scripts to exist in the repository root:

```text
1_migration_readiness_check.sh
2_migration.sh
3_post_migration_validation.sh
```

---

# Migration Engine

Repository migrations are performed using GitHub Enterprise Importer (GEI):

```bash
gh gl2gh migrate-repo
```

The workflow automates:

- Validation
- Authentication
- Storage configuration
- Migration execution
- Status tracking
- Artifact collection
- Post-migration validation

---

# Generated Artifacts

## Readiness Check

```text
migration-readiness-check-artifacts
```

Contents:

```text
1_migration_readiness_check.out
logs/
output_files/
```

---

## Migration

```text
migration-artifacts
```

Contents:

```text
2_migration.out
logs/
output_files/
```

---

## Validation

```text
post-migration-validation-artifacts
```

Contents:

```text
3_post_migration_validation.out
logs/
output_files/
```

---

# Migration Status Tracking

The migration stage generates:

```text
repos_with_status.csv
```

Example:

```csv
gitlab_group,gitlab_project,github_org,github_repo,gh_repo_visibility,MigrationStatus
team-a,repo1,target-org,repo1,private,Success
team-a,repo2,target-org,repo2,private,Failed
```

This file is used by the post-migration validation stage.

---

# Notes

- Supports GitHub Enterprise Cloud and Data Residency targets.
- Supports GitHub, Azure, and AWS migration storage backends.
- Migrations run in parallel for improved throughput.
- Manual approval is required before repository migration begins.
- Validation artifacts are retained for 7 days.
