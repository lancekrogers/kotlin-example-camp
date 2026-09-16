---
fest_type: task
fest_id: 01_authors_as_profiles.md
fest_name: authors_as_profiles
fest_parent: 02_article_foundation
fest_order: 1
fest_status: completed
fest_autonomy: high
fest_created: 2026-09-15T12:01:26.506093-06:00
fest_updated: 2026-09-15T15:08:04.816106-06:00
fest_tracking: true
---


# Task: Make article and comment authors Profiles

## Objective

Change `Article.author` and `Comment.author` from `User?` to `Profile?`, and add a raw-JSON assertion that proves no author secrets reach a response.

## Requirements

- [ ] `Article.kt:18` and `Comment.kt:12` declare `val author: Profile? = null`, and nothing in `src/main` still types an author as `User` (D008).
- [ ] `HttpUtil` gains raw-response helpers, and a reusable `assertNoAuthorSecrets(json)` is proven to fail on each forbidden field
- [ ] Work happens on branch `feat/article-foundation`, created from an up-to-date fork `master` after `01_ci_pipeline` merged (D012)

## Implementation

**Steps**

1. **Branch.**
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c feat/article-foundation
   ```
2. **Retype both authors.**
   - `src/main/kotlin/io/realworld/app/domain/Article.kt:18`: change `val author: User? = null` to `val author: Profile? = null`. `Profile` lives in the same package (`Profile.kt:5-8`: `username`, `bio`, `image`, `following`), so no import is needed.
   - `src/main/kotlin/io/realworld/app/domain/Comment.kt:12`: the same change.
3. **Confirm nothing else relies on the old type.** `grep -rn "author" src/main/kotlin` should show only these two declarations and commented-out stubs. Test code reads `author?.username`, which `Profile` also has.
4. **Add raw helpers** to `src/test/kotlin/io/realworld/app/web/util/HttpUtil.kt`, next to the typed helpers (`:37`-`:46`). Typed deserialization can't prove a field is *absent*, because Jackson silently ignores unknown fields. Tests need the exact JSON a client receives:
   ```kotlin
   fun getRaw(path: String): HttpResponse<String> =
       Unirest.get(origin + path).headers(headers).asString()

   fun postRaw(path: String, body: Any): HttpResponse<String> =
       Unirest.post(origin + path).headers(headers).body(body).asString()
   ```
5. **Add** `src/test/kotlin/io/realworld/app/web/util/JsonAssertions.kt`. The test client already uses `jacksonObjectMapper` (`HttpUtil.kt:3`).
   ```kotlin
   package io.realworld.app.web.util

   import com.fasterxml.jackson.databind.JsonNode
   import com.fasterxml.jackson.module.kotlin.jacksonObjectMapper
   import org.junit.Assert.assertFalse

   private val FORBIDDEN_AUTHOR_FIELDS = listOf("password", "email", "token")

   /** Fails if any object under an "author" key carries user secrets (D008). */
   fun assertNoAuthorSecrets(rawJson: String) {
       fun walk(node: JsonNode) {
           if (node.isObject) {
               node.fields().forEach { (key, value) ->
                   if (key == "author" && value.isObject) {
                       FORBIDDEN_AUTHOR_FIELDS.forEach { field ->
                           assertFalse("author must not expose '$field': $value", value.has(field))
                       }
                   }
                   walk(value)
               }
           } else if (node.isArray) {
               node.forEach { walk(it) }
           }
       }
       walk(jacksonObjectMapper().readTree(rawJson))
   }
   ```
6. **Prove the check can fail.** Add `src/test/kotlin/io/realworld/app/web/util/JsonAssertionsTest.kt` (plain JUnit, no `AppRule`) with these cases:
   - It passes for `{"article":{"author":{"username":"a","bio":null,"image":null,"following":false}}}`.
   - It throws `AssertionError` for an author carrying `"password":"x"`, and separately for `"email"` and for `"token"`.
   - It throws for a forbidden field on an author nested inside an `articles` array.
7. **Run in Docker.** `just build gradle compileTestKotlin`, then `just test only JsonAssertionsTest`.

**Error paths**

- **`Unresolved reference: Profile`:** the edited file is not in package `io.realworld.app.domain`. Check its `package` line.
- **A negative case passes:** the walker is not recursing into arrays or nested objects. The unit test exists to catch exactly that. Fix the walker; never loosen the test.

## Done When

- [ ] All requirements met
- [ ] `grep -rn 'author: User' src/main/kotlin` returns nothing
- [ ] `just test only JsonAssertionsTest` passes, including the negative cases that must throw