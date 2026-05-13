# ART Hotfix Workflow Demo Project

## Overview

This project demonstrates a simple enterprise-style hotfix workflow using Git and GitHub.

The ART platform contains customer-specific modules:

- Airtel
- Reliance
- Tata

The project was used to practice:

- release branches
- hotfix branches
- merge conflicts
- conflict resolution
- release synchronization
- cherry-pick workflow

---

# Repository Structure

```text
Workbench-Hotfix-Demo/
│
├── ART/
│   ├── Airtel/
│   │   └── README.md
│   │
│   ├── Reliance/
│   │   └── README.md
│   │
│   └── Tata/
│       └── README.md
│
└── docs/
    └── implementation.md
```

---

# Release Branch

```text
release/1.5
```

The release branch represents the active production release.

---

# Hotfix Branches

```text
hotfix/1.5-AIRTEL-101
hotfix/1.5-BUG-201
hotfix/1.5-BUG-202
```

These branches were created to isolate production fixes.

---

# Hotfix Workflow

## Initial Commit

```bash
git add .
git commit -m "Initial ART customer modules setup"
```

---

## Create Release Branch

```bash
git checkout -b release/1.5
```

---

## Create Hotfix Branch

```bash
git checkout -b hotfix/1.5-AIRTEL-101
```

---

## Implement Hotfix

Updated:

```text
ART/Airtel/README.md
```

Example fix:

```text
Deployment Command: npm run build-prod
```

---

## Commit Hotfix

```bash
git add .
git commit -m "fix: corrected Airtel deployment command"
```

---

## Merge Hotfix into Release Branch

```bash
git checkout release/1.5
git merge hotfix/1.5-AIRTEL-101
```

---

## Merge Release into Main

```bash
git checkout main
git merge release/1.5
```

---

# Merge Conflict Simulation

A merge conflict scenario was created using the Reliance README file.

Two hotfix branches modified the same line differently.

---

## First Hotfix Branch

Modified:

```text
Environment: Production-US
```

Commit:

```bash
git add .
git commit -m "fix: updated Reliance environment"
```

---

## Second Hotfix Branch

Modified same line differently:

```text
Environment: Production-India
```

Commit:

```bash
git add .
git commit -m "fix: updated Reliance deployment region"
```

---

## Merge Conflict

When merging both branches:

```bash
git merge hotfix/1.5-BUG-202
```

Git generated conflict:

```text
CONFLICT (content): Merge conflict in ART/Reliance/README.md
```

---

## Conflict Markers

```text
<<<<<<< HEAD
Environment: Production-US
=======
Environment: Production-India
>>>>>>> hotfix/1.5-BUG-202
```

---

## Conflict Resolution

Final example:

```text
Environment: Production-India-US
```

Completed with:

```bash
git add .
git commit -m "resolved merge conflict for Reliance environment"
```

---

# Cherry-pick Understanding

Selective synchronization can be performed using:

```bash
git cherry-pick <commit-id>
```

This is useful when:
- only specific fixes should move to another branch
- customer-specific changes should not affect main

---

# Key Learnings

From this project, I understood:

- how enterprise release branches work
- how hotfix branches isolate production fixes
- how merge conflicts occur
- how merge conflicts are resolved
- why release synchronization matters
- why cherry-pick is useful
- how enterprise Git workflows are managed

---
