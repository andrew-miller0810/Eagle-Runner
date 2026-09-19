# Eagle-Runner
This repository exists solely to run scheduled and manually-triggered GitHub Actions workflows on behalf of a private companion repository (Eagle). It contains **no application code and no data** — only workflow definitions.

## Why this repo exists

GitHub Actions minutes are billed per-account for private repositories, shared across a limited monthly free tier. Public repositories have no minute cap. Since this repo is public and holds only workflow orchestration (no sensitive code or data), it can run on any schedule without contributing to that private-repo minute budget. The actual application logic and data live in a separate private repository, which this repo checks out at runtime via an authenticated, read/write-scoped token.

## How it works

Each workflow in `.github/workflows/`:

1. Checks out the private companion repository using a fine-grained personal access token stored as a repository secret.
2. Installs dependencies and runs the relevant script(s) from the checked-out code.
3. Where applicable, commits and pushes any resulting data changes back to the private repository.

No code or data from the private repository is stored, cached, or exposed in this repo at any point — it exists only transiently in the runner's workspace during a given job.

## Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `monitor.yml` | Scheduled (frequent) | Polls for and reports on new activity across all configured leagues. |
| `lineup-check.yml` | Scheduled (daily) | Checks for lineup issues ahead of a deadline. |
| `weekly-recap.yml` | Scheduled (weekly) | Posts a recap of recent activity. |
| `season-end.yml` | Manual | Generates an end-of-season report. |
| `hall-of-fame.yml` | Manual | Generates historical/hall-of-fame results. |
| `_shared-league-matrix.yml` | Reusable (called by other workflows) | Determines which leagues to run against, optionally filtered to one. |

Scheduled workflows are currently commented out in favor of an external cron scheduler; trigger any workflow manually via the **Actions** tab using **Run workflow**.

## Secrets and Environments

This repository is configured with:

- A repository-level secret providing read/write access to the private companion repository (used for checkout and for committing results back).
- One GitHub Environment per league, each holding that league's own credentials and webhook secrets, referenced via each workflow's matrix strategy.

No secrets or Environment configuration are duplicated in the private repository — this repo is the sole place where automation credentials are configured and consumed.

## Notes

- Every job runs against a fresh, ephemeral checkout of the private repo's code — nothing persists between runs except what's explicitly committed back.
- Concurrency is controlled per-workflow to prevent overlapping runs from writing conflicting results.
- Because this repo is public, its workflow *files* (structure, step names, which scripts get invoked) are visible to anyone — but no credentials, league data, or application code are ever present here.