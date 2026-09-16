---
fest_type: task
fest_id: 01_prove_lower_on_clob_body.md
fest_name: prove_lower_on_clob_body
fest_parent: 03_article_search
fest_order: 1
fest_status: completed
fest_autonomy: high
fest_created: 2026-09-15T12:01:26.627195-06:00
fest_updated: 2026-09-16T02:02:59.31848-06:00
fest_tracking: true
---


# Task: Prove LOWER works on the text body column

## Objective

Before any search code exists, prove with a test that `lower(body) LIKE` with an escaped pattern works on H2's TEXT body column.

## Requirements

- [ ] The test runs exactly the expression D004 will use, `Articles.body.lowerCase() like (LikePattern("%", '\\') + LikePattern.ofLiteral(term) + "%")`, against a row whose only match is in `body`.
- [ ] If H2 rejects it, stop: record the error in `results/01_lower_on_clob.md` and propose an amendment to D004, before changing the schema or the query
- [ ] Work happens on branch `feat/article-search`, created from the fork's `master` after `02_article_foundation` merged (D012)

## Implementation

**Why this comes first.** Exposed 0.41.1 maps `text()` to `TEXT` (`vendors/Default.kt:63`), and H2 stores `TEXT` as a CLOB. Whether H2 2.2.224 (`gradle.properties:8`) accepts `LOWER` on a CLOB column inside `LIKE ... ESCAPE` was not verified during planning (D004).

**Steps**

1. Create the branch:
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c feat/article-search
   ```
2. Add `src/test/kotlin/io/realworld/app/domain/repository/LowerOnClobProbeTest.kt`:
   ```kotlin
   package io.realworld.app.domain.repository

   import io.realworld.app.config.DbConfig
   import org.jetbrains.exposed.sql.LikePattern
   import org.jetbrains.exposed.sql.SqlExpressionBuilder.like
   import org.jetbrains.exposed.sql.insert
   import org.jetbrains.exposed.sql.insertAndGetId
   import org.jetbrains.exposed.sql.lowerCase
   import org.jetbrains.exposed.sql.select
   import org.jetbrains.exposed.sql.transactions.transaction
   import org.junit.Assert.assertEquals
   import org.junit.BeforeClass
   import org.junit.Test
   import java.util.UUID

   class LowerOnClobProbeTest {
       companion object {
           @BeforeClass @JvmStatic
           fun db() {
               DbConfig.setup("jdbc:h2:mem:realworld;DB_CLOSE_DELAY=-1;DATABASE_TO_UPPER=false", "sa", "")
               ArticleRepository()
           }
       }

       @Test
       fun `lower on the text body matches case-insensitively with an escaped literal`() {
           val marker = "ZeBrA" + UUID.randomUUID().toString().take(8)
           transaction {
               val user = Users.insertAndGetId {
                   it[email] = "$marker@probe.test"; it[username] = marker; it[password] = "x"
               }
               Articles.insert {
                   it[slug] = marker.lowercase(); it[title] = "no match here"; it[description] = "d"
                   it[body] = "body holds the $marker word"; it[author] = user
                   it[createdAt] = 0L; it[updatedAt] = 0L
               }
           }
           val pattern = LikePattern("%", '\\') + LikePattern.ofLiteral(marker.lowercase()) + "%"
           val hits = transaction { Articles.select { Articles.body.lowerCase() like pattern }.count() }
           assertEquals(1L, hits)
       }
   }
   ```
   - `lowerCase` is declared at the top level (`SQLExpressionBuilder.kt:20`) and `like(LikePattern)` inside `SqlExpressionBuilder` (`:409-410`). If an import does not resolve, fix the import. Never change the expression under test.
   - `LikePattern("%", '\\')` must share its escape character with `ofLiteral`'s default, because `plus` rejects mixed escape characters (`SQLExpressionBuilder.kt:141`).
3. Run `just test only LowerOnClobProbeTest`.
4. Record the outcome, with the raw test output, in `results/01_lower_on_clob.md`.
   - **It passes:** keep the test, since it guards this dependency, and go on to task 02.
   - **It fails with an H2 SQL error:**
     1. Do not change the column type or the query in this task.
     2. Record the exact error, and `fest task block` this task.
     3. Add the options to `002_PLAN/decisions/D004_search_semantics.md` under Consequences, for review before proceeding. Examples: a `CAST(body AS VARCHAR)` custom expression, or H2's `ILIKE`.

**Error paths**

- `Table "Articles" not found`: `ArticleRepository()` did not run its `init`. `@BeforeClass` must construct it.
- A unique-index violation on `slug`: a previous run inserted the same marker. The UUID suffix prevents this, so check that it is applied.

## Done When

- [ ] All requirements met
- [ ] `just test only LowerOnClobProbeTest` passes; or, if it fails, `results/01_lower_on_clob.md` records the exact H2 error and D004 carries the proposed amendment