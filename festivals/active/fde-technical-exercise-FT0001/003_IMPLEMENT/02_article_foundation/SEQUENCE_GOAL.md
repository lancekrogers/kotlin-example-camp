---
fest_type: sequence
fest_id: 02_article_foundation
fest_name: article_foundation
fest_parent: 003_IMPLEMENT
fest_order: 2
fest_status: pending
fest_created: 2026-09-15T11:54:21.375987-06:00
fest_tracking: true
---


# Sequence Goal: 02_article_foundation

**Sequence:** 02_article_foundation | **Phase:** 003_IMPLEMENT | **Status:** Pending | **Created:** 2026-09-15T11:54:21-06:00

## Sequence Objective

**Primary Goal:** Articles can be created with tags and a unique slug, authors are returned as Profiles, and the author's create-article and tags tests run.

**Contribution to Phase Goal:** All three named features read articles. This is the shared foundation (R14) that Search, Popular and User Activity build on.

## Success Criteria

The sequence goal is achieved when:

### Required Deliverables

- [ ] **Article schema**: `Articles` and `ArticleTags` tables with a unique slug index and an author reference, created idempotently
- [ ] **Create endpoint**: `POST /articles` returns 200 with the created article (Profile author, tags, ISO-8601 dates); 401 without a token; 422 on invalid input
- [ ] **Enabled author tests**: `create article` and `get all tags` run and pass; the remaining ignored tests carry reasons; census recorded

### Quality Standards

- [ ] **No author data leaks**: a raw-JSON test proves no `password`, `email` or `token` appears under `author` (D008)
- [ ] **Isolated tests**: every new test uses unique users, titles and slugs, and asserts only on its own rows (D009)

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
| 01_authors_as_profiles | Make article and comment authors Profiles | Change `Article.author` and `Comment.author` from `User?` to `Profile?`, and add a raw-JSON assertion that proves no author secrets reach a response. |
| 02_articles_and_article_tags_schema | Add the Articles and ArticleTags schema | Add the `Articles` and `ArticleTags` tables and a tag get-or-create helper, created idempotently in dependency order. |
| 03_slug_generation | Implement slug generation | Implement D009's slug rule as a pure function plus a uniqueness helper, with unit tests for every case. |
| 04_article_repository_and_service_create | Add article create to the repository and service | Give `ArticleRepository` create, find-by-slug and article mapping, and add `ArticleService.create` with validation, wired through Kodein. |
| 05_wire_create_article_endpoint | Wire POST /articles | Make `POST /articles` create an article for the signed-in user and respond with `ArticleDTO`, turning malformed bodies into 422 and dates into ISO-8601 strings. |
| 06_enable_author_create_and_tag_tests | Enable the author's create and tags tests | Enable the author's `create article` and `get all tags` tests, add the missing negative and leak tests, and record what the suite now actually runs. |

## Dependencies

### Prerequisites (from other sequences)

- 01_ci_pipeline: the JDK 17/21 check that gates this slice's PR

### Provides (to other sequences)

- articles, tags, slugs and the author-as-Profile mapping: Used by 03_article_search, 04_popular_articles, 05_user_activity

## Working Directory

Target project: `projects/kotlin-ktor-realworld-example-app` (relative to campaign root)

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Tables that reference `Tags` and `Users` get created before those repositories exist | Medium | Medium | create `Users`, `Tags`, `Articles` and `ArticleTags` in one `SchemaUtils.create` call, which sorts by references and skips existing tables |

## Progress Tracking

### Milestones

- [ ] **Milestone 1**: authors are Profiles and the leak helper exists
- [ ] **Milestone 2**: create endpoint answers 200 with tags and slug
- [ ] **Milestone 3**: author tests enabled and the PR merged green

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