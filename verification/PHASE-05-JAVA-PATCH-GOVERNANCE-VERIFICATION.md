# PHASE-05-JAVA-PATCH-GOVERNANCE-VERIFICATION

## Required Checks
1. `G05-01` Patch policy is enforced for allowed edit boundaries.
   Evidence: patch policy and validation output.
2. `G05-02` L1/L2/L3 classification is attached to each patch.
   Evidence: patch decision metadata.
3. `G05-03` L2/L3 patches are blocked pending human review.
   Evidence: review queue and decision logs.
4. `G05-04` Rollback path is documented and testable.
   Evidence: rollback procedure record and dry-run output.

## Pass Rule
All four checks must pass.

## Reviewer
Automation governance owner.
