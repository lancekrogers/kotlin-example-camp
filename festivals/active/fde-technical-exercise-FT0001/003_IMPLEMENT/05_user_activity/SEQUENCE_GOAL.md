---
fest_type: sequence
fest_id: 05_user_activity
fest_name: user_activity
fest_parent: 003_IMPLEMENT
fest_order: 5
fest_status: completed
fest_created: 2026-09-15T11:54:21.451066-06:00
fest_updated: 2026-09-16T12:14:48.849291-06:00
fest_tracking: true
---



# Sequence Goal: 05_user_activity

**Sequence:** 05_user_activity | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** Comments can be added, and `GET /profiles/{username}/stats` reports articles authored, comments written, and favorites given.

**Contribution to Phase Goal:** Ships the brief's User Activity (R1) with the count meanings fixed in D006.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **Comments**: a `Comments` table and `POST /articles/{slug}/comments` returning a comment with a Profile author; 422 on a blank body; 404 on an unknown slug
- [ ] **Stats endpoint**: public `GET /profiles/{username}/stats` with three counts; 404 for an unknown user; zeros for a new user
- [ ] **Enabled author test**: `add comment for article by slug` runs and passes

### Quality Standards

- [ ] **Counts mean activity**: favoriting another user's article raises only the favoriter's `favoritesCount`, proven by test
- [ ] **No author data leaks**: a raw-JSON leak test on the comment response (D008)

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
| 01_comments_schema_and_add_comment | Add comments and POST /articles/{slug}/comments | Add the `Comments` table and make `POST /articles/{slug}/comments` create a comment authored by the signed-in user. |
| 02_profile_stats_service_and_route | Add GET /profiles/{username}/stats | Add `GET /profiles/{username}/stats`, returning how many articles the user authored, comments they wrote, and favorites they gave. |
| 03_stats_tests_and_enable_author_comment_test | Test User Activity and enable the author's add-comment test | Pin User Activity and comment creation with HTTP tests, enable the author's add-comment test, and record the census. |

## Dependencies

### Prerequisites (from other sequences)

- 04_popular_articles: favorites, which `favoritesCount` counts

### Provides (to other sequences)

- the final set of implemented endpoints: Used by 06_spec_api_ci

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Stats counts pick up rows left behind by earlier tests | High | High | each test creates a fresh, uniquely named user and asserts only that user's counts (D009) |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: comments can be added
- [ ] **Milestone 2**: stats endpoint answers anonymously
- [ ] **Milestone 3**: tests green and the PR merged

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