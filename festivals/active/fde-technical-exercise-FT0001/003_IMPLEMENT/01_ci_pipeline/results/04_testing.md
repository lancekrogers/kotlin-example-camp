# Gate 04: Testing and Verification — Results

> **Orchestrator verification (2026-09-15).** A cursor-agent subagent ran this gate (`composer-2.5`, session
> `89832068-0fa5-421e-a5fa-7ff666b3dc9c`, 20:56:17Z to 20:59:12Z). The orchestrator checked it against the raw files:
> every `04_*.txt` ends in `exit=0`; both no-cache runs show `> Task :test` executing, not `FROM-CACHE`;
> `04_build_matrix.txt` still shows `:test FROM-CACHE`, which is why the no-cache runs were added; the census matches;
> and the repository was unchanged afterwards. Two checkbox results below are amended:
>
> - **D010 census: "fail" is deferred, not open.** The four class-level `@Ignore`s without reasons predate this
>   workflow-only slice. `02_article_foundation` requires "the remaining ignored tests carry reasons; census
>   recorded" (see `06_iterate.md` T1).
> - **Unverified claims: "not applicable" is wrong, and the claim is now proven.** This sequence depended on the zero-run cause.
>   Workflows are now shown to run on the fork (run 35022352856, `01_zero_ci_runs.md` Confirmation). The
>   original cause stays `unknown` with what was ruled out, which the task allows.
>
> Cursor's stderr held only a transient `Connection lost, reconnecting … Retry attempt 1...`, with no effect on the results.

Sequence `01_ci_pipeline` on branch `ci/jdk-matrix`. Only `.github/workflows/gradle.yml` changed; no endpoints or application code.

## Command evidence

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `04_build_matrix.txt` |
| `JDK=17 just build gradle "cleanTest test --no-build-cache"` | 0 | `04_test_jdk17_nocache.txt` |
| `JDK=21 just build gradle "cleanTest test --no-build-cache"` | 0 | `04_test_jdk21_nocache.txt` |
| `just test census` | 0 | `04_census.txt` |
| `just security audit` | 0 | `04_security_audit.txt` |

`just docker verify` was **not run** — this sequence added or changed no endpoints.

## Census (verbatim from `04_census.txt`)

```
  ArticleControllerTest      ran=0   passed=0   failed=0   skipped=14  <-- entire class disabled
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=0   passed=0   failed=0   skipped=1  <-- entire class disabled
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0

  TOTAL ran=4 passed=4 failed=0 skipped=21

  WARNING: 21 test(s) skipped. A green build does not mean the application works.
```

## `:test` execution proof (steps 2 and 3)

`:test` ran on both JDKs (no `FROM-CACHE` or `UP-TO-DATE` suffix on the `:test` task line). Individual test outcomes:

**JDK 17** (`04_test_jdk17_nocache.txt`):

```
> Task :test

ArticleControllerTest > delete article by slug SKIPPED
… (17 more SKIPPED in ArticleControllerTest, CommentControllerTest, ProfileControllerTest, TagControllerTest)

UserControllerTest > update user data PASSED
UserControllerTest > get current user by token PASSED
UserControllerTest > success login with email and password PASSED
UserControllerTest > success register user PASSED

BUILD SUCCESSFUL in 13s
6 actionable tasks: 5 executed, 1 up-to-date
```

**JDK 21** (`04_test_jdk21_nocache.txt`):

```
> Task :test

ArticleControllerTest > delete article by slug SKIPPED
… (17 more SKIPPED in ArticleControllerTest, CommentControllerTest, ProfileControllerTest, TagControllerTest)

UserControllerTest > update user data PASSED
UserControllerTest > get current user by token PASSED
UserControllerTest > success login with email and password PASSED
UserControllerTest > success register user PASSED

BUILD SUCCESSFUL in 14s
6 actionable tasks: 5 executed, 1 up-to-date
```

Test counts: **4 passed, 21 skipped, 0 failed** on each JDK.

## Gate checkboxes

### Commands

| Checkbox | Result | Reason |
|----------|--------|--------|
| `just build matrix` ends with `matrix OK: JDK 17 and 21` | **pass** | Last line of `04_build_matrix.txt` is `matrix OK: JDK 17 and 21`. |
| `just test all` passes | **pass** | Not run directly; JDK 17 and JDK 21 `cleanTest test --no-build-cache` both exited 0 with `BUILD SUCCESSFUL`. |
| `just test census` output saved; no test class fully disabled without a reason (D010) | **fail** | Census saved to `04_census.txt`; four classes are fully disabled and `@Ignore` has no reason string in source. |
| `just security audit` passes | **pass** | `04_security_audit.txt` ends with `static audit: PASSED`, exit 0. |
| `just docker verify` (if endpoints changed) | **not applicable** | Sequence changed only CI workflow; no endpoints added or changed. |

### Behavior

| Checkbox | Result | Reason |
|----------|--------|--------|
| Every new endpoint has tests for success, 422, 404, anonymous access (D003) | **not applicable** | No new endpoints in this sequence. |
| Every article/comment endpoint passes `assertNoAuthorSecrets` (D008) | **not applicable** | No endpoint changes in this sequence. |
| New tests create uniquely named data and assert only on rows they created (D009) | **not applicable** | No new tests in this sequence. |
| Every unverified claim this sequence depends on is now proven or recorded as failing | **not applicable** | CI workflow replacement only; no application claims under test here. |

### Evidence

| Checkbox | Result | Reason |
|----------|--------|--------|
| Raw output of each command saved under `results/` | **pass** | Five evidence files written with full stdout+stderr and trailing `exit=N`. |
