# Gate 04: Testing and Verification — evidence

Verification run from `projects/kotlin-ktor-realworld-example-app` on branch `feat/spec-tests`. No application or CI files were modified in this gate; only evidence files under this `results/` directory were added.

## Command summary

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `04_build_matrix.txt` |
| `just build gradle "test --rerun-tasks"` | 0 | `04_test_all.txt` |
| `just test census` | 0 | `04_census.txt` |
| `just security audit` | 0 | `04_security_audit.txt` |
| `just docker verify` | 0 | `04_docker_verify.txt` |
| `just docker up` + `just docker spec` + comparator + `just docker down` | spec 1, comparator 0, overall 0 | `04_spec_run.txt` |
| `grep` pins + runner guard | 0 | `04_pins.txt` |

**Note:** Initial `just test all` returned `:test FROM-CACHE`; evidence uses `just build gradle "test --rerun-tasks"` for real execution (see `04_test_all.txt`).

**Note:** CI spec job red-path proof on a real PR (broken manifest → job red) is **deferred** until this branch can be pushed alone; requires push/PR/`gh` blocked per orchestrator scope.

**Note:** `just docker verify` only smoke-tests the user/auth endpoints (`/users`, `/users/login`, `/user`); it does not exercise search, popular, stats, or other slice endpoints.

## Census (verbatim)

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
  CommentCreateTest          ran=5   passed=5   failed=0   skipped=0
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileStatsTest           ran=5   passed=5   failed=0   skipped=0
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

Test counts from `build/test-results/test/*.xml` via `just test census`: **94 ran, 94 passed, 0 failed, 16 skipped**.

## Changes

| File | Description |
|------|-------------|
| `results/04_build_matrix.txt` | Raw output of `just build matrix` |
| `results/04_test_all.txt` | Raw output of `just build gradle "test --rerun-tasks"` |
| `results/04_census.txt` | Raw output of `just test census` |
| `results/04_security_audit.txt` | Raw output of `just security audit` |
| `results/04_docker_verify.txt` | Raw output of `just docker verify` |
| `results/04_spec_run.txt` | Raw output of docker up, spec, comparator, down |
| `results/04_pins.txt` | Workflow SHA pins and runner guard grep |
| `results/04_testing.md` | This evidence document |

Repository under test (`kotlin-ktor-realworld-example-app`):

```
$ git diff --stat
(empty)

$ git status --short --untracked-files=all
(empty)
```

## Commands

### `just build matrix` — exit 0

```
matrix OK: JDK 17 and 21
EXIT_CODE=0
```

Full output: `04_build_matrix.txt`

### `just build gradle "test --rerun-tasks"` — exit 0

Re-run because initial `just test all` showed `:test FROM-CACHE`.

```
BUILD SUCCESSFUL in 1m 4s
5 actionable tasks: 5 executed
EXIT_CODE=0
```

110 test methods executed (94 ran + 16 skipped per census). Full per-test PASSED/SKIPPED log: `04_test_all.txt`.

### `just test census` — exit 0

See Census (verbatim) above. Full output: `04_census.txt`.

### `just security audit` — exit 0

```
static audit: PASSED
EXIT_CODE=0
```

Full output: `04_security_audit.txt`

### `just docker verify` — exit 0

```
smoke: PASSED
stopped
EXIT_CODE=0
```

Full output: `04_docker_verify.txt`

### Spec run — `just docker up`, `just docker spec`, comparator, `just docker down`

`just docker spec` exit 1 (expected: Newman fails on stubbed endpoints). Comparator exit 0.

Newman stats from `spec-api/newman-report.json`:

```
requests: 31
failed requests (transport): 0
assertions total: 112
assertions failed: 16
```

Comparator:

```
18 failed, 18 expected
COMPARATOR_EXIT_CODE=0
```

Full Newman log and commands: `04_spec_run.txt`

### Pins and runner guard — exit 0

8 `uses:` lines, all SHA-pinned with tag comments. `spec-api/run-api-tests.sh` contains neither `set -x` nor `productionready`. Full output: `04_pins.txt`

## Done When

### Commands

| Checkbox | Result | Evidence |
|----------|--------|----------|
| `just build matrix` ends with `matrix OK: JDK 17 and 21` | **pass** | `04_build_matrix.txt` last lines |
| `just test all` passes | **pass** | `04_test_all.txt` BUILD SUCCESSFUL, census 94/94 passed |
| `just test census` saved; no test class fully disabled without reason (D010) | **pass** | `04_census.txt`; only `ProfileControllerTest` fully disabled with class-level `@Ignore("Profile get, follow and unfollow are still stubbed; no slice in this festival enables them per D001")` |
| `just security audit` passes | **pass** | `04_security_audit.txt` static audit PASSED |
| Endpoints changed → `just docker verify` + per-endpoint exercise | **n/a** | This sequence changed CI and spec runner only; `just docker verify` smoke-tests user endpoints (see note above) |

### Behavior

| Checkbox | Result | Evidence |
|----------|--------|----------|
| Every new endpoint has success/422/404/anonymous tests (D003) | **n/a** | No new endpoints in this sequence; prior slices cover search/popular/stats/create/favorite/comment with anonymous-public tests |
| Article/comment endpoints pass `assertNoAuthorSecrets` (D008) | **pass** | Tests in `ArticleCreateTest`, `ArticleSearchTest`, `PopularArticlesTest`, `CommentCreateTest`, `ArticleControllerTest` (favorite/unfavorite); all 94 ran tests green |
| Tests use unique data and assert own rows only (D009) | **pass** | Census green; isolation pattern in enabled tests per prior slices |
| Unverified claims proven or recorded failing | **pass** | LOWER-on-CLOB probe (`LowerOnClobProbeTest`), row persistence (`ArticleCreateTest` b_row_survives_new_app), slug collisions (`SlugTest`) all pass |

### Evidence

| Checkbox | Result | Evidence |
|----------|--------|----------|
| Raw output of each command saved under `results/` | **pass** | `04_*.txt` files listed above |

### Spec-specific (sequence 06)

| Checkbox | Result | Evidence |
|----------|--------|----------|
| 31 requests, 0 transport failures | **pass** | `04_spec_run.txt` Newman stats |
| 112 assertions, 16 failing | **pass** | `04_spec_run.txt` Newman stats |
| Comparator `18 failed, 18 expected` exit 0 | **pass** | `04_spec_run.txt` |
| All workflow `uses:` SHA-pinned with tag comment | **pass** | `04_pins.txt` 8/8 |
| `run-api-tests.sh` has no `set -x` or `productionready` | **pass** | `04_pins.txt` grep no matches |
| CI spec job red on broken manifest (real PR) | **deferred** | Branch stacked; push/PR/`gh` out of scope for this gate |

## Notes

- **Test re-run:** First `just test all` hit Gradle cache (`:test FROM-CACHE`). Evidence re-ran with `--rerun-tasks` per gate instructions.
- **Spec exit codes:** `just docker spec` exits 1 on expected Newman failures; CI uses `|| true` on the collection step and gates on the comparator only (D011).
- **Skipped tests:** 16 skipped, all with `@Ignore` reason strings citing D001 stub scope (D010). `ProfileControllerTest` is the only fully disabled class, with an explicit class-level reason.
- **Anchor drift:** None observed during this verification gate.
- **Repository changes:** None in `kotlin-ktor-realworld-example-app`; gate is read-only verification plus evidence files.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `52e4d5de-6452-407e-a761-471a2988baec`, 09:53:16Z to 09:59:46Z).
Checked against the raw files:

- **All seven evidence files exited 0.** `just build matrix` ends `matrix OK: JDK 17 and 21`; census reads
  **94 ran, 94 passed, 0 failed, 16 skipped**; `static audit: PASSED`; `smoke: PASSED`.
- **Cache honesty, fourth slice running.** The first `just test all` came back FROM-CACHE, so the gate re-ran with
  `--rerun-tasks` and recorded that instead of a no-op.
- **The spec run reproduces the baseline exactly:** 31 requests, 0 transport failures, 112 assertions with 16 failing,
  and the comparator printing `18 failed, 18 expected` at exit 0 — identical to tasks 01-02 and to the orchestrator's
  own independent runs. `just docker spec` itself exits 1, which is correct: newman fails on the expected failures and
  the comparator is the gate.
- **Pins and runner hygiene re-confirmed:** 8/8 `uses:` lines SHA-pinned with tag comments;
  `grep -nE 'set -x|productionready' spec-api/run-api-tests.sh` finds nothing.
- **The gate changed no application code** and left no container running.
- **Both limitations carried forward rather than marked green:** the CI red-path proof is deferred until the branch can
  be pushed alone, and `just docker verify` only smoke-tests the user endpoints.

No new findings from this gate. Gate 06 addresses the three accepted items from `results/05_review.md`.
