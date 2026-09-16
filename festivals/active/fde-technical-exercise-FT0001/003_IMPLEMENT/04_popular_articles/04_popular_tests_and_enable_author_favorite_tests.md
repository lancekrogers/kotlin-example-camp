---
fest_type: task
fest_id: 04_popular_tests_and_enable_author_favorite_tests.md
fest_name: popular_tests_and_enable_author_favorite_tests
fest_parent: 04_popular_articles
fest_order: 4
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.790329-06:00
fest_updated: 2026-09-16T02:53:36.843496-06:00
fest_tracking: true
---


# Task: Test Popular and enable the author's favorite tests

## Objective

Pin Popular and favorites with HTTP tests, enable the author's favorite and unfavorite tests, and record the census.

## Requirements

- [ ] `PopularArticlesTest` covers D005:
  - ordering by count, and equal counts ordered by recency
  - consecutive pages that neither overlap nor skip
  - zero-favorite articles included
  - an offset past the end returning an empty list with the correct `articlesCount`
  - a bad `limit` → 422, and anonymous access → 200
  - `favorited` true for the favoriting viewer and false anonymously
  - a double favorite leaving `favoritesCount` unchanged
  - the raw-JSON leak check
- [ ] In `ArticleControllerTest.kt`, the method-level `@Ignore` on `favorite article by slug` and `unfavorite article by slug` is removed. Both pass, with unique usernames where they collide (D010, D009)
- [ ] `just test census` output is recorded in `results/04_census.md`

## Implementation

**Test isolation matters most here.** Articles written by every earlier test are still in the table (D009), so Popular tests must never assert an absolute rank. They assert the *relative* order of their own articles only.

**Steps**

1. **Create** `src/test/kotlin/io/realworld/app/web/controllers/PopularArticlesTest.kt` with `@Rule @JvmField val appRule = AppRule()`.
   - **Helpers:**
     - register UUID-named users, each with its own `HttpUtil(appRule.port)` and token;
     - create articles whose titles contain a per-test token;
     - favorite through each user's client.
   - **Full walk:** request `limit=100` at increasing `offset` until a page comes back empty. Collect the slugs. Fail if this takes more than 50 pages.
   - **Cases:**
     - `more favorites ranks first`: A has 2 favorites, B has 1, C has 0 → relative order A, B, C.
     - `equal counts newest first`: D and E each have 1 favorite, and E was created after D → E comes before D. Sleep 5 ms between the two creates so their `createdAt` values differ.
     - `pages are stable`: the concatenation of single-item pages (`limit=1`, offsets 0..k) equals the first k+1 items of one `limit=k+1` page.
     - `zero favorite included`: an unfavorited article appears in the full walk.
     - `offset past end`: `offset = articlesCount` → empty `articles`, with `articlesCount` unchanged.
     - `bad limit`: `limit=0` → 422.
     - `anonymous is public`: a fresh `HttpUtil(appRule.port)` gets 200.
     - `favorited is viewer specific`: after user X favorites article F, X's request shows `favorited = true` for F, and an anonymous request shows `false`.
     - `double favorite`: POSTing favorite twice returns 200 both times, and `favoritesCount` stays 1.
     - `no author secrets`: the raw body of `getRaw("/articles/feed/popular?limit=5")` passes `assertNoAuthorSecrets`.
2. **`ArticleControllerTest.kt`.**
   - Remove the method-level `@Ignore` that `02_article_foundation` put on `favorite article by slug` (`:190`) and `unfavorite article by slug` (`:210`).
   - The unfavorite test registers username `user_name_test` (`:213`), which collides with `HttpUtil.createUser()`'s default. Give it a unique suffix.
   - Keep its slug expectations. Titles `slug test` and `slug test 2` are unique within the suite.
3. **Run** `just test only PopularArticlesTest` and `just test only ArticleControllerTest`, then `just test all` and `just test census`. Save the output to `results/04_census.md`.

**Error paths**

- **The equal-count case flakes:** both articles got the same `createdAt` millisecond, so the id tie-break decided. Keep the sleep, or assert the documented tie-break (higher id first).
- **Page stability fails intermittently:** another writer inserted rows between requests. JUnit 4 runs test methods sequentially within a class. If Gradle is ever set to fork tests in parallel, record that rather than weakening the test.

## Done When

- [ ] All requirements met
- [ ] `results/04_census.md` shows `PopularArticlesTest` and the author's `favorite article by slug` and `unfavorite article by slug` running and passing