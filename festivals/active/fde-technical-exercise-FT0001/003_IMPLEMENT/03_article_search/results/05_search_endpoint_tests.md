# Task 05: Search endpoint tests — Results

## Changes

- `src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt` — new HTTP integration tests pinning search contract (D004 edge cases, D003 public route, D008 author leak check, D009 isolation).

### git diff --stat

```
(no tracked changes; new untracked file)
```

### git status --short --untracked-files=all

```
?? src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt
```

## Commands

### `just test only ArticleSearchTest`

Exit code: **0**

```
> Task :test

ArticleSearchTest > no author secrets PASSED

ArticleSearchTest > bad paging PASSED

ArticleSearchTest > no match PASSED

ArticleSearchTest > case insensitive PASSED

ArticleSearchTest > anonymous request is public PASSED

ArticleSearchTest > blank q PASSED

ArticleSearchTest > limit below total PASSED

ArticleSearchTest > missing q PASSED

ArticleSearchTest > title only match PASSED

ArticleSearchTest > body only match PASSED

ArticleSearchTest > percent and underscore are literal PASSED

BUILD SUCCESSFUL in 16s
```

XML: `build/test-results/test/TEST-io.realworld.app.web.controllers.ArticleSearchTest.xml` — `tests="11" skipped="0" failures="0" errors="0"`.

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 31s
```

Full suite: 60 ran, 60 passed, 0 failed, 19 skipped (from `just test census` after full run).

### `just test census`

Exit code: **0**

```
  PagingTest                 ran=7   passed=7   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=60 passed=60 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

## Done When

- [x] **All requirements met** — pass: `ArticleSearchTest` covers all listed cases with `AppRule`, UUID tokens, `getRaw` for errors, `assertNoAuthorSecrets` on raw JSON, anonymous `HttpUtil` client.
- [x] **`just test only ArticleSearchTest` passes every listed case** — pass: 11/11 passed, 0 skipped (XML evidence above).
- [x] **`results/05_census.md` shows the class ran with nothing skipped** — pass: `ArticleSearchTest ran=11 passed=11 failed=0 skipped=0`.

## Notes

- **Anchor drift:** none. `UserControllerTest.kt:14-16`, `ArticleControllerTest.kt:27`, and `HttpUtil.kt:37-41` matched the task description.
- **Multi-article tests:** `limit below total` and `percent and underscore are literal` register one user then POST multiple articles (second `createWith` with the same token would fail on duplicate username per D009).
- **Blank/missing q tests:** use `getRaw` per task; blank q uses `URLEncoder.encode("   ", "UTF-8")` because `getRaw` has no `queryString` helper.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `f58258d9-9d30-4257-9a75-323c03106975`, 08:14:58Z to 08:19:57Z).

**All 11 cases executed fresh with the build cache disabled:**

```text
$ just build gradle "cleanTest test --no-build-cache"
suite_exit=0
  case: title only match ok                     case: no match ok
  case: body only match ok                      case: blank q ok
  case: case insensitive ok                     case: missing q ok
  case: percent and underscore are literal ok   case: bad paging ok
  case: limit below total ok                    case: anonymous request is public ok
  case: no author secrets ok
  TOTAL tests=79 ran=60 failed=0 skipped=19
```

Reported counts (60 ran, 19 skipped) match this measurement.

**The assertions were read, not just the pass/fail.** The two cases that guard regressions are genuine:

- **`anonymous request is public`** builds a fresh `HttpUtil(appRule.port)` with no token header and asserts 200 with a
  non-null `articles`. If the public block ever slips below the mandatory `authenticate` block, this fails with 401.
  That is D003's regression guard at the HTTP level, complementing the orchestrator's container check in task 04.
- **`percent and underscore are literal`** creates `"<token>100% pure_x"`, finds it by searching `"<token>100%"` with an
  exact count of 1, then creates `"<token>1000 purex"` and asserts the `"<token>100_"` search returns no article whose
  title equals the second one. If `_` were treated as a wildcard, `100_` would match `1000` and the assertion fails.
- **`limit below total`** asserts `articles.size == 2` with `articlesCount == 3`, which is D007 at the wire.
- **`bad paging`** covers `limit=0`, `limit=101`, `limit=abc` and `offset=-1`, each 422, using `getRaw` because typed
  deserialization cannot parse an error body.
- **`no author secrets`** runs `assertNoAuthorSecrets` on the raw search JSON (D008).
- **Isolation (D009).** Every case derives a UUID token, registers `user_<token>` with its own email, and searches for
  that token, so no case can see another's rows in the shared in-memory database.

**The subagent hit and fixed a real isolation bug of its own making.** Its first version called `createWith(token, …)`
twice in one test, which re-registered the same username and would have failed. It restructured to `loginAs(token)`
once followed by two `createArticle` calls. Recorded because this is exactly the class of false pass D009 exists to
prevent.

### Observations routed to the iterate gate (gate 08)

1. **Dead statements.** `blank q` and `missing q` each contain `UUID.randomUUID().toString().take(8)` as a discarded
   expression with a comment about isolation, but neither test uses the value. Those two cases need no isolation token,
   since they assert only a status code. Remove both lines.
2. **Loose assertions where exact ones are available.** `body only match` and `case insensitive` assert
   `articlesCount >= 1` with `articles.any { … }`. Their tokens are unique per test, so both can assert exactly 1, the
   way `title only match` does. As written they would still pass if the query started returning extra rows.
3. **The underscore case could assert a total.** It asserts only that the second title is absent. The repository-level
   test in task 02 already asserts exactly 0 matches for the same pattern, so behavior is covered; tightening this one
   to `articlesCount == 0` would make the HTTP test self-contained.

None of these is a defect in the shipped behavior: they are test-strength improvements, and gate 08 is where this
sequence's findings are addressed.
