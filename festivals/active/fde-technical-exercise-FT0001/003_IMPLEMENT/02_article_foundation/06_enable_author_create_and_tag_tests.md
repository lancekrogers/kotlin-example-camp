---
fest_type: task
fest_id: 06_enable_author_create_and_tag_tests.md
fest_name: enable_author_create_and_tag_tests
fest_parent: 02_article_foundation
fest_order: 6
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.606801-06:00
fest_updated: 2026-09-16T01:41:28.360409-06:00
fest_tracking: true
---


# Task: Enable the author's create and tags tests

## Objective

Enable the author's `create article` and `get all tags` tests, add the missing negative and leak tests, and record what the suite now actually runs.

## Requirements

- [ ] Per D010, the class-level `@Ignore` on `ArticleControllerTest` (`:18`) and `TagControllerTest` (`:14`) is replaced. Every test this slice cannot pass gets a method-level `@Ignore("<endpoint> is still stubbed; out of scope per D001")`, and `create article` and `get all tags` run.
- [ ] Every `/api` prefix in `ArticleControllerTest.kt`, `TagControllerTest.kt` and `HttpUtil.kt:70` is removed (D002). Colliding usernames get unique suffixes, and no assertion is weakened (D009)
- [ ] New tests: create without a token → 401; blank title → 422; missing `body` → 422; duplicate title → slug ending `-2`; raw-JSON author leak check (D008); ISO date string
- [ ] A test confirms or refutes D009's static reading that rows persist across test methods

## Implementation

**Steps**

1. **`HttpUtil.kt:70`.** Change `post<ArticleDTO>("/api/articles", ...)` to `post<ArticleDTO>("/articles", ...)`.
2. **`ArticleControllerTest.kt`.**
   - Remove the class-level `@Ignore` at `:18`.
   - Give every test except `create article` (`:124`) a method-level `@Ignore` whose reason names what is still stubbed: list and filters (`GET /articles`), feed, get by slug, update, favorite/unfavorite (enabled in `04_popular_articles`), and delete.
   - Replace every `"/api/` with `"/`.
3. **`TagControllerTest.kt`.**
   - Remove `@Ignore` at `:14` and replace the `/api/` paths.
   - Give the registered user a unique username. It currently registers `user_name_test` (`:30`), which is also `HttpUtil.createUser()`'s default (`HttpUtil.kt:61`). The users table makes `username` unique (`UserRepository.kt:21`), so the second registration fails silently, the later POST returns 401, and `tags.isNotEmpty()` can still pass on *another* test's data.
   - Assert the article POST returned 200, and that both `dragons` and `training` are present, so the test proves its own write.
4. **New test class** `src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt`.
   - Setup: `@Rule val appRule = AppRule()`, as in `UserControllerTest.kt:14`. Each test uses a `UUID`-suffixed user and title.
   - **No token:** a fresh `HttpUtil(appRule.port)` POSTs a valid article → 401.
   - **Blank title** → 422. **Payload without `body`** → 422, which exercises task 05's `runCatching`.
   - **Duplicate title:** two creates with the same title → the second slug ends with `-2`.
   - **Raw response:** the JSON from `postRaw("/articles", ...)` passes `assertNoAuthorSecrets` and contains `"createdAt":"` (an ISO string, not a number).
   - **Persistence probe (D009).**
     - Annotate the class with `@FixMethodOrder(MethodSorters.NAME_ASCENDING)`, since JUnit 4 does not otherwise guarantee method order.
     - `a_writes_row` creates a uniquely titled article and stores its slug in a companion-object field.
     - `b_row_survives_new_app` runs `transaction { Articles.select { Articles.slug eq storedSlug }.count() }` and asserts `1L`.
     - Record the outcome in `results/06_persistence.md`. If the row does not survive, D009's isolation rule is still right but its stated reason is wrong, so amend D009's context.
5. **Suite and census.** Run `just test all`, then `just test census` (`.justfiles/test.just:40-68`), and save both outputs to `results/06_census.md`. Expected:
   - `UserControllerTest`: 4 ran.
   - `ArticleControllerTest`: 1 ran, the rest skipped, each with a reason.
   - `TagControllerTest`: 1 ran.
   - The new classes: every test ran.

**Error paths**

- **`create article` fails with 401:** the default `createUser()` login failed because an earlier test registered `user@valid_user_mail.com` with a different password. Tests must not reuse that email with another password.
- **`get all tags` passes but the POST returned 401:** this is the silent false pass step 3 closes. The new status assertion must now fail it.
- **Census still shows a class as entirely disabled:** a class-level `@Ignore` was left in place.

## Done When

- [ ] All requirements met
- [ ] `results/06_census.md` shows `ArticleControllerTest` running `create article` with every other test skipped with a reason, `TagControllerTest` running `get all tags`, and every new test running and passing
- [ ] `results/06_persistence.md` states whether rows persisted across test methods, backed by the probe's actual result