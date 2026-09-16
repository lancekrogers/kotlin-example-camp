# Gate 09: Review Results and Iterate — Results

## Changes

| File | Description |
|------|-------------|
| `src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt` | Added D010 class-level `@Ignore` reason for stubbed comment endpoints (C1) |
| `src/test/kotlin/io/realworld/app/web/controllers/ProfileControllerTest.kt` | Added D010 class-level `@Ignore` reason for stubbed profile endpoints (C2) |
| `src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt` | Added HTTP-level `blank description returns 422` test (S5) |

```
 .../realworld/app/web/controllers/ArticleCreateTest.kt  | 17 +++++++++++++++++
 .../app/web/controllers/CommentControllerTest.kt        |  2 +-
 .../app/web/controllers/ProfileControllerTest.kt        |  2 +-
 3 files changed, 19 insertions(+), 2 deletions(-)
```

```
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/ProfileControllerTest.kt
```

## Commands

### `just test only ArticleCreateTest` — exit code 0

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin UP-TO-DATE
> Task :compileJava NO-SOURCE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :compileTestKotlin
> Task :compileTestJava NO-SOURCE
> Task :testClasses UP-TO-DATE

> Task :test

ArticleCreateTest > a_writes_row PASSED

ArticleCreateTest > b_row_survives_new_app PASSED

ArticleCreateTest > blank description returns 422 PASSED

ArticleCreateTest > blank title returns 422 PASSED

ArticleCreateTest > create without token returns 401 PASSED

ArticleCreateTest > duplicate title slug ends with -2 PASSED

ArticleCreateTest > missing body returns 422 PASSED

ArticleCreateTest > raw response has no author secrets and ISO createdAt PASSED

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 14s
5 actionable tasks: 3 executed, 2 up-to-date
EXIT_CODE: 0
```

### `just test all` — exit code 0

Full output saved to `results/09_test_all.txt`. Ends with:

```
BUILD SUCCESSFUL in 18s
5 actionable tasks: 2 executed, 3 up-to-date
EXIT_CODE: 0
```

Per-test summary: 51 tests completed (32 ran, 19 skipped, 0 failed). New test `ArticleCreateTest > blank description returns 422 PASSED`.

### `just test census` — exit code 0

Full output saved to `results/09_census.txt`:

```
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=8   passed=8   failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=32 passed=32 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
EXIT_CODE: 0
```

### `grep -n -B2 'class CommentControllerTest'` — exit code 0

```
13-
14-@Ignore("Comment endpoints are still stubbed; add-comment is enabled in 05_user_activity per D001")
15:class CommentControllerTest {
```

### `grep -n -B2 'class ProfileControllerTest'` — exit code 0

```
12-
13-@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")
14:class ProfileControllerTest {
```

### `git diff --stat` — exit code 0

```
 .../realworld/app/web/controllers/ArticleCreateTest.kt  | 17 +++++++++++++++++
 .../app/web/controllers/CommentControllerTest.kt        |  2 +-
 .../app/web/controllers/ProfileControllerTest.kt        |  2 +-
 3 files changed, 19 insertions(+), 2 deletions(-)
```

### `git status --short --untracked-files=all` — exit code 0

```
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleCreateTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/CommentControllerTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/ProfileControllerTest.kt
```

## Done When

### Findings addressed

| Finding | Source | Change | Evidence |
|---------|--------|--------|----------|
| D010 bare `@Ignore` on `CommentControllerTest` | Gate 07, Gate 08 C1 | Added class-level reason string per D001/D010 | `grep` shows `@Ignore("Comment endpoints are still stubbed; add-comment is enabled in 05_user_activity per D001")` at line 14 |
| D010 bare `@Ignore` on `ProfileControllerTest` | Gate 07, Gate 08 C2 | Added class-level reason string per D001/D010 | `grep` shows `@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")` at line 13 |
| No HTTP-level 422 for blank `description` | Gate 08 S5 | Added `blank description returns 422` test in `ArticleCreateTest` | `just test only ArticleCreateTest`: `ArticleCreateTest > blank description returns 422 PASSED`; census `ArticleCreateTest ran=8` |

### Definition of Done (gate 09)

- [x] **All critical findings are fixed** — **pass** — C1 and C2 resolved with documented `@Ignore` reasons; S5 accepted test added
- [x] **`just test all` passes after the changes** — **pass** — exit 0; 32 ran, 19 skipped, 0 failed (`results/09_test_all.txt`)
- [x] **Code review findings are addressed or explicitly deferred with a reason** — **pass** — C1/C2 fixed; S5 added; S1–S4 remain deferred per gate 08 with recorded reasons (no action required in this gate)
- [x] **Ready to commit** — **pass** — three test files only; all verification green

## Notes

- **Anchor drift:** None. `CommentControllerTest.kt:14`, `ProfileControllerTest.kt:13`, and `ArticleCreateTest.kt` blank-title test at lines 54–68 matched task expectations; new test inserted immediately after blank-title test.
- **Census `entire class disabled` marker:** Expected per task instructions. `just test census` flags from test counts, not `@Ignore` reason strings; both `CommentControllerTest` and `ProfileControllerTest` still show `ran=0 skipped=3 <-- entire class disabled`. D010 compliance is proven by the `grep` output showing non-empty reason strings.
- **Test count delta:** Census `TOTAL ran=32` (was 31 in gate 07) due to the new `blank description returns 422` test; `skipped=19` unchanged.
- **Out of scope preserved:** Original commented-out stub code in `ArticleController.kt` untouched; `/api/` paths in `CommentControllerTest` and `ProfileControllerTest` untouched; no application code changed.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `cbb962db-7f1b-4790-bcde-19d0fca430d5`, 07:46:38Z to 07:48:24Z).

- **Exactly the three requested files changed**, nothing else: `git status --untracked-files=all` lists
  `ArticleCreateTest.kt`, `CommentControllerTest.kt` and `ProfileControllerTest.kt`. The original author's commented-out
  stubs and the `/api` paths in the two still-disabled files are untouched, as instructed.
- **C1 and C2 fixed in source**, not just claimed:
  ```text
  CommentControllerTest.kt:14 @Ignore("Comment endpoints are still stubbed; add-comment is enabled in 05_user_activity per D001")
  ProfileControllerTest.kt:13 @Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")
  ```
  A repository-wide `grep -rn '@Ignore$' src/test/` now returns nothing, so no disabled test anywhere lacks a reason.
  As predicted, `just test census` still prints `<-- entire class disabled` for both classes, because it derives that
  marker from test counts and cannot see a reason string. The grep above is the evidence that satisfies D010.
- **S5 is a genuine boundary test.** `blank description returns 422` registers a UUID-suffixed user, posts through
  `postRaw` (needed because the 422 body is an error object that typed deserialization rejects), and asserts
  `SC_UNPROCESSABLE_ENTITY`. The service-level test in `ArticleServiceTest` remains; this proves the same rule at the
  HTTP layer the client actually sees.
- **Independent suite run with the cache off:**
  ```text
  $ just build gradle "cleanTest test --no-build-cache"
  suite_exit=0    BUILD SUCCESSFUL in 18s
  ArticleCreateTest cases: ['a_writes_row', 'b_row_survives_new_app', 'blank description returns 422',
    'blank title returns 422', 'create without token returns 401', 'duplicate title slug ends with -2',
    'missing body returns 422', 'raw response has no author secrets and ISO createdAt']
  TOTAL tests=51 ran=32 failed=0 skipped=19
  ```
  32 running, up from 31 at gate 07 and from 4 when this slice began.

All findings from gates 07 and 08 are now either fixed (C1, C2, S5) or deferred with a recorded reason (S1-S4 in
`results/08_review.md`). Nothing is outstanding.
