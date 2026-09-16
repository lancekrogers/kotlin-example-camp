# Gate 04: Testing and Verification

## Changes

No application source changes in this gate (verification only).

- `results/04_build_matrix.txt` — raw output of `just build matrix`
- `results/04_test_all.txt` — raw output of `just build gradle "test --rerun-tasks"` (real execution; see Notes)
- `results/04_census.txt` — raw output of `just test census`
- `results/04_security_audit.txt` — raw output of `just security audit`
- `results/04_docker_verify.txt` — raw output of `just docker verify`
- `results/04_testing.md` — this evidence file

### git diff --stat

```
(no changes in kotlin-ktor-realworld-example-app)
```

### git status --short --untracked-files=all

```
(no changes in kotlin-ktor-realworld-example-app)
```

Festival evidence files (untracked under camp repo):

```
?? 003_IMPLEMENT/05_user_activity/results/04_build_matrix.txt
?? 003_IMPLEMENT/05_user_activity/results/04_census.txt
?? 003_IMPLEMENT/05_user_activity/results/04_docker_verify.txt
?? 003_IMPLEMENT/05_user_activity/results/04_security_audit.txt
?? 003_IMPLEMENT/05_user_activity/results/04_test_all.txt
?? 003_IMPLEMENT/05_user_activity/results/04_testing.md
```

## Commands

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `results/04_build_matrix.txt` |
| `just build gradle "test --rerun-tasks"` | 0 | `results/04_test_all.txt` |
| `just test census` | 0 | `results/04_census.txt` |
| `just security audit` | 0 | `results/04_security_audit.txt` |
| `just docker verify` | 0 | `results/04_docker_verify.txt` |

### `just build matrix`

Exit code: **0**. Ends with `matrix OK: JDK 17 and 21`. Full output: `results/04_build_matrix.txt`.

### `just test all`

Exit code: **0**. JDK 17 leg of `just build matrix` reported `:test FROM-CACHE`; re-ran with `just build gradle "test --rerun-tasks"` so evidence shows real execution. All non-skipped tests PASSED. Full output: `results/04_test_all.txt`.

### `just test census`

Exit code: **0**. Verbatim:

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleFollowingMappingTest ran=1   passed=1   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=6   passed=6   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=1   passed=1   failed=0   skipped=2
  CommentCreateTest          ran=4   passed=4   failed=0   skipped=0
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileStatsTest           ran=5   passed=5   failed=0   skipped=0
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=93 passed=93 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

Test counts from `build/test-results/test/*.xml`: 109 tests defined, 93 ran, 93 passed, 0 failed, 16 skipped.

### `just security audit`

Exit code: **0**. Ends with `static audit: PASSED`. Full output: `results/04_security_audit.txt`.

### `just docker verify`

Exit code: **0**. Ends with `smoke: PASSED`. Full output: `results/04_docker_verify.txt`.

**Note:** `just docker verify` only smoke-tests the user/auth endpoints (register, login, get current user, wrong password, blank password, no token). It does **not** exercise POST `/articles/{slug}/comments` or GET `/profiles/{username}/stats`. Container exercises for those endpoints are recorded in `results/01_comments_schema_and_add_comment.md` and `results/02_profile_stats_service_and_route.md`.

## Done When

### Commands

- [x] **`just build matrix` ends with `matrix OK: JDK 17 and 21`** — pass. Evidence: `results/04_build_matrix.txt` line `matrix OK: JDK 17 and 21`, `EXIT_CODE=0`.
- [x] **`just test all` passes** — pass. Evidence: `results/04_test_all.txt` `BUILD SUCCESSFUL`, `EXIT_CODE=0`; census `TOTAL ran=93 passed=93 failed=0`.
- [x] **`just test census` output saved; no test class fully disabled without a reason (D010)** — pass. Evidence: `results/04_census.txt`. Every fully disabled class has a class-level `@Ignore` reason string (grep below).
- [x] **`just security audit` passes** — pass. Evidence: `results/04_security_audit.txt` `static audit: PASSED`, `EXIT_CODE=0`.
- [x] **`just docker verify` passes; new endpoints exercised per their tasks** — pass (with caveat). `just docker verify` exit 0 (`results/04_docker_verify.txt`). POST comment and GET profile stats container exercises are in `results/01_comments_schema_and_add_comment.md` and `results/02_profile_stats_service_and_route.md`.

### Behavior

- [x] **Every new endpoint has tests for success, 422, 404 where applicable, anonymous where public (D003)** — pass.
  - **POST `/articles/{slug}/comments` (auth required):** success — `add comment for article by slug` (CommentControllerTest); 422 — `blank body returns 422` (CommentCreateTest); 404 — `unknown slug returns 404` (CommentCreateTest); anonymous — n/a (401 without token: `no token returns 401`, CommentCreateTest).
  - **GET `/profiles/{username}/stats` (public read):** success — `new user has zero activity`, `article comment and favorite each count once`, `favorites count what the user gave` (ProfileStatsTest); 422 — n/a (no request body); 404 — `unknown username is 404` (ProfileStatsTest); anonymous — `anonymous request is public` (ProfileStatsTest, D003).
- [x] **Every endpoint returning an article or comment passes `assertNoAuthorSecrets` on raw JSON (D008)** — pass.
  - **POST `/articles/{slug}/comments`:** `raw response has no author secrets` (CommentCreateTest).
  - **GET `/profiles/{username}/stats`:** returns `ProfileStatsDTO` only (no article/comment author); n/a for D008.
- [x] **New tests create uniquely named data and assert only on rows they created (D009)** — pass. `CommentCreateTest`, `CommentControllerTest` `add comment for article by slug`, and all five `ProfileStatsTest` methods use `UUID.randomUUID()` token suffixes on emails, usernames, and titles; stats tests assert only on the named user's counts.
- [x] **Every unverified claim this sequence depends on is proven by a test or recorded as failing** — pass.
  - Test row persistence (D009): `ArticleCreateTest` `b_row_survives_new_app` still green (census `ArticleCreateTest ran=9 passed=9`).
  - LOWER on CLOB (D004): `LowerOnClobProbeTest` still green (census `ran=1 passed=1`).
  - H2 GROUP BY / aggregate counts (D005/D006): `ProfileStatsTest` `article comment and favorite each count once` proves articlesCount, commentsCount, and favoritesCount each reach 1; `ArticleFavoritesRepositoryTest` still green.
  - `CommentRepository.countByAuthor`: exercised indirectly via `commentsCount=1` in `ProfileStatsTest` `article comment and favorite each count once`.

### Evidence

- [x] **Raw output of each command saved under `results/`** — pass. Files listed in Commands table above.

## Notes

### D010 `@Ignore` reasons (grep, not census marker alone)

Current census accurately reflects D010 per-slice enablement:

- **ProfileControllerTest** — fully disabled (`ran=0 skipped=3`). Class-level reason:

```
src/test/kotlin/io/realworld/app/web/controllers/ProfileControllerTest.kt:13:@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")
```

- **CommentControllerTest** — runs 1 of 3. Method-level reasons on the other two:

```
src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt:50:    @Ignore("GET /articles/{slug}/comments is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt:70:    @Ignore("DELETE /articles/{slug}/comments/{id} is still stubbed; out of scope per D001")
```

- **ArticleControllerTest** — runs 6 of 17. Eleven method-level `@Ignore` reasons, all citing stubbed endpoints per D001:

```
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:26:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:38:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:52:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:68:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:85:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:103:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:119:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:149:    @Ignore("GET /articles/feed is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:169:    @Ignore("GET /articles/{slug} is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:184:    @Ignore("PUT /articles/{slug} is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:303:    @Ignore("DELETE /articles/{slug} is still stubbed; out of scope per D001")
```

No `@Ignore` without a reason string. The census `<-- entire class disabled` marker applies only to `ProfileControllerTest`.

### Other notes

- Branch/camp-fresh step skipped per orchestrator instruction (already on `feat/user-activity`).
- `just test all` was not invoked directly; JDK 17 matrix leg cached `:test FROM-CACHE`, so evidence uses `just build gradle "test --rerun-tasks"` equivalent to a fresh suite run.
- `just docker verify` smoke-tests auth only; sequence-specific endpoint container checks live in tasks 01 and 02 results files cited above.
- No file:line anchor drift encountered.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `0d26f9e8-eee9-45cf-b8f7-8d462648e7b2`, 09:25:43Z to 09:29:59Z).
Checked against the raw files rather than the summary:

- **All five commands exited 0**, each evidence file ending `EXIT_CODE=0`.
- **`just build matrix`** ends `matrix OK: JDK 17 and 21` (`04_build_matrix.txt:297`).
- **Cache honesty, third slice running.** The JDK 17 matrix leg reported `:test FROM-CACHE`, so the gate re-ran the
  suite with `--rerun-tasks` and recorded that run (`BUILD SUCCESSFUL in 1m 3s`) rather than filing a no-op.
- **Counts match an independent measurement:** 109 defined, 93 ran, 93 passed, 0 failed, 16 skipped — identical to the
  orchestrator's own `cleanTest test --no-build-cache` run during task 03 verification. Per class: `ProfileStatsTest` 5,
  `CommentCreateTest` 4, `CommentControllerTest` 1 of 3, `ProfileControllerTest` 0 of 3 (disabled with a reason).
- **`just security audit`** ends `static audit: PASSED`; **`just docker verify`** ends `smoke: PASSED`, with no `ktor`
  container left running afterwards.
- **The gate changed no application code**; `git status --untracked-files=all` on the project is empty.
- **Its stated limitation is accurate.** `just docker verify` runs `just docker smoke`, which exercises only the user
  endpoints. Container proof for `POST /articles/{slug}/comments` and `GET /profiles/{username}/stats` comes from
  `results/01_comments_schema_and_add_comment.md` and `results/02_profile_stats_service_and_route.md`, both of which
  include the orchestrator's own independent runs.
- **D010 proven by grep** across all classes: `ProfileControllerTest` disabled with a class-level reason,
  `CommentControllerTest` running 1 of 3 with method-level reasons, `ArticleControllerTest` running 6 of 17.

No new findings from this gate. Gate 06 addresses the one accepted item from `results/05_review.md`.
