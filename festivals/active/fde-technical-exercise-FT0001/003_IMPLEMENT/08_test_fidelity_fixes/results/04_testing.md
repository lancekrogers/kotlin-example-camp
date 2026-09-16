# Gate: Testing and Verification

## `just build matrix` — and why its own output was not accepted

The recipe reported success:

```text
BUILD SUCCESSFUL in 11s
10 actionable tasks: 7 executed, 3 from cache
matrix OK: JDK 17 and 21
```

But inspecting the log showed the tests had not run on either JDK:

```text
   2 > Task :test FROM-CACHE
   2 > Task :testClasses UP-TO-DATE
```

This is the identical false pass recorded in `01_ci_pipeline` earlier in this festival — `matrix OK` printed while
`:test` came from cache. An 11-second matrix across two JDKs is not a matrix. Re-run per JDK with the cache genuinely
disabled (`build gradle` takes the JDK from the `JDK` env var):

```console
$ JDK=17 just build gradle "build --rerun-tasks --no-build-cache"
  FROM-CACHE lines: 0
  tests passed: 95
BUILD SUCCESSFUL in 1m 39s

$ JDK=21 just build gradle "build --rerun-tasks --no-build-cache"
  FROM-CACHE lines: 0
  tests passed: 95
BUILD SUCCESSFUL in 1m 22s
```

95 tests actually executed on each JDK, zero cache hits, both green. Full logs in `04_matrix_jdk17.txt` and
`04_matrix_jdk21.txt`.

## `just test all`

Run as `just build gradle "test --rerun-tasks --no-build-cache"`, exit 0.

## `just test census`

```text
  TOTAL ran=95 passed=95 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

Saved as `04_census.txt`. 95 running, up one from the previous 94 — the new `postRawJson` guard test. `failed=0` and
`skipped=16` unchanged, so no test was weakened or lost.

Every one of the 16 skips is an upstream `@Ignore` carrying a D001 reason naming the stubbed endpoint; no test class is
fully disabled without one (D010).

## `just security audit`

```text
  ok    no mavenLocal()
  ok    all versions pinned
  ok    Gradle distribution checksum pinned
  ok    no hardcoded credentials in src/main
  ok    wrapper jar contains only stock Gradle classes
  ok    wrapper jar hardcodes no download URLs
  ok    sha256 7d3a4ac4de1c32b59bc6a4eb8ecb8e612ccd0cf1ae1e99f66902da64df296172
static audit: PASSED
```

## Endpoints

**Not applicable.** This sequence changed no production code — only four test files and the work log. `git status`
confirms `src/main` is untouched. The endpoint checks the gate lists apply to sequences that add or change endpoints.

`just docker verify` nevertheless ran as part of `just gate`:

```text
  ok    only port 8080 listening
  ok    token forged with the old committed key rejected (401)
=== gate: PASSED ===
```

## Behaviour verification specific to this sequence

The relevant proof is not that the new tests pass but that they can fail. Both were recorded failing against
deliberately broken behaviour and passing against the real code — see `results/03_verify_both_directions.md` and the
four red/green captures beside it.
