---
fest_type: task
fest_id: 02_discriminating_percent_test.md
fest_name: discriminating percent test
fest_parent: 08_test_fidelity_fixes
fest_order: 2
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-16T12:59:38.409145-06:00
fest_tracking: true
---

# Task: Make the `%` literal-wildcard tests able to fail

## Objective

Rewrite the two `%` assertions so they fail if `%` in a search term is treated as a wildcard. Both currently pass
whether the escaping works or not.

## Requirements

- [ ] `ArticleSearchRepositoryTest.percent sign in search term is literal` (`:89-99`) discriminates: it fails if `%` is
  not escaped
- [ ] The percent half of `ArticleSearchTest.percent and underscore are literal` (`:106-113`) discriminates in the same
  way. Its underscore half is already sound and must be left as it is
- [ ] Both keep a positive assertion — a term containing `%` still matches a title that really contains `%` — so the
  tests prove escaping rather than merely proving that nothing matches
- [ ] No assertion is deleted or loosened; the underscore coverage stays intact

## Implementation

**Why the current tests cannot fail.** The repository test inserts title `"${token}100% pure_x"` and searches
`"${token}100%"`. `ArticleRepository.search` builds:

```kotlin
val pattern = LikePattern("%", '\\') + LikePattern.ofLiteral(term.lowercase()) + "%"
```

- **Escaped (correct):** `%…100\%%` — needs a literal `%` after `…100`, which the title has. Matches. Count 1.
- **Unescaped (broken):** `%…100%%` — and `%%` collapses to a single `%` under LIKE. Also matches. Count 1.

Both readings give 1, so `assertEquals(1, page.articles.size)` holds either way. The same applies to the HTTP test at
`ArticleSearchTest.kt:110-113`, which asserts `articlesCount == 1` on the identical data shape.

**The fix: add a decoy that only a wildcard reading would match.** Insert a second article whose title would be caught
by `%…100%%` but not by `%…100\%%` — that is, a title starting with the token and `100` but containing **no** literal
`%`. Then a correct implementation returns exactly the percent-bearing article, and a broken one returns both.

**Steps**

1. Work on the `fix/test-fidelity` branch created in task 01.
2. **Repository test** — replace the body of `percent sign in search term is literal`
   (`src/test/kotlin/io/realworld/app/domain/repository/ArticleSearchRepositoryTest.kt:89-99`):
   ```kotlin
   @Test
   fun `percent sign in search term is literal`() {
       val token = UUID.randomUUID().toString()
       val userId = insertUser("user-$token")
       val withPercent = "${token}100% pure_x"
       // Decoy: a wildcard reading of "%" would match this too, an escaped literal "%" must not.
       val withoutPercent = "${token}100XX plain"
       insertArticle(userId, "slug-$token-percent", withPercent, "body", 1000L)
       insertArticle(userId, "slug-$token-decoy", withoutPercent, "body", 900L)

       val page = repo.search("${token}100%", limit = 20, offset = 0, viewerEmail = null)

       assertEquals(1, page.articles.size)
       assertEquals(1L, page.total)
       assertEquals(withPercent, page.articles.single().title)
   }
   ```
   The assertions are unchanged in form; the decoy is what gives them power. With escaping broken, `page.total` is 2
   and `.single()` throws, so the test fails in two independent ways.
3. **HTTP test** — in `src/test/kotlin/io/realworld/app/web/controllers/ArticleSearchTest.kt`, extend only the percent
   half of `percent and underscore are literal` (`:106-113`), leaving the underscore half at `:115-120` untouched:
   ```kotlin
   createArticle("${token}100% pure_x", "body")
   createArticle("${token}100XX plain", "body")   // decoy, matched only by a wildcard reading
   val percentResponse = search("${token}100%")
   assertEquals(HttpStatus.SC_OK, percentResponse.status)
   assertEquals(1, percentResponse.body.articlesCount)
   assertEquals(1, percentResponse.body.articles.size)
   assertEquals("${token}100% pure_x", percentResponse.body.articles.single().title)
   ```
   Note the decoy title must not contain an underscore, or it would interfere with the underscore assertions that
   follow in the same test.
4. **Sanity-check the decoy is actually a decoy.** Confirm by reading the pattern, not by guessing: `%…100%%`
   must match `…100XX plain`, and `%…100\%%` must not. The token prefix keeps both titles isolated per run (D009).

**Note on the live evidence.** The API already demonstrates this discriminatingly: with at least one article present,
`GET /articles/search?q=%` returns `articlesCount: 0`, where an unescaped `%` would have matched everything. That is
recorded in `003_IMPLEMENT/output_specs/requirements.md`. This task moves that property into the automated suite, where
it can catch a regression.

**Error paths**

- **The rewritten test fails against current code:** stop and investigate the escaping rather than adjusting the test.
  That would mean `LikePattern.ofLiteral` is not escaping `%` on this dialect, which is a real defect and the reason
  this test exists.
- **The decoy accidentally matches for an unrelated reason** (for example the term appearing in `body`): `search`
  matches title **or** body, so keep the decoy's body free of the token-plus-`100` sequence. Both inserts above use the
  literal body `"body"`.
- **`.single()` throws in a passing run:** more than one article matched. Print the returned titles before changing
  anything.

## Done When

- [ ] All requirements met
- [ ] Both tests contain a decoy title with no literal `%`, and assert on the exact matched title; the underscore
  assertions are byte-identical to before; `just test census` shows `ran` at 94 or higher with `failed=0` and
  `skipped=16`
