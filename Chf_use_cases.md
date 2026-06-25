## UC-001: Tenant-Specific Hotfix

### Objective

Validate that a tenant-specific hotfix is applied only to the target tenant branch and is not propagated to shared branches.

### Repository Setup

```text
main
 │
 └── release/3.8
        │
        ├── tenant/airtel/3.8
        └── tenant/reliance/3.8
```

### Preconditions

* tenant/airtel/3.8 is an active supported branch.
* Issue has been classified as tenant-specific.
* No active hotfix exists for the same ticket.

### Input

Ticket:
ART-001

Issue:
Airtel branding typo

Target Branch:
tenant/airtel/3.8

Propagation Mode:
Tenant Only

Additional Parameters:
None

### Execution

Create and execute a hotfix on the target tenant branch.

### Expected Result

* Hotfix branch created from tenant/airtel/3.8.
* Fix merged back into tenant/airtel/3.8.
* Deployment tag created.
* Deployment artifact generated.
* No propagation outside tenant branch.

### Verification

* Verify fix exists in tenant/airtel/3.8.
* Verify fix does not exist in release/3.8.
* Verify fix does not exist in main.
* Verify deployment tag exists.
* Verify deployment artifact exists.

### Pass Criteria

* Fix present in tenant/airtel/3.8.
* Fix absent from release/3.8.
* Fix absent from main.
* Tag created successfully.
* Artifact generated successfully.

### Failure Criteria

* Fix propagated to release branch.
* Fix propagated to main.
* Tag missing.
* Artifact missing.
* Merge failure.

### Notes

Validates tenant-only propagation rules.

---

## UC-002: Shared Product Hotfix

### Objective

Validate that a shared product defect is propagated through tenant, release, and main branches.

### Repository Setup

```text
main
 │
 └── release/3.8
        │
        ├── tenant/airtel/3.8
        ├── tenant/reliance/3.8
        └── tenant/tata/3.8
```

### Preconditions

* Defect affects multiple tenants.
* Release branch is supported.
* Impact analysis approved propagation.

### Input

Ticket:
BUG-451

Issue:
Shared framework defect

Target Branch:
tenant/airtel/3.8

Propagation Mode:
Release + Main

Additional Parameters:
Affected Release Line = 3.8

### Execution

Execute tenant hotfix and propagate validated fix to release and main.

### Expected Result

* Fix applied to tenant branch.
* Fix propagated to release branch.
* Fix propagated to main.
* Release artifact generated.

### Verification

* Verify fix exists in tenant/airtel/3.8.
* Verify fix exists in release/3.8.
* Verify fix exists in main.
* Verify release tag exists.
* Verify release artifact exists.

### Pass Criteria

* Fix available in all required branches.
* Release tag created.
* Release artifact generated.

### Failure Criteria

* Fix missing from release branch.
* Fix missing from main.
* Release tag missing.
* Release artifact missing.

### Notes

Validates shared-defect propagation flow.

---

## UC-003: Tenant CHF

### Objective

Validate that multiple approved tenant fixes can be packaged into a single CHF.

### Repository Setup

```text
tenant/airtel/3.8
```

### Preconditions

* Multiple approved fixes exist.
* Tenant branch is active.
* All fixes passed review.

### Input

Ticket:
CHF-001

Issue:
Multiple approved tenant fixes

Target Branch:
tenant/airtel/3.8

Propagation Mode:
Tenant Only

Additional Parameters:

HF-101
HF-102
HF-103

### Execution

Create CHF containing all approved fixes.

### Expected Result

* CHF branch created.
* All approved fixes included.
* CHF merged back into tenant branch.
* CHF tag created.
* CHF artifact generated.

### Verification

* Verify all fixes exist in CHF.
* Verify CHF tag exists.
* Verify CHF artifact exists.

### Pass Criteria

* All approved fixes included.
* CHF merged successfully.
* Tag generated.
* Artifact generated.

### Failure Criteria

* Missing approved fix.
* CHF merge failure.
* Tag missing.
* Artifact missing.

### Notes

Validates cumulative tenant patch delivery.

---

## UC-004: Release CHF

### Objective

Validate that multiple shared fixes can be aggregated into a release CHF.

### Repository Setup

```text
release/3.8
```

### Preconditions

* Multiple approved shared fixes exist.
* Release branch is supported.

### Input

Ticket:
CHF-REL-001

Issue:
Release patch set

Target Branch:
release/3.8

Propagation Mode:
Release Branch

Additional Parameters:

BUG-451
BUG-488
BUG-501

### Execution

Create a Consolidated Hotfix (CHF) branch from release/3.8 that combines all approved fixes included in the request and prepares them for release validation.

### Expected Result

* Release CHF created.
* Approved fixes aggregated.
* CHF merged into release branch.
* Release tag created.
* Release artifact generated.

### Verification

* Verify all fixes exist in release CHF.
* Verify release tag exists.
* Verify release artifact exists.

### Pass Criteria

* All approved fixes included.
* Release tag created.
* Release artifact generated.

### Failure Criteria

* Missing fix.
* CHF merge failure.
* Release tag missing.
* Release artifact missing.

### Notes

Validates release packaging workflow.

---

## UC-005: Forward-Port Release Fixes to Main

### Objective

Validate that validated release fixes are propagated to main.

### Repository Setup

```text
main
release/3.8
```

### Preconditions

* Release fixes already validated.
* Main branch available.

### Input

Ticket:
FP-001

Issue:
Forward-port validated release fixes

Target Branch:
main

Propagation Mode:
Forward Port

Additional Parameters:

BUG-451
BUG-488

### Execution

Propagate validated release fixes to main.

### Expected Result

* Validated fixes become available in main.

### Verification

* Verify fixes exist in main.
* Verify propagated fixes match release branch fixes.

### Pass Criteria

* All validated fixes available in main.

### Failure Criteria

* Missing fix in main.
* Propagation failure.

### Notes

Validates future-development synchronization.

---

## UC-008: Cherry-Pick Conflict Handling

### Objective

Validate escalation handling when a fix cannot be applied cleanly.

### Repository Setup

Target branch contains conflicting code.

### Preconditions

* Incoming fix conflicts with target branch state.

### Input

Ticket:
BUG-451

Issue:
Shared defect

Target Branch:
release/3.8

Propagation Mode:
Release Branch

Additional Parameters:
Conflict expected

### Execution

Attempt propagation of fix.

### Expected Result

* Execution stops.
* Escalation artifact generated.
* No automatic conflict resolution performed.

### Verification

* Verify escalation artifact exists.
* Verify conflict details recorded.
* Verify no further propagation occurred.

### Pass Criteria

* Conflict escalated successfully.

### Failure Criteria

* Silent conflict resolution.
* Partial propagation.
* Missing escalation record.

### Notes

Validates safe failure handling.

---

## UC-009: End-of-Life Branch Protection

### Objective

Validate that EoL branches reject new CHF and hotfix requests.

### Repository Setup

```text
release/3.8
Status: EoL
```

### Preconditions

* Branch marked End-of-Life.

### Input

Ticket:
CHF-999

Issue:
Attempt to patch EoL release

Target Branch:
release/3.8

Propagation Mode:
Release Branch

Additional Parameters:
None

### Execution

Attempt CHF creation.

### Expected Result

Request rejected.

### Verification

* Verify no CHF branch created.
* Verify no new commits added.
* Verify rejection recorded.

### Pass Criteria

* EoL branch remains unchanged.

### Failure Criteria

* CHF created.
* Commit added.
* Branch modified.

### Notes

Validates EoL enforcement.

---

