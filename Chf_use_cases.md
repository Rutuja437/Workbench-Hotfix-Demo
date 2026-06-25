# UC-001: Tenant-Specific Hotfix

## Brief Description

Validate that a tenant-specific hotfix is applied only to the target tenant branch and is not propagated to shared branches.

## Primary Actor

Developer

## Repository Setup

```text
main
 │
 └── release/3.8
        │
        ├── tenant/airtel/3.8
        └── tenant/reliance/3.8
```

## Preconditions

* tenant/airtel/3.8 is an active supported branch.
* Issue has been classified as tenant-specific.
* No active hotfix exists for the same ticket.

## Input

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

## Main Success Flow

1. Create a hotfix branch from tenant/airtel/3.8.
2. Apply the requested fix.
3. Validate the fix.
4. Merge the hotfix back into tenant/airtel/3.8.
5. Create deployment tag.
6. Generate deployment artifact.

## Alternative / Exception Flows

E1. Validation fails.
- Hotfix is not merged.

E2. Merge fails.
- Execution stops and reports failure.

## Expected Result

* Hotfix branch created from tenant/airtel/3.8.
* Fix merged back into tenant/airtel/3.8.
* Deployment tag created.
* Deployment artifact generated.
* No propagation outside tenant branch.

## Expected Git Graph

```text
A ── B ── C ─────────── M  (tenant/airtel/3.8)
          \            /
           HF-ART001──

release/3.8
    │
    └── unchanged

main
    │
    └── unchanged
```

## Verification

### Automated / Harness Checks

* Verify fix exists in tenant/airtel/3.8.
* Verify fix does not exist in release/3.8.
* Verify fix does not exist in main.
* Verify deployment tag exists.
* Verify deployment artifact exists.

### Manual Inspection Points

* Confirm no propagation outside tenant branch.

## Pass Criteria

* Fix present in tenant/airtel/3.8.
* Fix absent from release/3.8.
* Fix absent from main.
* Tag created successfully.
* Artifact generated successfully.

## Failure Criteria

* Fix propagated to release branch.
* Fix propagated to main.
* Tag missing.
* Artifact missing.
* Merge failure.

## Postconditions / Guarantees

* tenant/airtel/3.8 contains the fix.
* Shared branches remain unchanged.

## Notes

Validates tenant-only propagation rules.

---

# UC-002: Shared Product Hotfix

## Brief Description

Validate that a shared product defect is propagated through tenant, release, and main branches.

## Primary Actor

Developer

## Repository Setup

```text
main
 │
 └── release/3.8
        │
        ├── tenant/airtel/3.8
        ├── tenant/reliance/3.8
        └── tenant/tata/3.8
```

## Preconditions

* Defect affects multiple tenants.
* Release branch is supported.
* Impact analysis approved propagation.

## Input

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

## Main Success Flow

1. Create tenant hotfix branch.
2. Apply and validate the fix.
3. Merge fix into tenant/airtel/3.8.
4. Propagate validated fix to release/3.8.
5. Propagate validated fix to main.
6. Generate release artifact.

## Alternative / Exception Flows

E1. Validation fails.
- Propagation does not occur.

E2. Propagation fails.
- Execution stops and reports failure.

## Expected Result

* Fix applied to tenant branch.
* Fix propagated to release branch.
* Fix propagated to main.
* Release artifact generated.

## Expected Git Graph

```text
tenant/airtel/3.8

A ── B ── C ─────────── M
          \            /
           HF-451─────

release/3.8

R1 ── R2 ── FIX-451

main

M1 ── M2 ── FIX-451
```

## Verification

### Automated / Harness Checks

* Verify fix exists in tenant/airtel/3.8.
* Verify fix exists in release/3.8.
* Verify fix exists in main.
* Verify release tag exists.
* Verify release artifact exists.

### Manual Inspection Points

* Confirm propagation path follows approved impact analysis.

## Pass Criteria

* Fix available in all required branches.
* Release tag created.
* Release artifact generated.

## Failure Criteria

* Fix missing from release branch.
* Fix missing from main.
* Release tag missing.
* Release artifact missing.

## Postconditions / Guarantees

* Fix available in tenant, release, and main branches.

## Notes

Validates shared-defect propagation flow.

---

# UC-003: Tenant CHF

## Brief Description

Validate that multiple approved tenant fixes can be packaged into a single CHF.

## Primary Actor

Release Engineer

## Repository Setup

```text
tenant/airtel/3.8
```

## Preconditions

* Multiple approved fixes exist.
* Tenant branch is active.
* All fixes passed review.

## Input

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

## Main Success Flow

1. Create CHF branch.
2. Include approved fixes.
3. Validate combined patch set.
4. Merge CHF into tenant branch.
5. Create CHF tag.
6. Generate CHF artifact.

## Alternative / Exception Flows

E1. One fix fails validation.
- CHF execution stops.

## Expected Result

* CHF branch created.
* All approved fixes included.
* CHF merged back into tenant branch.
* CHF tag created.
* CHF artifact generated.

## Expected Git Graph

```text
A ── B ── C ─────────────────── M
          \                    /
           HF-101 ─ HF-102 ─ HF-103
```

## Verification

### Automated / Harness Checks

* Verify all fixes exist in CHF.
* Verify CHF tag exists.
* Verify CHF artifact exists.

### Manual Inspection Points

* Confirm all approved fixes are included.

## Pass Criteria

* All approved fixes included.
* CHF merged successfully.
* Tag generated.
* Artifact generated.

## Failure Criteria

* Missing approved fix.
* CHF merge failure.
* Tag missing.
* Artifact missing.

## Postconditions / Guarantees

* Tenant branch contains all approved fixes.

## Notes

Validates cumulative tenant patch delivery.

---

# UC-004: Release CHF

## Brief Description

Validate that multiple shared fixes can be aggregated into a release CHF.

## Primary Actor

Release Engineer

## Repository Setup

```text
release/3.8
```

## Preconditions

* Multiple approved shared fixes exist.
* Release branch is supported.

## Input

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

## Main Success Flow

1. Create CHF branch from release/3.8.
2. Aggregate approved fixes.
3. Validate combined patch set.
4. Merge CHF into release branch.
5. Create release tag.
6. Generate release artifact.

## Alternative / Exception Flows

E1. One fix fails validation.
- CHF execution stops.

## Expected Result

* Release CHF created.
* Approved fixes aggregated.
* CHF merged into release branch.
* Release tag created.
* Release artifact generated.

## Expected Git Graph

```text
R1 ── R2 ── R3 ─────────────── M
               \             /
                CHF-REL-001
                   │
                   ├── BUG-451
                   ├── BUG-488
                   └── BUG-501
```

## Verification

### Automated / Harness Checks

* Verify all fixes exist in release CHF.
* Verify release tag exists.
* Verify release artifact exists.

### Manual Inspection Points

* Confirm release patch set contains all approved fixes.

## Pass Criteria

* All approved fixes included.
* Release tag created.
* Release artifact generated.

## Failure Criteria

* Missing fix.
* CHF merge failure.
* Release tag missing.
* Release artifact missing.

## Postconditions / Guarantees

* Release branch contains approved CHF contents.

## Notes

Validates release packaging workflow.

---

# UC-005: Forward-Port Release Fixes to Main

## Brief Description

Validate that validated release fixes are propagated to main.

## Primary Actor

Release Engineer

## Repository Setup

```text
main
release/3.8
```

## Preconditions

* Release fixes already validated.
* Main branch available.

## Input

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

## Main Success Flow

1. Identify validated release fixes.
2. Propagate fixes to main.
3. Validate propagated changes.

## Alternative / Exception Flows

E1. Propagation failure.
- Execution stops and reports failure.

## Expected Result

* Validated fixes become available in main.

## Expected Git Graph

```text
release/3.8

R1 ── R2 ── FIX-451 ── FIX-488


main

M1 ── M2 ── FP-451 ── FP-488
```

## Verification

### Automated / Harness Checks

* Verify fixes exist in main.
* Verify propagated fixes match release branch fixes.

### Manual Inspection Points

* Confirm release and main contain equivalent fixes.

## Pass Criteria

* All validated fixes available in main.

## Failure Criteria

* Missing fix in main.
* Propagation failure.

## Postconditions / Guarantees

* Main contains validated release fixes.

## Notes

Validates future-development synchronization.

---

# UC-008: Cherry-Pick Conflict Handling

## Brief Description

Validate escalation handling when a fix cannot be applied cleanly.

## Primary Actor

HOTFIX-ORCHESTRATOR

## Repository Setup

Target branch contains conflicting code.

## Preconditions

* Incoming fix conflicts with target branch state.

## Input

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

## Main Success Flow

1. Attempt propagation of fix.
2. Detect conflict.
3. Stop execution.
4. Generate escalation information.

## Alternative / Exception Flows

E1. Conflict detected.
- Escalate and stop.

## Expected Result

* Execution stops.
* Escalation artifact generated.
* No automatic conflict resolution performed.

## Expected Git Graph

```text
release/3.8

R1 ── R2 ── CONFLICTING_CHANGE

(No new commit created)
```

## Verification

### Automated / Harness Checks

* Verify escalation artifact exists.
* Verify conflict details recorded.
* Verify no further propagation occurred.

### Manual Inspection Points

* Confirm no manual conflict resolution was performed.

## Pass Criteria

* Conflict escalated successfully.

## Failure Criteria

* Silent conflict resolution.
* Partial propagation.
* Missing escalation record.

## Postconditions / Guarantees

* Repository remains unchanged after conflict detection.

## Notes

Validates safe failure handling.

---

# UC-009: End-of-Life Branch Protection

## Brief Description

Validate that EoL branches reject new CHF and hotfix requests.

## Primary Actor

Release Manager

## Repository Setup

```text
release/3.8
Status: EoL
```

## Preconditions

* Branch marked End-of-Life.

## Input

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

## Main Success Flow

1. Receive CHF request.
2. Check branch support status.
3. Reject execution.

## Alternative / Exception Flows

E1. Branch incorrectly marked active.
- Continue normal processing.

## Expected Result

Request rejected.

## Expected Git Graph

```text
release/3.8 (EoL)

R1 ── R2 ── R3
```

## Verification

### Automated / Harness Checks

* Verify no CHF branch created.
* Verify no new commits added.
* Verify rejection recorded.

### Manual Inspection Points

* Confirm EoL policy enforcement.

## Pass Criteria

* EoL branch remains unchanged.

## Failure Criteria

* CHF created.
* Commit added.
* Branch modified.

## Postconditions / Guarantees

* EoL branch history remains unchanged.

## Notes

Validates EoL enforcement.
