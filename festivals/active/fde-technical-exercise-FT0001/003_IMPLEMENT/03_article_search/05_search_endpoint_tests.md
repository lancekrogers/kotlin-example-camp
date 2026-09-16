---
fest_type: task
fest_id: 05_search_endpoint_tests.md
fest_name: search_endpoint_tests
fest_parent: 03_article_search
fest_order: 5
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.709523-06:00
fest_updated: 2026-09-16T02:21:50.536367-06:00
fest_tracking: true
---


# Task: Test the search endpoint contract

## Objective

Pin Search's contract with HTTP tests covering every D004 edge case, the D003 route order and the D008 leak check, and record the census.

## Requirements

- [ ] One test class, `ArticleSearchTest`, uses `AppRule` and a per-test UUID token, and every test asserts only on articles it created (D009).
- [ ] Cases covered:
  - title-only match; body-only match; case-insensitivity
  - no match → empty list with `articlesCount` 0
  - blank `q` and missing `q` → 422
  - `%` and `_` treated as literals
  - `limit` below the total → page smaller than `articlesCount`
  - bad `limit`/`offset` → 422
  - anonymous request → 200 with `articles`
  - raw JSON passes `assertNoAuthorSecrets`
- [ ] `just test census` output recorded in `results/05_census.md`

## Implementation

**Steps**

1. Create `src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt` with `@Rule @JvmField val appRule = AppRule()`, as `UserControllerTest.kt:14` does.
2. Add helpers inside the class:
   - `createWith(title, body)`: register and log in a UUID user through `appRule.http.registerUser` and `loginAndSetTokenHeader`, then POST `/articles` and assert 200.
   - **Search** through `appRule.http.get<ArticlesDTO>("/articles/search", mapOf("q" to term))`. Parameters go through Unirest's `queryString` (`HttpUtil.kt:37`), so `%` and `_` are URL-encoded correctly. Never concatenate `q` into the path.
   - **Anonymous client:** a fresh `HttpUtil(appRule.port)` with no token header, the pattern at `ArticleControllerTest.kt:27`.
3. Write the cases. Each starts with `val token = UUID.randomUUID().toString().take(8)`.
   - `title only match`: title contains the token, body does not → one result, `articlesCount == 1`.
   - `body only match`: body contains the token → found.
   - `case insensitive`: store the token uppercased, search lowercased.
   - `no match`: search `"nomatch-$token"` → `articles` is empty and `articlesCount == 0`.
   - `blank q` (`"   "`) and `missing q` → 422 each.
   - `percent and underscore are literal`: title `"${token}100% pure_x"`, search `"${token}100%"` → 1 result. Then create `"${token}1000 purex"` and search `"${token}100_"`; it must not match the second title.
   - `limit below total`: three articles containing the token, `limit=2` → `articles.size == 2` and `articlesCount == 3`.
   - `bad paging`: `limit=0`, `limit=101`, `limit=abc` and `offset=-1` → 422 each.
   - `anonymous request is public`: the anonymous client gets 200 and a non-null `articles`. This fails if the route regresses behind auth (D003).
   - `no author secrets`: `appRule.http.getRaw("/articles/search?q=$token")` (the token is URL-safe) → `assertNoAuthorSecrets(response.body)`.
4. Run `just test only ArticleSearchTest`, then `just test all` and `just test census`. Save the outputs to `results/05_census.md`.

**Error paths**

- **The literal-character case matches too much:** the repository built the pattern without `LikePattern.ofLiteral`.
- **Tests pass alone but fail in the full suite:** a test searched a term that is not unique, and picked up another test's rows.

## Done When

- [ ] All requirements met
- [ ] `just test only ArticleSearchTest` passes every listed case, and `results/05_census.md` shows the class ran with nothing skipped