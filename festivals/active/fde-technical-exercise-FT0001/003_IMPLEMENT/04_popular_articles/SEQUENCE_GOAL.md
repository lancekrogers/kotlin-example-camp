---
fest_type: sequence
fest_id: 04_popular_articles
fest_name: popular_articles
fest_parent: 003_IMPLEMENT
fest_order: 4
fest_status: completed
fest_created: 2026-09-15T11:54:21.427879-06:00
fest_updated: 2026-09-16T12:14:48.824324-06:00
fest_tracking: true
---



# Sequence Goal: 04_popular_articles

**Sequence:** 04_popular_articles | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** Favorites work idempotently, article responses carry real favorite fields, and `GET /articles/feed/popular` is deterministically ordered and paged.

**Contribution to Phase Goal:** Ships the brief's Popular Articles (R1) under D005, and makes `favorited` and `favoritesCount` real everywhere, including Search results.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **Favorites**: an `ArticleFavorites` table; favorite and unfavorite are no-ops when repeated; 404 on an unknown slug
- [ ] **Popular endpoint**: public, ordered by favorite count, then createdAt, then id, and paged with the total count
- [ ] **Enabled author tests**: `favorite article by slug` and `unfavorite article by slug` run and pass

### Quality Standards

- [ ] **Stable paging**: two consecutive pages never overlap or skip, proven by test
- [ ] **Viewer-aware fields**: `favorited` is true only for the favoriting viewer, and false for anonymous requests

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
| 01_favorites_schema_and_repository | Add favorites storage with idempotent writes | Add the `ArticleFavorites` table, idempotent favorite and unfavorite repository operations, and the favorite count and viewer lookups that every article response needs. |
| 02_favorite_endpoints_and_real_counts | Wire the favorite and unfavorite endpoints | Wire `POST` and `DELETE /articles/{slug}/favorite` to the repository through `ArticleService`, responding with the article's current state. |
| 03_popular_query_service_and_route | Add GET /articles/feed/popular | Add `GET /articles/feed/popular`: every article, ordered by favorite count, then `createdAt`, then `id`, paged, public, and returning the total count. |
| 04_popular_tests_and_enable_author_favorite_tests | Test Popular and enable the author's favorite tests | Pin Popular and favorites with HTTP tests, enable the author's favorite and unfavorite tests, and record the census. |

## Dependencies

### Prerequisites (from other sequences)

- 03_article_search: the paging helper and the public-read route block

### Provides (to other sequences)

- favorites data and favorite counts: Used by 05_user_activity (favorites given)

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| H2 rejects the grouped popular query | Medium | Medium | select ids and counts grouped by the selected columns only, then load the articles; prove it with a test |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: favorites idempotent
- [ ] **Milestone 2**: popular endpoint ordered and paged
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