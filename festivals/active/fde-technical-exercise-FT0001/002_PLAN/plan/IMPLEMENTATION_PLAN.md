# Implementation Plan

## Overview

Seven implementation slices, then a delivery phase. The feature slices build the article data layer the
codebase never had, but only the parts the three features read. Every slice ends with the testing, review,
iterate and `fest commit` gates, and a PR that is green on JDK 17 and 21 (D001, D012). If time runs out,
the submission is the last merged slice.

All work happens in `projects/kotlin-ktor-realworld-example-app`. Builds and tests run in Docker through the
existing `just` modules, never on the host toolchain (C7). Decisions are cited as D###. Requirements (R#) and
constraints (C#) come from `001_INGEST/output_specs/`.

**Two rules apply to every task:**

- **Test isolation (D009).** Rows persist across test methods in one JVM. Tests create uniquely named data
  and assert only on what they created.
- **No password leaks (D008).** Any endpoint that returns an article or a comment gets a raw-JSON test
  showing `password`, `email` and `token` never appear under `author`.

## Phase Breakdown

### Phase 003: IMPLEMENT

**Type:** implementation
**Goal:** Deliver CI, the article foundation, the three named features, and the spec-test job, each as a
merged, green slice.

#### Sequences

1. **01_ci_pipeline** (R3, R10; D011, D012)
   - **Goal:** CI that actually runs on the fork, covers JDK 17 and 21, caches Gradle, and makes failures
     readable.
   - **Tasks:**
     - [ ] 01_diagnose_zero_ci_runs: find why the fork has 0 runs (per-fork opt-in hypothesis), get a run
       to trigger, and record the real cause.
     - [ ] 02_replace_workflow_with_jdk_matrix: rewrite `.github/workflows/gradle.yml` with a 17/21 matrix
       and `fail-fast: false`. Use setup-java v6.0.1, setup-gradle v6.3.0, checkout v7.0.1 and
       action-junit-report v6.5.0, all pinned by SHA, plus `workflow_dispatch`, least-privilege
       permissions, and a report artifact on failure.
     - [ ] 03_prove_ci_fails_red: a throwaway branch with one failing test must turn the check red with an
       annotation. Record the evidence, then delete the branch.

2. **02_article_foundation** (R14, R2, R12; D002, D008, D009, D010)
   - **Goal:** articles can be created with tags and a unique slug, authors are served as Profiles, and
     the author's create and tags tests run.
   - **Tasks:**
     - [ ] 01_authors_as_profiles: change `Article.author` and `Comment.author` to `Profile?`, and add the
       raw-JSON leak assertion helper.
     - [ ] 02_articles_and_article_tags_schema: add an `Articles` table (unique slug, author reference,
       timestamps) and an `ArticleTags` join table, and give `TagRepository` get-or-create by name.
     - [ ] 03_slug_generation: a slug function per D009 (normalize, kebab-case, reserved words, numeric
       suffix), with unit tests.
     - [ ] 04_article_repository_and_service_create: `ArticleRepository` create, findBySlug and mapping
       (Profile author, tags); `ArticleService.create` with 422 validation; Kodein bindings.
     - [ ] 05_wire_create_article_endpoint: `ArticleController.create` responds `ArticleDTO` through
       `ctx.respond`, on the existing mandatory-auth POST route.
     - [ ] 06_enable_author_create_and_tag_tests: enable `create article` and `get all tags` with root
       paths and isolated data, add the method-level `@Ignore` reasons, and record census output.

3. **03_article_search** (R1, R2, R13, R15; D003, D004, D007)
   - **Goal:** `GET /articles/search?q=` is public and case-insensitive, treats its term literally, is
     paged, and returns total counts.
   - **Tasks:**
     - [ ] 01_prove_lower_on_clob_body: first, a failing-then-passing test proving `lower(body) LIKE` works
       on H2's TEXT column. If it does not, stop and amend D004.
     - [ ] 02_search_repository_query: a title OR body match using `lowerCase()` and
       `LikePattern.ofLiteral`, ordered newest first, plus a total-count query.
     - [ ] 03_search_service_validation_and_paging: `q` required and non-blank, `limit` 1..100 (default 20),
       `offset` ≥ 0 (default 0); violations are 422 through `require`.
     - [ ] 04_public_search_route: register it in `authenticate(optional = true)` **before** the mandatory
       block in `Routing.articles`, with a comment citing D003.
     - [ ] 05_search_endpoint_tests: every D004 edge case, the anonymous-200 route-order pin, and the leak
       test.

4. **04_popular_articles** (R1, R2, R13, R15; D003, D005, D007, D010)
   - **Goal:** favorites work idempotently, article responses carry real favorite fields, and
     `GET /articles/feed/popular` is ordered deterministically and paged.
   - **Tasks:**
     - [ ] 01_favorites_schema_and_repository: an `ArticleFavorites` table keyed on (user, article); favorite
       and unfavorite are no-ops when already in the target state; count and viewer-favorited queries.
     - [ ] 02_favorite_endpoints_and_real_counts: wire favorite and unfavorite (404 on an unknown slug), and
       fill `favorited` and `favoritesCount` in every article response.
     - [ ] 03_popular_query_service_and_route: order by count, then createdAt, then id; paging per D004
       bounds; a GROUP BY form H2 accepts; a public route registered before the mandatory block.
     - [ ] 04_popular_tests_and_enable_author_favorite_tests: ordering, tie-breaks, page continuity, zero
       favorites, the viewer-dependent `favorited` value, double-favorite, and enabling the two author
       favorite tests.

5. **05_user_activity** (R1, R2, R13, R15; D003, D006, D008, D010)
   - **Goal:** comments can be added, and `GET /profiles/{username}/stats` reports the user's own activity.
   - **Tasks:**
     - [ ] 01_comments_schema_and_add_comment: a `Comments` table, `CommentRepository` and `CommentService`
       (422 on a blank body, 404 on an unknown slug); `CommentController.add` responds `CommentDTO` with a
       Profile author; Kodein bindings.
     - [ ] 02_profile_stats_service_and_route: `ProfileStats` and its DTO, a `ProfileStatsService` (articles
       authored, comments written, favorites given), `ProfileController.stats`, a public route inside the
       existing optional-auth block, and 404 for an unknown user.
     - [ ] 03_stats_tests_and_enable_author_comment_test: zeros for a new user, each count moving on its
       action, favorites given vs received, 404, anonymous 200, and enabling the author's add-comment test.

6. **06_spec_api_ci** (R9; D011)
   - **Goal:** the bundled RealWorld collection runs in CI against a live container, and fails only on
     regressions or a stale manifest.
   - **Tasks:**
     - [ ] 01_pin_newman_and_guard_api_url: pin `newman@6.2.2` in `run-api-tests.sh` and refuse to run
       without an explicit `APIURL`.
     - [ ] 02_expected_failures_manifest_and_compare_script: `spec-api/expected-failures.txt` from a real
       local run, plus a compare script that fails on unexpected failures and unexpected passes.
     - [ ] 03_spec_job_in_workflow: a `spec` job that builds the image, starts the app with a generated
       `JWT_SECRET`, waits for health, then runs newman and the compare script.

7. **07_submission_docs** (R4, R7, R11; D001, D002, D006, D007)
   - **Goal:** a grader can understand the scope, the API paths, the counts, and how agents were used and
     verified.
   - **Tasks:**
     - [ ] 01_readme_api_notes: base-path mapping from `/api/...` to root, the three endpoints and their
       parameters, what `articlesCount` and each stat mean, and how to run tests, the JDK matrix and spec
       tests locally.
     - [ ] 02_agent_worklog: `AGENT_WORKLOG.md` covering all six brief §5 points, drawn from
       `001_INGEST/input_specs/agent-usage-record.md` and each sequence's `results/`. It must include real
       mistakes and the reasoning for shipping all three features.
     - [ ] 03_commit_and_open_docs_pr: this sequence has no gates, so commit it with `fest commit`, open the
       PR with a pinned repo and base, and merge when green.

### Phase 004: DELIVER

**Type:** non_coding_action
**Goal:** The submission is complete and reachable by graders.
**Action items:** record the walkthrough (human); add its link to `AGENT_WORKLOG.md` and merge; verify the
repository and the recording from a logged-out session; run `camp refs-sync` to update the submodule
pointer; record the final state.

## Dependencies

| Item | Blocked By | Rationale |
|------|------------|-----------|
| 02_article_foundation | 01_ci_pipeline | Feature PRs need visible CI on both JDKs (D001, D012). |
| 03_article_search | 02_article_foundation | Search reads articles, tags and authors created there. |
| 04_popular_articles | 02_article_foundation | Favorites reference articles; D001 also orders it after 03. |
| 05_user_activity | 04_popular_articles | `favoritesCount` counts favorites given, which only exist after 04 (D006). |
| 06_spec_api_ci | 05_user_activity, 01_ci_pipeline | The expected-failures manifest must reflect the final endpoints, and the job extends the workflow. |
| 07_submission_docs | 06_spec_api_ci | The docs and work log describe the finished state. |
| 004_DELIVER | 003_IMPLEMENT | The recording shows the finished work. |
| INGEST gate step 3, PLAN gate step 2 | the user | `operator_attestation` checkpoints cannot be judged. |

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| The fork's workflows stay disabled, so CI never runs | High | High | 01_diagnose_zero_ci_runs goes first; enabling the per-fork opt-in is a human click, then proven with a dispatch run. |
| `LOWER` on H2's TEXT/CLOB body fails | Medium | Medium | 03/01 proves it before any search code; if it fails, stop and amend D004. |
| H2 rejects the popular GROUP BY form | Medium | Low | Use every selected column or a count subquery; a test proves it (D005). |
| Route registration order regresses and search goes behind auth | Low | High | Anonymous-200 pinning test plus a comment at the registration point (D003). |
| Data left by earlier tests breaks counts, slugs or usernames | High | Medium | D009 isolation rule; tests assert only on their own rows. |
| Author responses leak password hashes | High if unaddressed | High | Profile authors (D008) and raw-JSON leak tests in every article or comment slice. |
| Three features overrun the time guidance | High | Medium | Ordered slices and the per-slice cutoff (D001); submission is the last merged slice. |
| Wrong pinned action SHA, e.g. an annotated tag not dereferenced | Low | Medium | Resolve through the git refs API with dereferencing, and confirm with a real run. |
| Unpinned `npx newman`, or the runner hitting a remote host | Medium | Medium | Pin 6.2.2 and require an explicit `APIURL` (06/01). |
| `operator_attestation` gates block phase completion | High | Low | Hand both attestations to the user with exact commands; nothing else waits on them. |

## Deferred (recorded, not planned)

- The `unfollow` row-orientation bug (R8, P2). It is unreachable because no feature wires follow/unfollow.
  Record it in `AGENT_WORKLOG.md`.
- The personal feed, the article list endpoint, get-by-slug, update, delete, comment list/delete, and
  profile get/follow/unfollow. These stay stubbed, and their author tests carry `@Ignore` reasons (D010).
