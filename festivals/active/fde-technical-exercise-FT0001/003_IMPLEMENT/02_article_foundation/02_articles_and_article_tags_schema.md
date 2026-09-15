---
fest_type: task
fest_id: 02_articles_and_article_tags_schema.md
fest_name: articles_and_article_tags_schema
fest_parent: 02_article_foundation
fest_order: 2
fest_status: pending
fest_autonomy: high
fest_created: 2026-09-15T12:01:26.526437-06:00
fest_tracking: true
---

# Task: Add the Articles and ArticleTags schema

## Objective

Add the `Articles` and `ArticleTags` tables and a tag get-or-create helper, created idempotently in dependency order.

## Requirements

- [ ] `Articles` has a unique `slug`, `title`, `description`, a text `body`, an `author` referencing `Users`, and `created_at`/`updated_at` as epoch-millisecond longs. `ArticleTags` is keyed on `(article, tag)`, like `Follows` (`UserRepository.kt:38-43`).
- [ ] All tables are created with one `SchemaUtils.create(Users, Tags, Articles, ArticleTags)` call. In Exposed 0.41.1 that call sorts tables by foreign-key references and skips tables that already exist (`SchemaUtils.kt:90,96`)
- [ ] `Tags` gains a get-or-create by name that runs inside the caller's transaction
- [ ] No new dependency is added: timestamps are `long` columns, not `exposed-java-time`

## Implementation

**Why timestamps are longs.** `build.gradle:23-25` includes only exposed-core, exposed-dao and exposed-jdbc. Date columns would need the separate `exposed-java-time` module. Epoch milliseconds avoid a new dependency and convert directly to the `java.util.Date` fields on `Article` (`Article.kt:14-15`).

**Steps**

1. **Create** `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt`. Tables live in their repository file, as `Users`/`Follows` do (`UserRepository.kt:19-43`) and `Tags` does (`TagRepository.kt:9`).
   ```kotlin
   package io.realworld.app.domain.repository

   import org.jetbrains.exposed.dao.id.EntityID
   import org.jetbrains.exposed.dao.id.LongIdTable
   import org.jetbrains.exposed.sql.Column
   import org.jetbrains.exposed.sql.SchemaUtils
   import org.jetbrains.exposed.sql.Table
   import org.jetbrains.exposed.sql.transactions.transaction

   internal object Articles : LongIdTable() {
       val slug: Column<String> = varchar("slug", 255).uniqueIndex()
       val title: Column<String> = varchar("title", 255)
       val description: Column<String> = varchar("description", 1000)
       val body: Column<String> = text("body")
       val author: Column<EntityID<Long>> = reference("author", Users)
       val createdAt: Column<Long> = long("created_at")
       val updatedAt: Column<Long> = long("updated_at")
   }

   internal object ArticleTags : Table() {
       val article: Column<EntityID<Long>> = reference("article", Articles)
       val tag: Column<EntityID<Long>> = reference("tag", Tags)
       override val primaryKey = PrimaryKey(article, tag)
   }

   class ArticleRepository {
       init {
           transaction {
               // One call: SchemaUtils sorts by foreign-key references and skips existing tables,
               // so this is safe no matter which repository Kodein constructs first.
               SchemaUtils.create(Users, Tags, Articles, ArticleTags)
           }
       }
   }
   ```
   **Why `Users` and `Tags` are in this call:** `AppConfig.kt:89-93` installs routes in the order users, profiles, articles, tags, and Kodein constructs `TagRepository` lazily. Without `Tags` here, the `ArticleTags` foreign key could be created before its target table exists.
2. **Add get-or-create to `Tags`.** In `src/main/kotlin/io/realworld/app/domain/repository/TagRepository.kt`, add a function inside `internal object Tags` (`:9`), which already declares `name` unique (`:10`):
   ```kotlin
   /** Ids for [names], inserting any missing tag. Call inside an open transaction. */
   fun idsFor(names: Collection<String>): List<EntityID<Long>> =
       names.map { tagName ->
           select { name eq tagName }.singleOrNull()?.get(id)
               ?: insertAndGetId { it[name] = tagName }
       }
   ```
   Imports: `org.jetbrains.exposed.dao.id.EntityID`, `org.jetbrains.exposed.sql.select`, `org.jetbrains.exposed.sql.insertAndGetId`, `org.jetbrains.exposed.sql.SqlExpressionBuilder.eq`.
3. **Compile:** `just build compile`.
4. **Add a schema test**, `src/test/kotlin/io/realworld/app/domain/repository/ArticleSchemaTest.kt` (plain JUnit, no HTTP).
   - Setup: call `DbConfig.setup("jdbc:h2:mem:realworld;DB_CLOSE_DELAY=-1;DATABASE_TO_UPPER=false", "sa", "")` (the URL from `AppConfig.kt:44`), then construct `ArticleRepository()` **twice** to prove creation is idempotent.
   - Data: inside `transaction { }`, insert a user, then one article.
   - Assert that calling `Tags.idsFor(listOf(name))` twice returns the same id.
   - Assert that inserting a second article with the same slug throws `org.jetbrains.exposed.exceptions.ExposedSQLException`.
   - Use a `UUID` suffix on every name, because the database outlives each test method (D009).
5. **Run:** `just test only ArticleSchemaTest`.

**Error paths**

- **`Table "Tags" not found`, or a foreign-key error at startup:** `Tags` is missing from the `SchemaUtils.create` call.
- **`Unresolved reference: Users` or `Tags`:** both are `internal object`s in the same package and module. Check the file's `package` line.
- **The duplicate-slug insert succeeds:** the unique index is missing from `slug`. Only `slug` is indexed; `body` must not be.

## Done When

- [ ] All requirements met
- [ ] `just test only ArticleSchemaTest` passes: creation is idempotent, get-or-create returns stable ids, and a duplicate slug is rejected
- [ ] `just build compile` succeeds, and the dependencies block of `build.gradle` is unchanged
