---
fest_type: sequence
fest_id: 01_ci_pipeline
fest_name: ci_pipeline
fest_parent: 003_IMPLEMENT
fest_order: 1
fest_status: completed
fest_created: 2026-09-15T11:54:21.349502-06:00
fest_updated: 2026-09-16T01:21:33.556919-06:00
fest_tracking: true
---



# Sequence Goal: 01_ci_pipeline

**Sequence:** 01_ci_pipeline | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** Make CI actually run on the fork, and give every later slice a pinned JDK 17/21 build-and-test check with readable failures and Gradle caching.

**Contribution to Phase Goal:** Every feature PR in 003_IMPLEMENT uses this check as its merge criterion (D012). This sequence also satisfies R3 and R10.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **Zero-run cause**: the real reason the fork had 0 workflow runs, with raw API output and the fix recorded in `results/`
- [ ] **JDK 17/21 workflow**: `.github/workflows/gradle.yml` replaced: 17/21 matrix, fail-fast off, setup-gradle caching, SHA-pinned actions, JUnit annotations, and a report artifact on failure
- [ ] **Red-path proof**: a throwaway failing test turned the check red with a readable annotation; evidence recorded, branch deleted

### Quality Standards

- [ ] **Pinned supply chain**: every third-party action is pinned by full commit SHA, with its release tag in a comment
- [ ] **Both JDKs green**: the PR shows passing JDK 17 and JDK 21 checks before merge

### Completion Criteria

- [ ] All tasks in sequence completed successfully
- [ ] Quality verification tasks passed
- [ ] Code review completed and issues addressed
- [ ] Documentation updated

## Task Alignment

> **Note:** This table should be populated AFTER creating task files.
> SEQUENCE_GOAL.md defines WHAT to accomplish. Task files define HOW.
> Run `fest create task` to create tasks, then update this table.

| Task | Task Objective | Contribution to Sequence Goal |
|------|----------------|-------------------------------|
| 01_diagnose_zero_ci_runs | Diagnose why the fork has zero CI runs | Find out why `CI with Gradle` has never run on the fork, get the fork to a state where workflows run, and record the real cause with evidence. |
| 02_replace_workflow_with_jdk_matrix | Replace the workflow with a pinned JDK 17/21 matrix | Replace `.github/workflows/gradle.yml` with a JDK 17/21 matrix that caches Gradle, pins every action by commit SHA, and reports test failures as annotations. |
| 03_prove_ci_fails_red | Prove CI turns red with a readable annotation | Prove the new pipeline turns red, with a per-test annotation, when a test fails; then remove the probe without merging it. |

## Dependencies

### Prerequisites (from other sequences)

- 002_PLAN: approved decisions D011 and D012

### Provides (to other sequences)

- a required green JDK 17/21 check: Used by 02_article_foundation through 07_submission_docs

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| The fork's workflows stay disabled until a human enables them in the GitHub UI | High | High | diagnose first, ask the user to enable the fork opt-in, then confirm with a real run |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: cause of zero runs recorded
- [ ] **Milestone 2**: matrix workflow green on its PR
- [ ] **Milestone 3**: red-path evidence recorded and the PR merged

## Quality Gates

### Testing and Verification

- [ ] All unit tests pass
- [ ] Integration tests complete
- [ ] Performance benchmarks met

### Code Review

- [ ] Code review conducted
- [ ] Review feedback addressed
- [ ] Standards compliance verified

### Iteration Decision

- [ ] Need another iteration? No
- [ ] If yes, new tasks created: None