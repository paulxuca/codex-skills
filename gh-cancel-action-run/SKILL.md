---
name: gh-cancel-action-run
description: "Cancel or force-cancel a GitHub Actions workflow run by run ID using the GitHub CLI (gh api) and the current repository's owner/repo from .git. Use for requests like \"cancel run 12345\", \"stop a workflow run by ID\", or \"force-cancel an in-progress GitHub Actions run.\""
---

# Gh Cancel Action Run

## Overview

Cancel a GitHub Actions workflow run by ID using the GitHub CLI, deriving owner/repo from the current repo's `.git` remote. This skill focuses on force-canceling runs but can also use the normal cancel endpoint when needed.

## Workflow

### 1) Confirm repo context

Ensure the current working directory is inside the target repo:

```bash
git rev-parse --show-toplevel
```

If this fails or the repo is not the target, stop and ask for the correct repo path.

### 2) Derive owner/repo from `.git`

Read the `origin` remote from `git remote -v` and extract `OWNER/REPO` using shell tools:

```bash
REMOTE_URL="$(
  git remote -v | awk '$1=="origin" {print $2; exit}'
)"
OWNER_REPO="$(
  printf '%s' "$REMOTE_URL" | perl -ne 'if (m{^(?:git@|https://)github.com[:/](.+?)(?:\\.git)?$}) {print $1}'
)"
if [ -z "$OWNER_REPO" ] || [ "$OWNER_REPO" = "$REMOTE_URL" ]; then
  echo "Could not parse GitHub owner/repo from git remote -v"
  exit 1
fi
```

If parsing fails or the remote is not GitHub, ask the user for `OWNER/REPO`.

### 3) Force-cancel the run by ID (loop until 409)

Use the run ID provided by the user:

```bash
RUN_ID="21150788856"
while true; do
  RESPONSE="$(
    gh api \
      --method POST \
      --include \
      -H "Accept: application/vnd.github+json" \
      -H "X-GitHub-Api-Version: 2022-11-28" \
      "/repos/$OWNER_REPO/actions/runs/$RUN_ID/force-cancel" 2>&1
  )"
  STATUS="$(printf '%s' "$RESPONSE" | awk 'toupper($0) ~ /^HTTP\\// {print $2; exit}')"
  if [ -z "$STATUS" ]; then
    echo "Could not read HTTP status from response"
    echo "$RESPONSE"
    exit 1
  fi
  if [ "$STATUS" = "409" ]; then
    echo "Run already completed (HTTP 409). Stopping."
    break
  fi
done
```

If the user requests a normal cancel instead of force-cancel, swap the endpoint to:

```bash
while true; do
  RESPONSE="$(
    gh api \
      --method POST \
      --include \
      -H "Accept: application/vnd.github+json" \
      -H "X-GitHub-Api-Version: 2022-11-28" \
      "/repos/$OWNER_REPO/actions/runs/$RUN_ID/cancel" 2>&1
  )"
  STATUS="$(printf '%s' "$RESPONSE" | awk 'toupper($0) ~ /^HTTP\\// {print $2; exit}')"
  if [ -z "$STATUS" ]; then
    echo "Could not read HTTP status from response"
    echo "$RESPONSE"
    exit 1
  fi
  if [ "$STATUS" = "409" ]; then
    echo "Run already completed (HTTP 409). Stopping."
    break
  fi
done
```

### Failure handling notes

- If the run is already completed, GitHub returns HTTP 409 with `Cannot cancel a workflow run that is completed.` This is expected; report that the run is already done.
