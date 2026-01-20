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

Read the `origin` remote and extract `OWNER/REPO` from GitHub URLs (https or ssh):

```bash
REMOTE_URL="$(git config --get remote.origin.url)"
OWNER_REPO="$(
  python3 - <<'PY'
import os, re, sys
url = os.environ.get("REMOTE_URL", "")
m = re.search(r"github\\.com[:/](.+?)(?:\\.git)?$", url)
if not m:
    sys.exit("Could not parse GitHub owner/repo from remote.origin.url")
print(m.group(1))
PY
)"
```

If parsing fails or the remote is not GitHub, ask the user for `OWNER/REPO`.

### 3) Force-cancel the run by ID (repeat 10x)

Use the run ID provided by the user:

```bash
RUN_ID="21150788856"
for _ in {1..10}; do
  gh api \
    --method POST \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2022-11-28" \
    "/repos/$OWNER_REPO/actions/runs/$RUN_ID/force-cancel"
done
```

If the user requests a normal cancel instead of force-cancel, swap the endpoint to:

```bash
for _ in {1..10}; do
  gh api \
    --method POST \
    -H "Accept: application/vnd.github+json" \
    -H "X-GitHub-Api-Version: 2022-11-28" \
    "/repos/$OWNER_REPO/actions/runs/$RUN_ID/cancel"
done
```
