# Hotfix Lifecycle - My Understanding

## Overview

Our repository contains the following types of branches:

* **main** – Contains the latest stable code and serves as the source for future releases.
* **release** – Represents a product release (e.g., `release/1.0`).
* **tenant** – Contains customer-specific customizations for a particular release.
* **hotfix** – A temporary branch created to fix a bug.
* **CHF (Combined Hotfix)** – An optional branch used to combine multiple hotfixes before merging.

---

# Branch Structure

```text
main
 │
 └── release/1.0
        │
        ├── tenant/customerA
        ├── tenant/customerB
        └── tenant/customerC
```
*main → Future development.
*release/1.0 → Stable code for version 1.0.
*tenant/customerA → Version 1.0 + Customer A's customizations.

---

# Hotfix Lifecycle

## Step 1 - Bug Report

A customer reports an issue.

First, determine whether the issue is:

* Tenant-specific (affects only one customer)
* Generic (affects all customers)

---

## Step 2 - Create a Hotfix Branch

Create the hotfix branch from the corresponding tenant branch.

```text
tenant/customerA
      │
      └── hotfix/HF-101
```

The developer implements the fix in this branch.

---

## Step 3 - Code Review & Testing

* Raise a Pull Request.
* Perform code review.
* Execute testing.
* Once approved, the hotfix is ready to merge.

---

## Step 4 - Merge the Hotfix

### Option 1: Using a CHF Branch

If multiple hotfixes are being released together:

```text
HF-101
HF-102
HF-103
    │
    ▼
CHF
    │
    ▼
tenant/customerA
```

### Option 2: Without a CHF Branch

If only one hotfix exists:

```text
hotfix/HF-101
      │
      ▼
tenant/customerA
```

---

# Cherry-Pick Strategy

## Generic Fix

If the fix applies to all customers:

1. Merge the hotfix into the tenant branch.
2. Cherry-pick the commit to the release branch.
3. Cherry-pick the same commit to the main branch.

```text
hotfix/HF-101
      │
      ▼
tenant/customerA
      │
Cherry Pick
      ▼
release/1.0
      │
Cherry Pick
      ▼
main
```

**Reason:**

* The **release** branch receives the fix for the current release.
* The **main** branch receives the fix so that future releases created from `main` also include it.

---

## Tenant-Specific Fix

If the fix is only applicable to one customer:

* Merge it only into the tenant branch.
* Do **not** cherry-pick it to the release branch.
* Do **not** cherry-pick it to the main branch.

```text
hotfix/HF-101
      │
      ▼
tenant/customerA
```

Examples:

* Customer branding
* Customer-specific configuration
* Customer-specific business logic

---

# What Happens to the Main Branch?

The **main** branch should contain all **generic product fixes**.

After a generic fix is applied to the tenant and release branches, it should also be cherry-picked to the **main** branch.

This ensures:

* Future releases created from `main` include the fix.
* The same bug does not reappear in future versions.

Tenant-specific changes should **never** be merged into the **main** branch.

---

# Branch Cleanup

| Branch  | Action                                                               |
| ------- | -------------------------------------------------------------------- |
| Hotfix  | Delete after it is merged.                                           |
| CHF     | Delete after it is merged into the tenant branch.                    |
| Tenant  | Keep for future tenant hotfixes (unless the team decides otherwise). |
| Release | Keep until the release reaches End of Life (EOL).                    |
| Main    | Never delete.                                                        |

---

# Complete Flow

```text
Customer reports bug
        │
        ▼
Create Hotfix Branch
        │
        ▼
Developer fixes the issue
        │
        ▼
Code Review & Testing
        │
        ▼
Merge Hotfix
        │
        ▼
(Optional) Merge via CHF
        │
        ▼
Merge into Tenant Branch
        │
        ▼
Is the fix generic?
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
Cherry-pick  Keep only in
to Release   Tenant
   │
   ▼
Cherry-pick
to Main
   │
   ▼
Delete Hotfix / CHF
```

---
