# Gate 07: Testing and Verification — Results

Verification-only gate. No application source files were modified.

## Changes

- `results/07_build_matrix.txt` — raw output of `just build matrix`
- `results/07_test_all.txt` — raw output of `just test all` (plus fresh `--rerun-tasks` per-test lines because Gradle cached `:test UP-TO-DATE`)
- `results/07_census.txt` — raw output of `just test census`
- `results/07_security_audit.txt` — raw output of `just security audit`
- `results/07_docker_verify.txt` — raw output of `just docker verify`
- `results/07_testing.md` — this evidence document

```
(no diff — new files only)
```

```
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_build_matrix.txt
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_census.txt
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_docker_verify.txt
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_security_audit.txt
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_test_all.txt
?? festivals/active/fde-technical-exercise-FT0001/003_IMPLEMENT/02_article_foundation/results/07_testing.md
```

## Command summary

| Command | Exit code | Evidence file |
|---------|-----------|---------------|
| `just build matrix` | 0 | `results/07_build_matrix.txt` |
| `just test all` | 0 | `results/07_test_all.txt` |
| `just test census` | 0 | `results/07_census.txt` |
| `just security audit` | 0 | `results/07_security_audit.txt` |
| `just docker verify` | 0 | `results/07_docker_verify.txt` |

## Commands

### `just build matrix` — exit code 0

Ends with `matrix OK: JDK 17 and 21`. JDK 21 leg ran all 50 tests (31 passed, 19 skipped, 0 failed). Full output: `results/07_build_matrix.txt`.

### `just test all` — exit code 0

Gradle reported `:test UP-TO-DATE` (cached). BUILD SUCCESSFUL. Fresh per-test output captured via `just build gradle "test --rerun-tasks"` in the same session: 50 completed, 31 passed, 19 skipped, 0 failed. Full output: `results/07_test_all.txt`.

### `just test census` — exit code 0

```
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=7   passed=7   failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=31 passed=31 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

Full output: `results/07_census.txt`.

### `just security audit` — exit code 0

`static audit: PASSED`. Full output: `results/07_security_audit.txt`.

### `just docker verify` — exit code 0

`smoke: PASSED` (register, login, get user, auth error paths). Does not exercise POST /articles; see `results/05_wire_create_article_endpoint.md` for that. Full output: `results/07_docker_verify.txt`.

## Done When

### Commands

- [x] **`just build matrix` ends with `matrix OK: JDK 17 and 21`** — **pass** (`results/07_build_matrix.txt` last line)
- [x] **`just test all` passes** — **pass** (exit 0; 31 passed / 19 skipped / 0 failed on fresh run)
- [ ] **`just test census` saved; no test class fully disabled without a reason (D010)** — **fail** — `CommentControllerTest` and `ProfileControllerTest` remain class-level `@Ignore` with no reason string (`CommentControllerTest.kt:14`, `ProfileControllerTest.kt:13`); census flags both as `entire class disabled`
- [x] **`just security audit` passes** — **pass** (`static audit: PASSED`)
- [x] **Sequence added endpoints: `just docker verify` passes; each new endpoint exercised in container** — **pass** — `just docker verify` exit 0; POST /articles exercised in `results/05_wire_create_article_endpoint.md` (200 with token, 401 without, 422 missing body/blank title, ISO dates, secret-free author)

### Behavior

- [x] **Every new endpoint has tests for success, validation failures (422), not-found (404) where applicable, anonymous access where public (D003)** — **pass** — POST /articles (auth-required, not public): success `ArticleControllerTest > create article`; 422 `blank title returns 422`, `missing body returns 422`; 401 `create without token returns 401`; duplicate slug `duplicate title slug ends with -2`. GET /tags: `TagControllerTest > get all tags`. 404 n/a for POST /articles; no new public read endpoints in this sequence.
- [x] **Every endpoint returning an article or comment passes `assertNoAuthorSecrets` on raw JSON (D008)** — **pass** — `ArticleCreateTest > raw response has no author secrets and ISO createdAt` calls `assertNoAuthorSecrets(rawJson)` on POST /articles response
- [x] **New tests create uniquely named data and assert only on rows they created (D009)** — **pass** — all `ArticleCreateTest` methods use `UUID.randomUUID()` suffixes; persistence probe asserts only its own slug in `b_row_survives_new_app`
- [x] **Every unverified claim this sequence depends on is proven by a test or recorded as failing** — **pass** — D009 row persistence confirmed in `results/06_persistence.md` (`a_writes_row` / `b_row_survives_new_app`); slug rules covered by `SlugTest` and `ArticleServiceTest`; H2/LOWER not in scope for this slice

### Evidence

- [x] **Raw output of each command saved under `results/`** — **pass** (six files listed in Changes)

## Notes

- **D010 finding (iterate gate):** Per D010, `ProfileControllerTest` should carry a class-level reason string and out-of-scope tests should use method-level `@Ignore("…")`. `CommentControllerTest` and `ProfileControllerTest` still use bare `@Ignore` at class level with no reason. Not fixed in this gate per instructions.
- **`just test all` cache:** First and final `just test all` runs reported `:test UP-TO-DATE`. Per-test names and counts taken from `just build gradle "test --rerun-tasks"` and JDK 21 matrix leg.
- **`just docker verify` scope:** Built image and smoke-tested user auth only. POST /articles container proof lives in task 05 results (`results/05_wire_create_article_endpoint.md`), including orchestrator re-verification.
- **Anchor drift:** None encountered during this verification gate.
- **Behavior test name index:**

| Check | Test name |
|-------|-----------|
| 401 without token | `ArticleCreateTest > create without token returns 401` |
| 422 blank title | `ArticleCreateTest > blank title returns 422` |
| 422 missing body | `ArticleCreateTest > missing body returns 422` |
| Duplicate slug `-2` | `ArticleCreateTest > duplicate title slug ends with -2` |
| Raw-JSON author leak | `ArticleCreateTest > raw response has no author secrets and ISO createdAt` |
| ISO `createdAt` string | same test (`assertTrue(..., rawJson.contains("\"createdAt\":\""))`) |
| Success path (author test) | `ArticleControllerTest > create article` |
| Container POST /articles | `results/05_wire_create_article_endpoint.md` |

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `c831dde4-8541-4203-ac3b-729d25611577`, 07:41:28Z to 07:45:12Z).
Checked against the raw evidence files rather than the summary:

- **Every command exited 0.** `07_build_matrix.txt`, `07_test_all.txt`, `07_census.txt`, `07_security_audit.txt` and
  `07_docker_verify.txt` each record their exit code; `07_docker_verify.txt` ends with a clarifying note after it.
- **`just build matrix`** ends with `matrix OK: JDK 17 and 21` (`07_build_matrix.txt:211`).
- **Fresh test execution, not a cache hit.** The first `just test all` reported `:test UP-TO-DATE`, so the gate re-ran it
  with `--rerun-tasks`: `50 tests completed, 31 passed, 19 skipped, 0 failed` (`07_test_all.txt:55`). That matches the
  orchestrator's own independent `cleanTest test --no-build-cache` run during task 06 verification.
- **Census matches** the per-class XML the orchestrator parsed: 31 ran, 31 passed, 0 failed, 19 skipped.
- **`just security audit`** ends `static audit: PASSED`, including the wrapper-jar checks and the pinned distribution
  checksum.
- **`just docker verify`** ends `smoke: PASSED` with all seven smoke checks ok, including "register returned no password
  material". `docker ps` afterwards shows no `ktor` container, so the recipe's teardown ran.
- **The gate changed no application code.** `git status --untracked-files=all` on the project is empty; the only new
  files are this sequence's `results/07_*`.
- **The gate's own limitation is stated honestly, and is correct.** `just docker verify` runs `just docker smoke`
  (`.justfiles/docker.just:120-125`), which exercises only the user endpoints. Container evidence for `POST /articles`
  comes from `results/05_wire_create_article_endpoint.md`: the subagent's run plus the orchestrator's independent run
  (200 with a token, 401 without, 422 for a missing `body` and for a blank title, clean author JSON, ISO `createdAt`).

### Findings carried to gate 09

1. **D010 fails (also raised by gate 08 as critical).** `CommentControllerTest` and `ProfileControllerTest` are fully
   disabled with a bare `@Ignore`. Note that `just test census` flags a class from its counts, not from its reason
   string, so the `<-- entire class disabled` marker will persist after the fix; the reason must be verified in source.
2. **Gate 08 suggestion S5 accepted:** add an HTTP-level 422 test for a blank `description`, which is currently proven
   only at the service level.
