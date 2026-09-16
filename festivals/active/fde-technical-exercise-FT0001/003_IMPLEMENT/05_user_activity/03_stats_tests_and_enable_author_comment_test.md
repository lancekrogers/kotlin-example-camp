---
fest_type: task
fest_id: 03_stats_tests_and_enable_author_comment_test.md
fest_name: stats_tests_and_enable_author_comment_test
fest_parent: 05_user_activity
fest_order: 3
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.850535-06:00
fest_updated: 2026-09-16T03:25:43.218087-06:00
fest_tracking: true
---


# Task: Test User Activity and enable the author's add-comment test

## Objective

Pin User Activity and comment creation with HTTP tests, enable the author's add-comment test, and record the census.

## Requirements

- [ ] `ProfileStatsTest` covers D006:
  - a new user has zero activity
  - each count rises after its action
  - favoriting another user's article raises the favoriter's `favoritesCount` but not the author's
  - a comment on another user's article counts for the commenter
  - an unknown username → 404; an anonymous request → 200
- [ ] In `CommentControllerTest`, the class-level `@Ignore` (`:14`) is replaced by method-level reasons on the list and delete tests, `/api` paths are removed, and `add comment for article by slug` is enabled and passes
- [ ] Comment creation tests: blank body → 422, unknown slug → 404, no token → 401, plus the raw-JSON leak check
- [ ] `just test census` output is recorded in `results/03_census.md`

## Implementation

**Steps**

1. **Create `ProfileStatsTest`** at `src/test/kotlin/io/realworld/app/web/controllers/ProfileStatsTest.kt`, using `AppRule`. Every test registers fresh UUID-named users, so counts start at zero no matter what other tests left behind (D009).
   - `new user has zero activity`
   - `article comment and favorite each count once`: user U creates one article, comments once on another user's article, and favorites another user's article → `1/1/1`.
   - `favorites count what the user gave`: V favorites W's article → V's `favoritesCount` is 1 and W's is 0.
   - `unknown username is 404`: request `/profiles/nobody-<uuid>/stats`.
   - `anonymous request is public`: a fresh `HttpUtil(appRule.port)` gets 200.
2. **Create `CommentCreateTest`** at `src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt`.
   - A blank body → 422.
   - An unknown slug → 404.
   - No token → 401.
   - The `postRaw` comment response passes `assertNoAuthorSecrets`.
3. **Update `CommentControllerTest.kt`.**
   - Remove the class-level `@Ignore` (`:14`).
   - Add `@Ignore("GET /articles/{slug}/comments is still stubbed; out of scope per D001")` to `get all comments for article by slug` (`:35`).
   - Add `@Ignore("DELETE /articles/{slug}/comments/{id} is still stubbed; out of scope per D001")` to `delete comment for article by slug` (`:54`).
   - Replace every `/api/` path.
   - Leave `add comment for article by slug` (`:21`) enabled.
4. **Run** the three classes with `just test only <Class>`, then `just test all` and `just test census`. Save the output to `results/03_census.md`.

**Error paths**

- **Stats counts are higher than expected:** the test reused a username or email from another test. Use UUIDs.
- **`add comment for article by slug` fails with 401:** the default `createArticle()` login failed. See the error path in `02_article_foundation` task 06.

## Done When

- [ ] All requirements met
- [ ] `results/03_census.md` shows `ProfileStatsTest`, `CommentCreateTest` and `add comment for article by slug` running and passing, with the remaining comment tests skipped with reasons