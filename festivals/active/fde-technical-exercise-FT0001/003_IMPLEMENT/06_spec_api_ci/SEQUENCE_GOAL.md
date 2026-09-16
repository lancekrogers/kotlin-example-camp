---
fest_type: sequence
fest_id: 06_spec_api_ci
fest_name: spec_api_ci
fest_parent: 003_IMPLEMENT
fest_order: 6
fest_status: completed
fest_created: 2026-09-15T11:54:21.475387-06:00
fest_updated: 2026-09-16T12:14:48.875282-06:00
fest_tracking: true
---



# Sequence Goal: 06_spec_api_ci

**Sequence:** 06_spec_api_ci | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** The bundled RealWorld Postman collection runs in CI against a live container, and fails only on regressions or a stale expected-failures manifest.

**Contribution to Phase Goal:** Delivers the brief's §4 bonus (R9) in a form that stays meaningful while some endpoints remain stubbed.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **Pinned runner**: `spec-api/run-api-tests.sh` pins `newman@6.2.2` and refuses to run without an explicit `APIURL`
- [ ] **Manifest and comparator**: `spec-api/expected-failures.txt` generated from a real run, plus a comparator that fails on unexpected failures and on unexpected passes
- [ ] **CI spec job**: a `spec` job that builds the image, starts the app with a generated `JWT_SECRET`, waits for health, then runs newman and the comparator

### Quality Standards

- [ ] **No remote calls**: the job and the local recipe always point `APIURL` at the local container
- [ ] **Honest signal**: a deliberately broken manifest entry makes the job fail; proven once and recorded

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
| 01_pin_newman_and_guard_api_url | Pin newman and require an explicit APIURL | Pin newman, make the spec runner refuse to run without an explicit `APIURL`, and add a containerized local recipe that runs the collection against the local app. |
| 02_expected_failures_manifest_and_compare_script | Add the expected-failures manifest and comparator | Generate `spec-api/expected-failures.txt` from a real run, and add a comparator that fails on any unexpected failure or unexpected pass. |
| 03_spec_job_in_workflow | Add the spec-test job to CI | Add a `spec` job to the CI workflow that runs the RealWorld collection against a freshly built container and gates on the comparator. |

## Dependencies

### Prerequisites (from other sequences)

- 05_user_activity: the final implemented endpoints that the manifest reflects

### Provides (to other sequences)

- spec-test evidence for the submission: Used by 07_submission_docs

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Postman request names are not unique across folders | High | High | the comparator keys results by folder plus request name, resolved from the collection file |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: runner pinned and guarded
- [ ] **Milestone 2**: manifest generated from a real run
- [ ] **Milestone 3**: spec job green on its PR and merged

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