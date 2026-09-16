# Gate 05: Testing and Verification

## Changes

No application source changes in this gate (verification only).

- `results/05_build_matrix.txt` — raw output of `just build matrix`
- `results/05_test_all.txt` — raw output of `just test all` (cached) plus `just build gradle "test --rerun-tasks"`
- `results/05_census.txt` — raw output of `just test census`
- `results/05_security_audit.txt` — raw output of `just security audit`
- `results/05_docker_verify.txt` — raw output of `just docker verify`
- `results/05_testing.md` — this evidence file

### git diff --stat

```
(no changes in kotlin-ktor-realworld-example-app)
```

### git status --short --untracked-files=all

```
(no changes in kotlin-ktor-realworld-example-app)
```

## Commands

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `results/05_build_matrix.txt` |
| `just test all` | 0 | `results/05_test_all.txt` (initial run reported `:test FROM-CACHE`; re-ran with `--rerun-tasks` in same file) |
| `just test census` | 0 | `results/05_census.txt` |
| `just security audit` | 0 | `results/05_security_audit.txt` |
| `just docker verify` | 0 | `results/05_docker_verify.txt` |

### `just build matrix`

Exit code: **0**. Ends with `matrix OK: JDK 17 and 21`. Full output: `results/05_build_matrix.txt`.

### `just test all`

Exit code: **0**. Initial invocation reported `:test FROM-CACHE` / `UP-TO-DATE`; re-ran with `just build gradle "test --rerun-tasks"` so evidence shows real execution. All non-skipped tests PASSED. Full output: `results/05_test_all.txt`.

### `just test census`

Exit code: **0**. Verbatim:

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=3   passed=3   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  PopularArticlesTest        ran=10  passed=10  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=78 passed=78 failed=0 skipped=17

  WARNING: 17 test(s) skipped. A green build does not mean the application works.
```

XML totals from `build/test-results/test/*.xml`: 95 tests defined, 78 ran, 78 passed, 0 failed, 17 skipped.

### `just security audit`

Exit code: **0**. Ends with `static audit: PASSED`. Full output: `results/05_security_audit.txt`.

### `just docker verify`

Exit code: **0**. Ends with `smoke: PASSED`. Full output: `results/05_docker_verify.txt`.

**Note:** `just docker verify` only smoke-tests the user/auth endpoints (register, login, get current user, wrong password, blank password, no token). It does not exercise GET `/articles/feed/popular`, POST `/articles/{slug}/favorite`, or DELETE `/articles/{slug}/favorite`. Container exercises for those endpoints are recorded in `results/02_favorite_endpoints_and_real_counts.md` and `results/03_popular_query_service_and_route.md`.

## Done When

### Commands

- [x] **`just build matrix` ends with `matrix OK: JDK 17 and 21`** — pass. Evidence: `results/05_build_matrix.txt` line `matrix OK: JDK 17 and 21`, `EXIT_CODE: 0`.
- [x] **`just test all` passes** — pass. Evidence: `results/05_test_all.txt` `BUILD SUCCESSFUL`, `EXIT_CODE: 0`; census `TOTAL ran=78 passed=78 failed=0`.
- [x] **`just test census` output saved; no test class fully disabled without a reason (D010)** — pass. Evidence: `results/05_census.txt`. D010 `@Ignore` reasons verified by grep (see Notes).
- [x] **`just security audit` passes** — pass. Evidence: `results/05_security_audit.txt` `static audit: PASSED`, `EXIT_CODE: 0`.
- [x] **`just docker verify` passes; new endpoints exercised per their tasks** — pass (with caveat). `just docker verify` exit 0 (`results/05_docker_verify.txt`). Favorite/unfavorite and popular endpoint container exercises are in `results/02_favorite_endpoints_and_real_counts.md` and `results/03_popular_query_service_and_route.md`.

### Behavior

- [x] **Every new endpoint has tests for success, 422, 404 where applicable, anonymous where public (D003)** — pass (with notes).
  - **GET `/articles/feed/popular` (public):** success — `more favorites ranks first`, `equal counts newest first`, `zero favorite included`, `pages are stable`, `offset past end`; 422 — `bad limit`; 404 — n/a (list endpoint); anonymous — `anonymous is public` (D003).
  - **POST `/articles/{slug}/favorite`:** success — `favorite article by slug` (ArticleControllerTest), `double favorite` (PopularArticlesTest); 422 — n/a (slug from path); 404 — `unknown slug throws NotFoundException` (ArticleFavoritesRepositoryTest) plus container 404 in `results/02_favorite_endpoints_and_real_counts.md`; anonymous — n/a (auth required; 401 without token in task 02 container evidence).
  - **DELETE `/articles/{slug}/favorite`:** success — `unfavorite article by slug` (ArticleControllerTest); 404 — repo test + task 02 container; anonymous — n/a (auth required).
- [x] **Every endpoint returning an article passes `assertNoAuthorSecrets` on raw JSON (D008)** — pass for popular; partial for favorite/unfavorite.
  - **GET `/articles/feed/popular`:** `no author secrets` (PopularArticlesTest).
  - **POST/DELETE favorite:** no dedicated HTTP `assertNoAuthorSecrets` test; author mapping uses `Profile` via shared `toArticles` path also covered by `ArticleCreateTest` `raw response has no author secrets and ISO createdAt` and `ArticleSearchTest` `no author secrets`.
- [x] **New tests create uniquely named data and assert only on rows they created (D009)** — pass. PopularArticlesTest uses `UUID.randomUUID()` token suffixes on emails, usernames, and titles; ranking tests use `assertRelativeOrder` on slugs created in the test. ArticleControllerTest favorite/unfavorite tests use distinct `user_name_test_favorite` / `user_name_test_unfavorite` usernames.
- [x] **Unverified claims this sequence depends on are proven** — pass.
  - H2 GROUP BY / `COUNT(user)` ranking: `PopularArticlesTest` `more favorites ranks first` (zero-favorite article ranks below favorited).
  - LOWER on CLOB: `LowerOnClobProbeTest` (from search slice, still green).
  - Test row persistence: `ArticleCreateTest` `b_row_survives_new_app`.
  - COUNT(user) vs COUNT(*): `PopularArticlesTest` `more favorites ranks first` — article C with zero favorites ranks below B with one.

### Evidence

- [x] **Raw output of each command saved under `results/`** — pass. Files listed in Commands table above.

## Notes

### D010 `@Ignore` reasons (grep, not census marker alone)

```
src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt:14:@Ignore("Comment endpoints are still stubbed; add-comment is enabled in 05_user_activity per D001")
src/test/kotlin/io/realworld/app/web/controllers/ProfileControllerTest.kt:13:@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")
```

`CommentControllerTest` and `ProfileControllerTest` remain fully disabled at class level with reason strings (3 skipped tests each per census).

`ArticleControllerTest` runs 3 of 14 tests; 11 carry method-level `@Ignore` reasons:

```
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:24:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:36:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:50:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:66:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:83:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:101:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:117:    @Ignore("GET /articles is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:147:    @Ignore("GET /articles/feed is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:167:    @Ignore("GET /articles/{slug} is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:182:    @Ignore("PUT /articles/{slug} is still stubbed; out of scope per D001")
src/test/kotlin/io/realworld/app/web/controllers/ArticleControllerTest.kt:247:    @Ignore("DELETE /articles/{slug} is still stubbed; out of scope per D001")
```

Enabled tests (no `@Ignore`): `create article`, `favorite article by slug`, `unfavorite article by slug`.

### Anchor drift

None observed during this verification gate.

### Other notes

- `just test all` initially reported `:test FROM-CACHE` because the matrix build had already executed the suite; re-ran with `--rerun-tasks` per gate instructions.
- Favorite/unfavorite HTTP 404 is covered at repository level (`ArticleFavoritesRepositoryTest.unknown slug throws NotFoundException`) and by manual container curls in `results/02_favorite_endpoints_and_real_counts.md`, not by a dedicated HTTP integration test.
- `just docker verify` smoke-tests user endpoints only; sequence endpoint verification relies on integration tests plus task 02/03 container evidence.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `bbff3678-e997-4750-9023-b540c1231ef8`, 08:53:36Z to 09:00:14Z).
Checked against the raw files:

- **All five commands exited 0**, each evidence file ending `EXIT_CODE: 0`.
- **`just build matrix`** ends `matrix OK: JDK 17 and 21`.
- **Cache honesty, again handled correctly.** The first `just test all` came back `:test FROM-CACHE`, so the gate
  re-ran it with `--rerun-tasks` and recorded both, rather than filing a no-op as evidence.
- **Counts match an independent measurement:** 78 ran, 78 passed, 0 failed, 17 skipped, identical to the orchestrator's
  own `cleanTest test --no-build-cache` run in task 04 verification (95 total tests). `PopularArticlesTest` 10,
  `ArticleControllerTest` 3 of 14.
- **`just security audit`** ends `static audit: PASSED`. **`just docker verify`** ends `smoke: PASSED`, and `docker ps`
  afterwards shows no `ktor` container.
- **D010 proven by grep**, not by the census marker: both class-level reasons plus all eleven method-level reasons in
  `ArticleControllerTest`.
- **The gate changed no application code**; `git status --untracked-files=all` on the project is empty.

**It also raised two gaps of its own, honestly, rather than marking the checkboxes green:**

1. The favorite and unfavorite HTTP responses have no dedicated `assertNoAuthorSecrets` test. Same as the reviewer's S1.
2. **A gap the reviewer missed:** favorite's 404 on an unknown slug is proven at the repository level and by the
   orchestrator's container run (`results/02_favorite_endpoints_and_real_counts.md`), but no HTTP test pins it. The
   gate's Behavior checkbox asks for 404 coverage where it applies, so this is accepted for gate 07.

### Findings carried to gate 07

| # | Finding | Source |
|---|---|---|
| 1 | No raw-JSON leak test on the favorite and unfavorite responses (D008 requires one per article-returning endpoint) | review S1, testing gate |
| 2 | No HTTP test for favorite on an unknown slug → 404 | testing gate |
| 3 | No bad-`offset` case on popular; only `limit=0` is exercised there | review S2 |
| 4 | `following` is one `Follows` query per row in `toArticles`, while favorite lookups are batched | review S3 |
| 5 | **Orchestrator addition:** nothing anywhere proves `following` at all. The follow endpoints are still stubs, so the
      flag cannot be set through the API, and no test inserts a `Follows` row directly. Batching that query (finding 4)
      would otherwise be an unverified refactor of a field no test covers, so gate 07 adds a repository test that
      inserts a `Follows` row and asserts `following` is true for that viewer and false for another. | orchestrator |
