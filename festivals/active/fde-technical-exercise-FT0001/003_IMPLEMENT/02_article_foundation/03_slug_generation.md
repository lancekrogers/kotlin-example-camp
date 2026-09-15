---
fest_type: task
fest_id: 03_slug_generation.md
fest_name: slug_generation
fest_parent: 02_article_foundation
fest_order: 3
fest_status: pending
fest_autonomy: high
fest_created: 2026-09-15T12:01:26.547177-06:00
fest_tracking: true
---

# Task: Implement slug generation

## Objective

Implement D009's slug rule as a pure function plus a uniqueness helper, with unit tests for every case.

## Requirements

- [ ] `String.toSlugBase()` applies NFD normalization and strips combining marks, lowercases, collapses every run of characters outside `[a-z0-9]` into `-`, trims `-`, and falls back to `article` when nothing is left (D009).
- [ ] `uniqueSlug(base, isTaken)` treats the reserved words `search` and `feed` as taken (D003), then returns the first free slug from `base`, `base-2`, `base-3`, and so on
- [ ] Unit tests cover kebab case, punctuation runs, diacritics, an all-symbol title, both reserved words, and successive collisions

## Implementation

**Steps**

1. **Create** `src/main/kotlin/io/realworld/app/ext/Slug.kt`, next to the existing `ext/String.kt`:
   ```kotlin
   package io.realworld.app.ext

   import java.text.Normalizer

   /** Slugs an article may not use: constant route segments shadow them (D003). */
   val RESERVED_SLUGS = setOf("search", "feed")

   private val COMBINING_MARKS = Regex("\\p{M}+")
   private val NON_SLUG_CHARS = Regex("[^a-z0-9]+")

   fun String.toSlugBase(): String {
       val stripped = Normalizer.normalize(this, Normalizer.Form.NFD).replace(COMBINING_MARKS, "")
       val slug = stripped.lowercase().replace(NON_SLUG_CHARS, "-").trim('-')
       return slug.ifEmpty { "article" }
   }

   /** The first free slug: [base], then base-2, base-3, and so on, never a reserved word. */
   fun uniqueSlug(base: String, isTaken: (String) -> Boolean): String {
       fun taken(candidate: String) = candidate in RESERVED_SLUGS || isTaken(candidate)
       if (!taken(base)) return base
       return generateSequence(2) { it + 1 }.map { "$base-$it" }.first { !taken(it) }
   }
   ```
   `lowercase()` is available because the project uses Kotlin 1.9.25 (`gradle.properties:3`).
2. **Create** `src/test/kotlin/io/realworld/app/ext/SlugTest.kt` (plain JUnit) with these assertions:
   - `"How to train your dragon".toSlugBase() == "how-to-train-your-dragon"`
   - `"slug test".toSlugBase() == "slug-test"`, and `"slug test 2".toSlugBase() == "slug-test-2"`. These are the author's own expectations (`ArticleControllerTest.kt:198`, `:222`).
   - `"Hello,   World!!  ".toSlugBase() == "hello-world"`
   - `"Café Crème".toSlugBase() == "cafe-creme"`
   - `"!!!".toSlugBase() == "article"`
   - `uniqueSlug("search") { false } == "search-2"`, and `uniqueSlug("feed") { false } == "feed-2"`
   - With `taken = setOf("a", "a-2")`: `uniqueSlug("a") { it in taken } == "a-3"`
3. **Run:** `just test only SlugTest`.

**Error paths**

- **The diacritics case returns `caf-cr-me`:** the non-slug replacement ran before normalization. Normalize first, then lowercase, then replace.
- **A test hangs:** the `isTaken` lambda always returns true. That is a bug in the test, not in `uniqueSlug`, whose sequence is lazy.

## Done When

- [ ] All requirements met
- [ ] `just test only SlugTest` passes with every case listed above
