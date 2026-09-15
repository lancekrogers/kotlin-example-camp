# D007: `articlesCount` is the total number of matching articles

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R15, D004, D005

## Context

The article-list response is `ArticlesDTO(articles, articlesCount)` (`Article.kt:7`). The brief tells
Search to reuse "the existing article-list response format", but the codebase gives `articlesCount` two
different meanings:

- **Page size.** The author's commented-out handler returns `ArticlesDTO(articles, articles.size)`
  (`ArticleController.kt:18`), and every list test in `ArticleControllerTest` asserts
  `articles.size == articlesCount` (first at `ArticleControllerTest.kt:32`).
- **Total matches.** In the RealWorld API specification, `articlesCount` is the total count a client uses
  to build pagination.

## Options

### Option A: Page size
- **Pros:** matches the author's commented code.
- **Cons:** tells a client nothing it can't already count from the array, so a paging UI has no way to
  find the last page.

### Option B: Total matches, ignoring `limit` and `offset`
- **Pros:** matches the RealWorld spec and makes `limit`/`offset` usable.
- **Cons:** one extra `COUNT` query per list request.

## Decision

**Option B.**

## Consequences

- The author's list tests assert `size == articlesCount` only in cases where the total fits in one page.
  They stay compatible as long as each test controls its own data. D009 makes that true.
- New tests for Search and Popular include one where `limit` is smaller than the total. Those tests
  assert that `articles.size` is smaller than `articlesCount`.
- The README states the meaning once.
