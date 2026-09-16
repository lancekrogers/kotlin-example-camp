# Evidence: raw-JSON helper and the missing-field test

## Changes

| File | Change |
|---|---|
| `src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt` | added `postRawJson(path: String, json: String)`; commented `postRaw` to say a raw JSON string must not be passed to it |
| `src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt` | `missing body returns 422` now uses `postRawJson`; added `postRawJson sends the object rather than a quoted string` |

`postRaw`'s signature is unchanged, so the twelve DTO call sites keep the object-serializing path, which is correct
for them.

## How the wire payload was proven, and why not with a println

The task offered a `println` of the serialized body. That would only have echoed the argument, not what Unirest did
with it. A stronger discriminator exists: with a **valid** payload the two overloads diverge observably.

- Sent raw via `body(String)`: the server receives `{"comment":{"body":"..."}}`, accepts it, returns **200**.
- Sent via `body(Object)`: the mapper JSON-encodes the string, the server receives `"{\"comment\":...}"`, which is a
  type mismatch, returning **422**.

So the guard test asserts 200 and the echoed comment body. That makes the helper's typing falsifiable in the suite
itself, rather than in a transcript.

## Red: the defect reintroduced

`postRawJson`'s parameter changed from `String` to `Any` — that alone reproduces the original bug, since the overload
is chosen from the declared type.

```text
CommentCreateTest > postRawJson sends the object rather than a quoted string FAILED
    java.lang.AssertionError: expected:<200> but was:<422>
CommentCreateTest > raw response has no author secrets PASSED
CommentCreateTest > blank body returns 422 PASSED
CommentCreateTest > missing body returns 422 PASSED
CommentCreateTest > unknown slug returns 404 PASSED
CommentCreateTest > no token returns 401 PASSED
> Task :test FAILED
BUILD FAILED in 16s
```

Two things to read here. The guard test fails, so the `String` typing is load-bearing. And
`missing body returns 422` **still passes** — it cannot detect the defect, which is exactly how the original problem
went unnoticed through a full review and merge.

## Green: reverted, with the cache disabled

The first green run after reverting printed `BUILD SUCCESSFUL` with no test lines at all — Gradle served a cached
`:test`, because the inputs had returned to a previously built state. A cached pass is not evidence, so the run was
repeated with `--rerun-tasks --no-build-cache`:

```text
$ just build gradle "test --tests '*CommentCreateTest*' --rerun-tasks --no-build-cache"
CommentCreateTest > postRawJson sends the object rather than a quoted string PASSED
CommentCreateTest > raw response has no author secrets PASSED
CommentCreateTest > blank body returns 422 PASSED
CommentCreateTest > missing body returns 422 PASSED
CommentCreateTest > unknown slug returns 404 PASSED
CommentCreateTest > no token returns 401 PASSED
BUILD SUCCESSFUL in 36s
```

`grep -n 'fun postRawJson'` confirms the parameter is `String` again.

## The other raw call sites, checked not changed

`ArticleControllerTest.kt:263` and `:297` pass `""` to `postRaw` for the favorite endpoints. Through `body(Object)`
that sends `""` — a two-character JSON string — rather than an empty body. Both endpoints ignore the request body, and
both tests pass, so this is recorded and left alone rather than changed opportunistically.

## Production behaviour was never wrong

A genuinely missing `body` field returns `422 {"errors":{"body":["Comment is invalid."]}}`, confirmed earlier against a
running container and now by the corrected test. This task fixed a test, not a bug.
