# Evidence: both directions recorded for both tests

## Why this task exists

A test observed only green is indistinguishable from a test that cannot fail — which is the defect this sequence set
out to remove. Replacing two unfalsifiable tests with two more would have been the easiest way to make things worse, so
each changed test was run against deliberately broken behaviour first.

## Summary

| Test | Broken behaviour introduced | Recorded failure | Restored |
|---|---|---|---|
| `postRawJson sends the object rather than a quoted string` | helper parameter `String` → `Any` | `expected:<200> but was:<422>` | `03_rawjson_green.txt` |
| `percent sign in search term is literal` | escaping removed from `ArticleRepository.search` | `expected:<1> but was:<2>` | `03_percent_green.txt` |
| `percent and underscore are literal` (HTTP) | same | `expected:<1> but was:<2>` | `03_percent_green.txt` |
| `underscore in search term is literal` | same | `expected:<0> but was:<1>` | `03_percent_green.txt` |

The fourth row was not changed by this sequence. It failed alongside the others, which independently confirms it was
discriminating all along — the reason the shipped escaping was never actually broken.

Raw captures: `03_rawjson_red.txt`, `03_rawjson_green.txt`, `03_percent_red.txt`, `03_percent_green.txt`.

## The one place where red is not available

Breaking the raw-JSON helper does **not** change what `missing body returns 422` reports. That test passed in the red
run too:

```text
CommentCreateTest > postRawJson sends the object rather than a quoted string FAILED
    java.lang.AssertionError: expected:<200> but was:<422>
CommentCreateTest > missing body returns 422 PASSED
```

Both payloads return an identical `422 {"errors":{"body":["Comment is invalid."]}}`, so no assertion on that request
could ever detect the defect. That is why the guard is a separate test using a **valid** payload, where the two
overloads diverge observably. The table above records the guard failing; the missing-field test's own correctness rests
on it now sending real JSON, which the guard proves.

## Cache discipline

Every run used `--rerun-tasks --no-build-cache`. The first green run after reverting the helper printed
`BUILD SUCCESSFUL` with **no test lines at all** — Gradle served a cached `:test` because the inputs had returned to a
previously built state. A cached pass is not evidence of anything, and this is the second time in this festival that a
cache hit nearly passed for a verification.

## Suite and gate

```text
  TOTAL ran=95 passed=95 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

95 running, up from 94: the new helper guard. `failed=0` and `skipped=16` are unchanged, so nothing was weakened and no
test was lost.

```text
  ok    only port 8080 listening
  ok    token forged with the old committed key rejected (401)
=== gate: PASSED ===
```

## Nothing temporary survived

```console
$ git status --short
 M AGENT_WORKLOG.md
 M src/test/kotlin/io/realworld/app/domain/repository/ArticleSearchRepositoryTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt
 M src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt
 M src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt

$ grep -rn 'TEMPORARY' src/main src/test --include='*.kt'
  none
```

Only the four intended test files and the work log are modified; no production source is touched. The single
`println` in the tree is a pre-existing `System.err.println` in `JwtProvider.kt:64`, not introduced here.

## Work log updated

`AGENT_WORKLOG.md`'s deferred section now records both items as fixed, with how each is falsifiable, and keeps the R8
`unfollow` bug, which remains genuinely deferred and unreachable. 90 lines, within the ~150 ceiling.
