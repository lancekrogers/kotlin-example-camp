---
fest_type: sequence
fest_id: 03_article_search
fest_name: article_search
fest_parent: 003_IMPLEMENT
fest_order: 3
fest_status: pending
fest_created: 2026-09-15T11:54:21.401189-06:00
fest_tracking: true
---


# Sequence Goal: 03_article_search

**Sequence:** 03_article_search | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** `GET /articles/search?q=` is public, case-insensitive, treats its term literally, is paged, and returns the total match count.

**Contribution to Phase Goal:** Ships the brief's Article Search (R1) with the semantics fixed in D004 and D007.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **LOWER-on-CLOB proof**: a test proving `lower(body) LIKE` works on H2's TEXT column, run before any search code
- [ ] **Search endpoint**: a public route registered before the mandatory auth block, returning `ArticlesDTO` newest first
- [ ] **Edge-case tests**: every D004 case, plus the anonymous-200 route-order pin and the leak test

### Quality Standards

- [ ] **Literal matching**: `%` and `_` in `q` match themselves, proven by test
- [ ] **Route order pinned**: an anonymous request returns 200 with an `articles` list, and the test fails if the route moves after the auth block (D003)

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
| 01_prove_lower_on_clob_body | Prove LOWER works on the text body column | Before any search code exists, prove with a test that `lower(body) LIKE` with an escaped pattern works on H2's TEXT body column. |
| 02_search_repository_query | Add the search query to ArticleRepository | Add `ArticleRepository.search`, returning one page of articles whose title or body contains the term, plus the total match count. |
| 03_search_service_validation_and_paging | Validate search input and paging | Add `ArticleService.search` and a shared `Paging` parser that turn raw query parameters into validated inputs, rejecting bad ones with 422. |
| 04_public_search_route | Expose GET /articles/search publicly | Expose `GET /articles/search` as a public read, registered before the mandatory auth block, responding with `ArticlesDTO`. |
| 05_search_endpoint_tests | Test the search endpoint contract | Pin Search's contract with HTTP tests covering every D004 edge case, the D003 route order and the D008 leak check, and record the census. |

## Dependencies

### Prerequisites (from other sequences)

- 02_article_foundation: articles with tags, Profile authors, and the article mapping helper

### Provides (to other sequences)

- the shared paging helper and the public-read route block: Used by 04_popular_articles

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| H2 rejects LOWER on a CLOB column | Medium | Medium | task 01 proves it first; on failure, stop and amend D004 before changing the schema or query |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: LOWER on CLOB proven
- [ ] **Milestone 2**: search endpoint answers anonymously
- [ ] **Milestone 3**: edge-case suite green and the PR merged

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