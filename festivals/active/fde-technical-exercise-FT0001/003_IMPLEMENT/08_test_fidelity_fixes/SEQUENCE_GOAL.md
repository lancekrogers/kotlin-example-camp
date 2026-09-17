---
fest_type: sequence
fest_id: 08_test_fidelity_fixes
fest_name: test fidelity fixes
fest_parent: 003_IMPLEMENT
fest_order: 8
fest_status: completed
fest_created: 2026-09-16T12:58:37.209057-06:00
fest_updated: 2026-09-16T13:41:37.489843-06:00
fest_tracking: true
---


# Sequence Goal: 08_test_fidelity_fixes

**Sequence:** 08_test_fidelity_fixes | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-16T12:58:37-06:00

## Sequence Objective

**Primary Goal:** Make two tests that currently pass without proving their claims able to fail, and remove the helper
defect that caused one of them.

**Contribution to Phase Goal:** The phase requires each slice merged as a green slice with review feedback either
incorporated or explicitly deferred with justification. Both items were found by PR review in slices 03 and 05 and
deferred for a reason that has since expired: the fix would have rewritten published branches carrying fresh
approvals. Everything is now merged and those branches are deleted, so the justification no longer holds and the
findings should be incorporated instead.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **A `String`-typed raw-JSON request helper**: `HttpUtil` gains a method whose body parameter is declared `String`,
  so Kotlin binds Unirest's raw `body(String)` overload instead of `body(Object)`. The existing `postRaw(path, body: Any)`
  is left in place for DTO call sites, where object serialization is correct, and gains a comment saying why a raw JSON
  string must not be passed to it.
- [ ] **`CommentCreateTest.missing body returns 422` exercises an absent field**: it sends the object `{"comment":{}}`
  on the wire and fails if the server does not reject a payload whose `body` field is missing. Proven by capturing the
  request body actually sent, not by the status code alone — both the old and new payloads return an identical
  `422 {"errors":{"body":["Comment is invalid."]}}`, so status cannot distinguish them.
- [ ] **A discriminating `%` literal-wildcard test**: an assertion that fails if `%` in a search term is treated as a
  wildcard. The existing test cannot fail, because term `…100%` against title `…100% pure_x` matches whether or not the
  `%` is escaped (`%%` collapses to `%`). The replacement must assert **0** matches for a term containing `%` where a
  wildcard reading would produce a match, mirroring the live API result `GET /articles/search?q=%` → `articlesCount: 0`.

### Quality Standards

- [ ] **Every changed test is shown to fail before it is shown to pass**: for each of the two tests, run it against
  deliberately broken behaviour and record the failure, then against the real code and record the pass. A test that has
  only ever been green is exactly the defect this sequence exists to remove, so recording both directions is the point
  of the sequence rather than an extra.
- [ ] **No test is weakened and no assertion is deleted to make anything pass**: the census must not lose running tests.
  `ran` stays at 94 or rises; `failed` stays 0; `skipped` stays at 16.
- [ ] **All builds and tests run in Docker through the `just` modules, never on the host toolchain (C7)**
- [ ] **Commits use `fest commit` with no AI attribution; the PR targets `lancekrogers/kotlin-ktor-realworld-example-app` `master` explicitly (D012)**

### Completion Criteria

- [ ] All tasks in sequence completed successfully
- [ ] Quality verification tasks passed
- [ ] Code review completed and issues addressed
- [ ] Documentation updated

## Task Alignment

| Task | Task Objective | Contribution to Sequence Goal |
|------|----------------|-------------------------------|
| 01_raw_json_helper_and_missing_field_test | Add the `String`-typed helper and make the missing-field test real | Removes the helper defect and the false-confidence test it caused |
| 02_discriminating_percent_test | Replace the `%` assertion with one that can fail | Closes the second review finding |
| 03_verify_both_directions | Prove each changed test fails on broken behaviour and passes on real code | Supplies the evidence the quality standard demands |

## Dependencies

### Prerequisites (from other sequences)

- 03_article_search: the `LikePattern` escaping and `ArticleSearchRepositoryTest`, whose `%` assertion this replaces.
- 05_user_activity: `CommentCreateTest` and the `HttpUtil` raw helpers added during slice 02's testing work.
- 07_submission_docs: `AGENT_WORKLOG.md`'s Deferred test fixes section, which this sequence resolves and must update.

### Provides (to other sequences)

- A raw-JSON test helper that cannot silently double-encode: used by any future test that posts a hand-written JSON body.
- 004_DELIVER records the final state, so it inherits the corrected census and the resolved deferred list.

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| The new missing-field test passes for a third unrelated reason | Med | High | Capture the actual request body sent by the helper; do not accept a 422 as proof on its own |
| Changing `postRaw`'s signature breaks the DTO call sites | Med | Med | Add a new method rather than changing `postRaw`; the 12 DTO call sites keep the object-serializing path |
| The `%` replacement is also non-discriminating | Med | High | Require a recorded failing run against escaping deliberately removed, before the passing run |
| Reopening a judge-approved phase | High | Low | Expected: the phase gate re-runs and must re-approve with this sequence included, which is the correct outcome for new work |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: `HttpUtil` has a `String`-typed raw helper and `missing body returns 422` uses it
- [ ] **Milestone 2**: the `%` test asserts 0 matches where a wildcard reading would match
- [ ] **Milestone 3**: both tests recorded failing on broken behaviour and passing on real code; `AGENT_WORKLOG.md`'s deferred list updated

## Quality Gates

### Testing and Verification

- [ ] All unit tests pass
- [ ] Integration tests complete
- [ ] Performance benchmarks met — not applicable to this sequence; no performance-sensitive code changes

### Code Review

- [ ] Code review conducted
- [ ] Review feedback addressed
- [ ] Standards compliance verified

### Iteration Decision

- [ ] Need another iteration? No — decided once tasks 01-03 pass their gates; revisit only if the recorded failing runs
  show a test still cannot fail
- [ ] If yes, new tasks created: n/a