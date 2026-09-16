# Gate 06: Review Results and Iterate

## Finding addressed

**S1 (from `results/05_review.md`, disposition: Accepted):** `CommentCreateTest` had no `missing body returns 422` test. The `CommentController` `runCatching` path that turns a Jackson mapping failure into 422 (instead of 500) was proven only by container evidence; removing that wrapper would not fail the suite.

**Fix:** Added `missing body returns 422` to `CommentCreateTest.kt`, mirroring `ArticleCreateTest`'s equivalent test: register a UUID-suffixed user, create an article, POST raw JSON `{"comment":{}}` to `/articles/{slug}/comments` with a valid token via `postRaw`, assert `HttpStatus.SC_UNPROCESSABLE_ENTITY`.

**Evidence:** `CommentCreateTest > missing body returns 422 PASSED` (see `results/06_comment_create_tests.txt`); census `CommentCreateTest ran=5` and `TOTAL ran=94 ... skipped=16` (see `results/06_census.txt`).

## Changes

| File | Description |
|------|-------------|
| `src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt` | Added `missing body returns 422` test using `postRaw` with `{"comment":{}}` |

```
 .../io/realworld/app/web/controllers/CommentCreateTest.kt     | 11 +++++++++++
 1 file changed, 11 insertions(+)
```

```
 M src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt
```

## Commands

### `just test only CommentCreateTest` (exit 0)

Full output: `results/06_comment_create_tests.txt`

```
CommentCreateTest > raw response has no author secrets PASSED

CommentCreateTest > blank body returns 422 PASSED

CommentCreateTest > missing body returns 422 PASSED

CommentCreateTest > unknown slug returns 404 PASSED

CommentCreateTest > no token returns 401 PASSED

BUILD SUCCESSFUL in 16s
EXIT_CODE=0
```

### `just test all` (exit 0)

Full output: `results/06_test_all.txt`

```
CommentCreateTest > missing body returns 422 PASSED
...
BUILD SUCCESSFUL in 1m 3s
EXIT_CODE=0
```

### `just test census` (exit 0)

Full output: `results/06_census.txt`

```
  CommentCreateTest          ran=5   passed=5   failed=0   skipped=0
  ...
  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
EXIT_CODE=0
```

### `git diff --stat` / `git status --short --untracked-files=all` (exit 0)

See **Changes** section above.

## Done When

Definition of Done (gate 06):

- [x] **All critical findings are fixed** — pass. Review had no critical findings; accepted suggestion S1 is fixed.
- [x] **`just test all` passes after the changes** — pass. `BUILD SUCCESSFUL`, exit 0 (`results/06_test_all.txt`).
- [x] **Code review findings are addressed or explicitly deferred with a reason** — pass. S1 fixed; S2–S4 deferred in `results/05_review.md` (layering boundary, README owned by `07_submission_docs`, cosmetic stub).
- [x] **Ready to commit** — pass. Single focused test addition; suite green; no production code changed.

Test count expectations:

- [x] CommentCreateTest: 4 → 5 ran — pass (`CommentCreateTest ran=5` in census).
- [x] Suite total: 93 → 94 ran — pass (`TOTAL ran=94` in census).
- [x] Skipped unchanged at 16 — pass (`skipped=16` in census).

## Notes

- No anchor drift; `CommentCreateTest.kt` helpers and imports unchanged.
- No production code, author tests, or existing assertions modified.
- Evidence files `06_comment_create_tests.txt` and `06_test_all.txt` were captured with `just build gradle "test --tests '*CommentCreateTest*' --rerun-tasks"` and `just build gradle "test --rerun-tasks"` respectively because a plain `just test all` immediately after `just test only` returned `> Task :test FROM-CACHE` without re-executing tests. The task-specified `just test only` / `just test all` / `just test census` commands were all run successfully during verification (exit 0, expected counts).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `e76b560f-f409-4df3-bea0-e6b399db99e0`, 09:30:43Z to 09:36:09Z).

- **Scope: one test file, no production code.** `git status --porcelain | grep 'src/main/'` returns nothing.
- **The test has the shape that makes it meaningful.** It posts the raw string `{"comment":{}}` through `postRaw` and
  asserts 422. A typed helper could not express this payload at all, because `Comment.body` is non-null — which is
  exactly why the controller needs `runCatching` around `receive`, and why this path could only be proven by a raw
  string. Delete that wrapper now and this test fails with a 500 instead of passing silently.
- **Independent run with the cache off:**
  ```text
  $ just build gradle "cleanTest test --no-build-cache"
  suite_exit=0    BUILD SUCCESSFUL in 57s
    CommentCreateTest cases: ['raw response has no author secrets', 'blank body returns 422',
      'missing body returns 422', 'unknown slug returns 404', 'no token returns 401']
    TOTAL tests=110 ran=94 failed=0 skipped=16
  ```
  `CommentCreateTest` 4 → 5 and the suite 93 → 94 ran, skips unchanged at 16, matching the reported census.
- **The subagent caught its own cached evidence.** Its first `just test all` produced incomplete output from a cache
  hit, so it re-ran without the cache before writing the evidence file.

Every finding from gates 04 and 05 is now closed (S1) or deferred with a recorded reason (S2-S4).
