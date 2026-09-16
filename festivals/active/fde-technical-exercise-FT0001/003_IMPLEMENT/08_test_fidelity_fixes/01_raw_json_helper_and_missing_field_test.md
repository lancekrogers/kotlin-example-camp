---
fest_type: task
fest_id: 01_raw_json_helper_and_missing_field_test.md
fest_name: raw json helper and missing field test
fest_parent: 08_test_fidelity_fixes
fest_order: 1
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-16T12:59:38.405307-06:00
fest_updated: 2026-09-16T13:07:52.404084-06:00
fest_tracking: true
---


# Task: Send raw JSON as raw JSON, and make the missing-field test real

## Objective

Add a request helper whose body parameter is declared `String`, and rewrite
`CommentCreateTest.missing body returns 422` so it actually sends `{"comment":{}}` and actually tests a missing `body`
field.

## Requirements

- [ ] `HttpUtil` gains a method that posts a raw JSON string unchanged. Its body parameter must be declared `String`,
  not `Any`
- [ ] `postRaw(path: String, body: Any)` (`src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt:43`) is left in place
  for the twelve call sites that pass DTOs, and gains a comment stating that a raw JSON string must not be passed to it
  and why
- [ ] `CommentCreateTest.missing body returns 422` uses the new helper and still asserts 422
- [ ] The request body the helper puts on the wire is captured and recorded. A 422 alone is not acceptable evidence,
  because the broken and fixed payloads both return 422
- [ ] No existing assertion is deleted or loosened

## Implementation

**The defect.** `HttpUtil.postRaw` is declared:

```kotlin
fun postRaw(path: String, body: Any): HttpResponse<String> =
    Unirest.post(origin + path).headers(headers).body(body).asString()
```

Kotlin and Java resolve overloads on the **declared** type, so `.body(body)` always binds Unirest's
`body(Object)` overload, never `body(String)`. `body(Object)` serializes its argument with the mapper registered in
`HttpUtil`'s `init` block (`HttpUtil.kt:15-26`), whose `writeValue` is `jacksonObjectMapper().writeValueAsString(value)`.
Given a `String`, that JSON-encodes the string *itself*.

So `postRaw(path, """{"comment":{}}""")` puts this on the wire:

```text
"{\"comment\":{}}"
```

a JSON **string**, where the server expects an object. `ctx.receive<CommentDTO>()` fails on the type mismatch, the
surrounding `runCatching` turns it into `IllegalArgumentException`, and `ErrorExceptionMapping` maps that to 422. The
test passes for a reason unrelated to a missing field.

Both payloads were sent to a running container and are indistinguishable from the outside:

```text
A: {"comment":{}}                    -> 422  {"errors":{"body":["Comment is invalid."]}}
B: "{\"comment\":{}}"                -> 422  {"errors":{"body":["Comment is invalid."]}}
```

That is why this task requires capturing the request body, not just the status.

**Steps**

1. **Branch.** `cd projects/kotlin-ktor-realworld-example-app`, then `camp fresh`, then
   `git switch -c fix/test-fidelity`. `master` already contains every slice, so this branches from a complete tree and
   nothing needs stacking (the D012 stacking deviation that caused the mis-merge in
   `07_submission_docs/results/04_stacked_pr_mismerge_and_recovery.md` must not be repeated: base the PR on `master`).
2. **Add the helper** next to `postRaw` in `HttpUtil.kt`:
   ```kotlin
   // Unirest picks its body() overload from the DECLARED parameter type. A String parameter binds
   // body(String), which sends the bytes unchanged. Use this for hand-written JSON.
   fun postRawJson(path: String, json: String): HttpResponse<String> =
       Unirest.post(origin + path).headers(headers).body(json).asString()
   ```
3. **Comment `postRaw`**, without changing its signature:
   ```kotlin
   // Serializes body through the registered ObjectMapper, which is correct for DTOs. Do NOT pass a raw
   // JSON string: the declared Any parameter binds body(Object), which would JSON-encode the string
   // itself and send "{\"k\":1}" instead of {"k":1}. Use postRawJson for raw JSON.
   fun postRaw(path: String, body: Any): HttpResponse<String> = ...
   ```
4. **Rewrite the test** in `src/test/kotlin/io/realworld/app/web/controllers/CommentCreateTest.kt` (currently at
   `:46-55`). Keep the existing 422 assertion and add the body assertion:
   ```kotlin
   @Test
   fun `missing body returns 422`() {
       val token = UUID.randomUUID().toString().take(8)
       val (http, slug) = registerAndCreateArticle(token)
       val response = http.postRawJson("/articles/$slug/comments", """{"comment":{}}""")
       assertEquals(HttpStatus.SC_UNPROCESSABLE_ENTITY, response.status)
   }
   ```
5. **Capture what went on the wire.** The point of the task is proving the payload changed, and the response cannot
   show that. Use one of these and paste the output into `results/01_raw_json_helper.md`:
   - Preferred: start the app with `just docker up`, then `curl` both payloads exactly as recorded above, showing they
     differ on the wire and agree in response. That reproduces the ambiguity the test was suffering from.
   - Or: add a temporary `println(...)` of the serialized body inside the helper, run only this test, paste the output,
     then remove the `println` and note its removal.
6. **Check the other raw call sites.** `ArticleControllerTest.kt:263` and `:297` call `postRaw(path, "")` for the
   favorite endpoints. Through `body(Object)` that sends `""` — a two-character JSON string — rather than an empty
   body. Those endpoints ignore the body, so behaviour is unaffected. Record the finding; do not change them in this
   task unless a test actually fails.

**Error paths**

- **`postRawJson` does not compile because `body(String)` is ambiguous:** check Unirest's `RequestBodyEntity` API for
  the exact raw-body method on version 1.4.9 and use that; do not cast to `Object` to make it compile, which would
  reintroduce the defect.
- **The rewritten test now fails with 400 or 500 rather than 422:** that is a real finding, not a reason to revert.
  The previous test never exercised this path. Record the actual status and body, and fix the cause.
- **A different test breaks:** `postRaw`'s signature must not have changed. Confirm nothing else was edited.

## Done When

- [ ] All requirements met
- [ ] `grep -n 'postRawJson' src/test` shows the helper defined with a `String` parameter and used by
  `missing body returns 422`; `results/01_raw_json_helper.md` records the wire payload evidence and the recorded 422,
  and `just test census` still shows `ran=94 failed=0 skipped=16` or better