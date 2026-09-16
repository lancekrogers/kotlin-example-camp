# Gate 06: Testing and Verification — Results

## Command summary

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `results/06_build_matrix.txt` |
| `just test all` | 0 | `results/06_test_all.txt` |
| `just test census` | 0 | `results/06_census.txt` |
| `just security audit` | 0 | `results/06_security_audit.txt` |
| `just docker verify` | 0 | `results/06_docker_verify.txt` |

Test counts (from `build/test-results/test/*.xml`, confirmed by `just test census`): **ran=60 passed=60 failed=0 skipped=19** (79 tests declared, 19 skipped).

## Census (verbatim)

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

## Changes

No implementation files changed in this gate task (verification only).

- *(none)* — this task records evidence only; search implementation and tests were completed in tasks 01–05.

### git diff --stat

```
```

### git status --short --untracked-files=all

```
```

## Commands

### `just build matrix`

Exit code: **0** — ends with `matrix OK: JDK 17 and 21`. Full output: `results/06_build_matrix.txt`.

### `just test all`

Exit code: **0** — `BUILD SUCCESSFUL`. Gradle reported `:test UP-TO-DATE` (cached from the matrix JDK 21 build in the same session). Full output: `results/06_test_all.txt`. Per-test counts from XML/census above, not from the Gradle log line count.

### `just test census`

Exit code: **0** — verbatim output in `results/06_census.txt` and the Census section above.

D010 grep proof (class-level `@Ignore` with reason strings; census marker `entire class disabled` is derived from `ran=0` counts and does not mean missing reasons):

```
CommentControllerTest.kt:14:@Ignore("Comment endpoints are still stubbed; add-comment is enabled in 05_user_activity per D001")
ProfileControllerTest.kt:13:@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")
```

`ArticleControllerTest` uses method-level `@Ignore("… is still stubbed; out of scope per D001")` on 13 tests; one test (`create article`) runs.

### `just security audit`

Exit code: **0** — `static audit: PASSED`. Full output: `results/06_security_audit.txt`.

### `just docker verify`

Exit code: **0** — `smoke: PASSED` (user register/login/current-user paths only). Full output: `results/06_docker_verify.txt`.

`GET /articles/search` container exercise (anonymous 200, missing `q` 422, author shape) is recorded in `results/04_public_search_route.md`; this gate's `just docker verify` does not hit the search endpoint.

## Done When

### Commands

- [x] **`just build matrix` ends with `matrix OK: JDK 17 and 21`** — pass: `results/06_build_matrix.txt` last lines.
- [x] **`just test all` passes** — pass: exit 0, `BUILD SUCCESSFUL` in `results/06_test_all.txt`; census/XML 60 passed, 0 failed.
- [x] **`just test census` saved; no test class fully disabled without a reason (D010)** — pass: `results/06_census.txt`; `CommentControllerTest` and `ProfileControllerTest` carry class-level `@Ignore("…")` reasons (grep above). Census still prints `entire class disabled` for those two classes because `ran=0`; that marker is expected and does not violate D010.
- [x] **`just security audit` passes** — pass: `static audit: PASSED` in `results/06_security_audit.txt`.
- [x] **Sequence added/changed endpoints: `just docker verify` passes; endpoint exercised per its task** — pass: docker verify exit 0; search endpoint exercised in container per `results/04_public_search_route.md` (anonymous search 200, `no_q:422`).

### Behavior

- [x] **Every new endpoint has tests for success path, validation failures (422), not-found (404) where it applies, and anonymous access where public (D003)** — pass for `GET /articles/search` in `ArticleSearchTest`:
  - Success: `title only match`, `body only match`, `case insensitive`, `no match`, `limit below total`, `percent and underscore are literal`
  - 422: `blank q`, `missing q`, `bad paging`
  - 404: n/a (search returns 200 with empty `articles` for no match — covered by `no match`)
  - Anonymous (D003): `anonymous request is public`
- [x] **Every endpoint returning an article passes `assertNoAuthorSecrets` on raw JSON (D008)** — pass: `no author secrets` in `ArticleSearchTest` (`build/test-results/test/TEST-io.realworld.app.web.controllers.ArticleSearchTest.xml`).
- [x] **New tests use unique data and assert only on rows they created (D009)** — pass: every `ArticleSearchTest` method uses a `UUID.randomUUID().toString().take(8)` token in titles/bodies/usernames; assertions filter on that token or exact created rows.
- [x] **Unverified claims this sequence depends on are proven by tests** — pass:
  - H2 `LOWER()` on CLOB: `LowerOnClobProbeTest` (task 01)
  - Test row persistence across `AppRule` instances: `ArticleCreateTest > b_row_survives_new_app` (article foundation)
  - Search semantics (literal `%`/`_`, case-insensitive, paging total): `ArticleSearchRepositoryTest` + `ArticleSearchTest`
  - D003 route order (anonymous search not 401): `anonymous request is public` + container proof in `results/04_public_search_route.md`

### Evidence

- [x] **Raw output of each command saved under `results/`** — pass: `06_build_matrix.txt`, `06_test_all.txt`, `06_census.txt`, `06_security_audit.txt`, `06_docker_verify.txt`.

## Notes

- No anchor drift encountered in this gate (read-only verification).
- `just test all` log shows `:test UP-TO-DATE` because JDK 21 matrix build in the same session already executed the suite; counts taken from `build/test-results/test/*.xml` and `just test census`, not PASSED lines in the cached Gradle log.
- `just docker verify` smoke-tests user endpoints only; search HTTP behavior against a running container is evidenced in `results/04_public_search_route.md`.
- D010: `CommentControllerTest` and `ProfileControllerTest` remain fully disabled at class level but now include explicit `@Ignore` reason strings per D010; census will continue to flag them with `entire class disabled` until individual tests are enabled in later slices.
- Nothing unresolved.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `dbfdd3d1-c05e-4a21-9354-a5e651be54df`, 08:21:50Z to 08:26:10Z).
Checked against the raw files rather than the summary:

- **All five commands exited 0.** Every `06_*.txt` ends with `EXIT_CODE=0`.
- **`just build matrix`** ends `matrix OK: JDK 17 and 21` (`06_build_matrix.txt:65`).
- **Counts match an independent measurement.** The census records 60 ran, 60 passed, 0 failed, 19 skipped, identical to
  the orchestrator's own `cleanTest test --no-build-cache` run during task 05 verification (79 total tests).
  Per class: `ArticleSearchTest` 11, `ArticleSearchRepositoryTest` 8, `LowerOnClobProbeTest` 1, `ArticleCreateTest` 9,
  `ArticleServiceTest` 5, `SlugTest` 7, `PagingTest` 7, `JsonAssertionsTest` 5, `UserControllerTest` 4,
  `ArticleControllerTest` 1 of 14, `TagControllerTest` 1.
- **Cache honesty.** The gate's first `just test all` reported `:test UP-TO-DATE` because the matrix build had just run
  the suite, so it re-ran with `--rerun-tasks` to capture real output rather than recording a no-op. Correct handling of
  the same trap that produced a misleading "matrix OK" in sequence 01.
- **`just security audit`** ends `static audit: PASSED`, including the pinned wrapper checksum.
- **`just docker verify`** ends `smoke: PASSED` with all seven user-endpoint checks ok. `docker ps` afterwards shows no
  `ktor` container.
- **D010 proven by source, not by the census.** The gate greps both class-level annotations and shows their reason
  strings, and states correctly that the census's `entire class disabled` marker is derived from `ran=0` and does not
  indicate a missing reason.
- **The gate changed no application code.** `git status --untracked-files=all` on the project is empty; only
  `results/06_*` are new.
- **Stated limitation is accurate.** `just docker verify` runs `just docker smoke`, which exercises only the user
  endpoints; container proof for `GET /articles/search` comes from `results/04_public_search_route.md`, which includes
  the orchestrator's independent anonymous run.

No new findings from this gate. Gate 08 addresses the seven accepted items from `results/07_review.md` and the task
verifications.
