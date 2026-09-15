# D009: Slug generation, collisions, and test data that persists across tests

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R14, R2, D003, D004, D005, D010

## Context

Articles are addressed by slug, and the author's tests fix the rule's basic shape:

- Title `slug test` → slug `slug-test` (`ArticleControllerTest.kt:190`, `:198`).
- Title `slug test 2` → `slug-test-2` (`:210`, `:222`).

Two facts make collisions a normal case rather than a rare one:

1. **The helper reuses one title.** `HttpUtil.createArticle()` always creates "How to train your dragon"
   (`HttpUtil.kt:76`), and many tests call it.
2. **Rows outlive the test that wrote them.** The database URL uses `DB_CLOSE_DELAY=-1`
   (`AppConfig.kt:44`). Per H2's documented semantics, that keeps the in-memory database alive for
   the lifetime of the JVM. `AppRule` builds a fresh app for every test method (`AppRule.kt:13`),
   but each app reconnects to the *same* database. So rows written by one test method remain visible to
   later methods in the same Gradle test JVM. This is a static reading; the first test task confirms it.

D003 also makes the constant path segments `search` and `feed` shadow `GET /articles/{slug}` for articles
whose slug is exactly that word.

## Options

### Option A: Deterministic slug, numeric suffix on collision
- **Pros:** matches the author's tests; readable and predictable.
- **Cons:** needs an existence check before insert.

### Option B: Always append a random suffix
- **Pros:** no collision check.
- **Cons:** breaks `slug-test` and `slug-test-2`, and makes slugs unpredictable for clients and tests.

### Option C: Reject duplicate titles
- **Pros:** trivial.
- **Cons:** a title is not an identifier. Unrelated users could no longer reuse a common title.

## Decision

**Option A.**

1. **Normalize** the title with NFD and drop combining marks, so `café` becomes `cafe`.
2. **Convert to kebab-case:** lowercase, replace every run of characters outside `[a-z0-9]` with `-`, and
   trim leading and trailing `-`.
3. **Fall back** to `article` if the result is empty.
4. **Reserve** `search` and `feed`, which D003 shadows. Either one is treated as taken.
5. **On collision**, append `-2`, `-3`, and so on until the slug is free.

The existence check and the insert run in one transaction, and a unique index on `slug` backs it.
Concurrent creates of the same title in this single-process, in-memory app can still race into a
unique-index violation. That residual risk is documented, not engineered away.

The slug is fixed at create time. Update is out of scope.

## Consequences

- **Unit tests for the slug function:**
  - basic kebab case;
  - punctuation runs;
  - diacritics;
  - an all-symbol title;
  - both reserved words;
  - successive collisions.
- **Isolation rule for every new test in this festival.**
  - **Unique data:** each test creates users, emails, titles, and body text with a unique suffix.
  - **Own rows only:** each test asserts only on rows it created, and never assumes an empty database or
    an absolute count or rank.
  - **Enabled author tests:** these may be adjusted to follow the same rule (D010), as long as none of
    their assertions is weakened.
