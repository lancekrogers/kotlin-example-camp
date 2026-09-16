---
fest_type: phase
fest_id: 003_IMPLEMENT
fest_name: IMPLEMENT
fest_parent: fde-technical-exercise-FT0001
fest_order: 3
fest_status: completed
fest_created: 2026-09-15T11:32:01.603011-06:00
fest_updated: 2026-09-16T12:55:31.860885-06:00
fest_phase_type: implementation
fest_tracking: true
---


# Phase Goal: 003_IMPLEMENT

**Phase:** 003_IMPLEMENT | **Status:** Pending | **Type:** Implementation

## Phase Objective

**Primary Goal:** CI that runs on JDK 17 and 21, the article foundation, Article Search, Popular Articles, User Activity, the RealWorld spec-test CI job, and the submission docs, each merged into the fork as a green slice.

**Context:** It executes the decisions approved in 002_PLAN (D001-D012) in the order D001 fixes. Every slice ends with the testing, review, iterate and commit gates and a PR that is green on both JDKs (D012). The finished, merged work is what 004_DELIVER records in the walkthrough.

## Required Outcomes

Deliverables this phase must produce:

- [ ] CI runs on the fork on JDK 17 and 21, with SHA-pinned actions, Gradle caching, and JUnit annotations on failures (01_ci_pipeline)
- [ ] `POST /articles` creates articles with tags, a unique slug and a Profile author; the author's create and tags tests pass (02_article_foundation)
- [ ] `GET /articles/search?q=` is public, case-insensitive, matches its term literally, is paged, and returns the total count (03_article_search)
- [ ] `GET /articles/feed/popular` is public, ordered by favorite count then recency, and paged; favorite writes are idempotent (04_popular_articles)
- [ ] `GET /profiles/{username}/stats` returns articles authored, comments written and favorites given (05_user_activity)
- [ ] The bundled RealWorld collection runs in CI and fails only on regressions or a stale manifest (06_spec_api_ci)
- [ ] README API notes and `AGENT_WORKLOG.md` are merged (07_submission_docs)

<!-- Add more required outcomes as needed -->

## Quality Standards

Quality criteria for all work in this phase:

- [ ] All builds and tests run in Docker through the `just` modules, never on the host toolchain (C7)
- [ ] Every endpoint returning an article or comment has a raw-JSON test proving no `password`, `email` or `token` under `author` (D008)
- [ ] Every new test uses uniquely named data and asserts only on rows it created (D009)
- [ ] `just test census` output is recorded in the sequence's `results/` (D010)
- [ ] Commits use `fest commit` with no AI attribution; PRs target `lancekrogers/kotlin-ktor-realworld-example-app` `master` explicitly (D012)

<!-- Add more quality standards as needed -->

## Sequence Alignment

| Sequence | Goal | Key Deliverable |
|----------|------|-----------------|
| 01_ci_pipeline | CI that actually runs, on JDK 17 and 21 | pinned matrix workflow with JUnit annotations |
| 02_article_foundation | articles, tags, slugs, Profile authors | `POST /articles` and enabled author tests |
| 03_article_search | public literal-safe search | `GET /articles/search` |
| 04_popular_articles | favorites and a popular feed | `GET /articles/feed/popular` |
| 05_user_activity | comments and profile stats | `GET /profiles/{username}/stats` |
| 06_spec_api_ci | spec tests in CI | `spec` job with expected-failures manifest |
| 07_submission_docs | grader-facing docs | README API notes and `AGENT_WORKLOG.md` |

<!-- Add rows as sequences are created -->

## Pre-Phase Checklist

Before starting implementation:

- [ ] Planning phase complete
- [ ] Architecture/design decisions documented
- [ ] Dependencies resolved
- [ ] Development environment ready

## Phase Progress

### Sequence Completion

- [ ] 01_ci_pipeline
- [ ] 02_article_foundation
- [ ] 03_article_search
- [ ] 04_popular_articles
- [ ] 05_user_activity
- [ ] 06_spec_api_ci
- [ ] 07_submission_docs

<!-- Track sequence completion here -->

## Notes

Working directory for every sequence: `projects/kotlin-ktor-realworld-example-app`. The toolchain stays fixed: Ktor 1.2.3, Exposed 0.41.1, Kotlin 1.9.25, Gradle 8.14, JVM 17 (C4). H2 is in-memory with `DB_CLOSE_DELAY=-1`, so rows persist across test methods (D009). Public reads must be registered before the mandatory `authenticate` block (D003). Pushes, PRs and merges are outward-facing and need the user's authorization. Unverified until their tasks run: LOWER on H2 CLOB (03/01), the H2 GROUP BY form (04/03), row persistence across tests (02/06), and the zero-run cause (01/01).

---

*Implementation phases use numbered sequences. Create sequences with `fest create sequence`.*