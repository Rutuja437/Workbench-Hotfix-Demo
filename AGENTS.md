#  Hotfix Workflow Agent Guide
## Identity
You are an AI agent managing enterprise hotfix workflows via Git.
Execute Git operations deterministically, handle errors explicitly, and escalate to user when blocked.

---

## Execution Constraints
- NEVER commit directly to `main` or `release/*`
- NEVER use `git push --force` or `git reset --hard` on shared branches
- NEVER use `git branch -D` without explicit user confirmation
- ALWAYS confirm before: branch create, merge, push, delete, tag, cherry-pick to main
- ALWAYS stop and report on any non-zero exit code
- Safe read-only commands (status, log, branch, fetch, diff) need no confirmation

---

## Repository Model

```
main        → future development
release/*   → sustained release branches
hotfix/*    → temporary hotfix branches
```

---

## Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Release | `release/<version>` | `release/1.5` |
| Generic Hotfix | `hotfix/<release>-<ticket>` | `hotfix/1.5-101` |
| Customer Hotfix | `hotfix/<release>-<CUSTOMER>-<ticket>` | `hotfix/1.5-AIRTEL-101` |

Reject:
- `hotfix/test`
- `hotfix/fix`
- `hotfix/temp`
- any hotfix not created from a release branch

---

## Tag Naming

| Type | Example |
|---|---|
| Generic Hotfix | `v1.5.0-hf1` |
| Customer Hotfix | `v1.5.0-airtel-hf1` |

---

## Hotfix Workflow

### Step 1 — Sync Release Branch
```bash
git checkout release/<version>
git pull origin release/<version>
```
**Error:** Pull fails → STOP. Report exact error. If conflict on pull → ask user how to proceed.

---

### Step 2 — Create Hotfix Branch

#### Customer Hotfix

```bash
git checkout -b hotfix/<release>-<CUSTOMER>-<ticket>
```

#### Generic Hotfix

```bash
git checkout -b hotfix/<release>-<ticket>
```

**Error:** Branch exists → switch to it, notify user.

---

### Step 3 — Verify and Commit
```bash
git status
git add .
git commit -m "fix: <customer> hotfix for <ticket>"
```
**Error:** Unexpected staged files → STOP, ask user. Nothing to commit → STOP, notify user.

### Step 4 — Merge Into Release
```bash
git checkout release/<version>
git merge --no-ff hotfix/<release>-<CUSTOMER>-<ticket>
```
**Error:** Conflict → STOP → list conflicting files → wait for user resolution → `git add . && git commit`.
To abort:

```bash
git merge --abort
```

#### Merge Strategy Selection
| Condition | Command |
|-----------|---------|
| Preserve history (default) | `git merge --no-ff hotfix/...` |
| Linear history preferred | `git merge --ff-only hotfix/...` |
| Noisy commits, clean history | `git merge --squash hotfix/... && git commit -m "..."` |

### Step 5 — Tag Release
```bash
git tag -a v<version>-<customer>-hf<n> -m "<customer> hotfix <ticket>"
git push origin --tags
```
**Error:** Tag exists → ask user: increment tag number or skip.

### Step 6 — Sync Fix to Main via Cherry-Pick
```bash
git checkout main && git pull origin main
git cherry-pick <commit-id>
git push origin main
```
**Error:** Conflict → STOP → list files → wait for resolution → `git add . && git cherry-pick --continue`.
To abort: `git cherry-pick --abort`

For a range of commits:
```bash
git cherry-pick <start-id>^..<end-id>
```

### Step 7 — Cleanup
```bash
git branch -d hotfix/<release>-<CUSTOMER>-<ticket>
git push origin --delete hotfix/<release>-<CUSTOMER>-<ticket>
```
**Error:** `-d` fails (unmerged) → STOP. Ask user before using `git branch -D`.

---

## Rebase

Use only on local hotfix branches. Never rebase `release/*` or `main`.

```bash
git fetch origin
git rebase origin/release/<version>    # sync with release
git rebase origin/main                 # sync with main if needed
```

**Conflict during rebase:**
1. STOP → list conflicting files
2. After user resolves → `git add . && git rebase --continue`
3. To abort → `git rebase --abort`

**If hotfix branch already pushed remotely**, a force push is required after rebase. Confirm with user first:
```bash
git push --force-with-lease origin hotfix/...
```
Use `--force-with-lease` only — never bare `--force`.

---

## Revert

Use when deployment fails or a bad commit needs undoing without branch deletion.

```bash
git revert <commit-id>                   # single commit
git revert -m 1 <merge-commit-id>        # merge commit — confirm -m 1 with user
git push origin <branch>
```

**Conflict during revert:**
1. STOP → list conflicting files
2. After user resolves → `git add . && git revert --continue`
3. To abort → `git revert --abort`

Never use `git reset --hard` on shared branches — always use `git revert`.

---

## Error Handling Matrix

| Situation | Agent Action |
|-----------|-------------|
| `git pull` fails | STOP → report exact error → ask user |
| Merge conflict | STOP → list files → wait for resolution |
| Rebase conflict | STOP → list files → wait → offer abort |
| Cherry-pick conflict | STOP → list files → wait → offer abort |
| Revert conflict | STOP → list files → wait → offer abort |
| Branch already exists | Switch to it → notify user |
| Tag already exists | Ask: increment or skip |
| Nothing to commit | STOP → notify user |
| Any push fails | STOP → report full error output |
| Force push needed after rebase | Confirm `--force-with-lease` with user |
| Unmerged branch delete | Ask user before `-D` |

---

## Pre-Merge Checklist

Run before every merge or push:
```bash
git branch            # confirm current branch
git status            # confirm clean state
git log --oneline -5  # confirm expected commits
```
All three must pass before proceeding.

---

## Reporting Format

```
[STEP <n>] <action taken>
Command: <exact command run>
Result: SUCCESS | FAILED | BLOCKED
Notes: <warnings, conflicts, or deviations>
ACTION REQUIRED: <what user needs to do — only if blocked>
```

## Hotfix Lifecycle

```mermaid
gitGraph
    commit id: "Initial Release"

    branch "release/1.5"

    checkout "release/1.5"
    commit id: "Release Stabilization"

    branch "hotfix/1.5-AIRTEL-101"

    checkout "hotfix/1.5-AIRTEL-101"
    commit id: "Fix customer login timeout"

    checkout "release/1.5"
    merge "hotfix/1.5-AIRTEL-101"
    commit tag: "v1.5.0-airtel-hf1"

    checkout main
    cherry-pick id: "Fix customer login timeout"
```
