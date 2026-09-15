---
fest_type: task
fest_id: 01_favorites_schema_and_repository.md
fest_name: favorites_schema_and_repository
fest_parent: 04_popular_articles
fest_order: 1
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.729292-06:00
fest_tracking: true
---

# Task: Add favorites storage with idempotent writes

## Objective

Add the `ArticleFavorites` table, idempotent favorite and unfavorite repository operations, and the favorite count and viewer lookups that every article response needs.

## Requirements

- [ ] `ArticleFavorites(user → Users, article → Articles)` has primary key `(user, article)`, following `Follows` (`UserRepository.kt:38-43`), and is created in the same `SchemaUtils.create` call as the other article tables (D005).
- [ ] Favoriting an already-favorited article, or unfavoriting one that is not favorited, changes nothing and raises no error. An unknown slug or email throws `NotFoundException`
- [ ] `toArticles` fills `favoritesCount` and `favorited` for every article it maps, so Search results become accurate too
- [ ] Work happens on branch `feat/popular-articles`, created from the fork's `master` after `03_article_search` merged (D012)

## Implementation

**Steps**

1. **Branch.**
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c feat/popular-articles
   ```
2. **Add the table and create it.** In `ArticleRepository.kt`, add:
   ```kotlin
   internal object ArticleFavorites : Table() {
       val user: Column<EntityID<Long>> = reference("user", Users)
       val article: Column<EntityID<Long>> = reference("article", Articles)
       override val primaryKey = PrimaryKey(user, article)
   }
   ```
   Then extend the existing `init` call to `SchemaUtils.create(Users, Tags, Articles, ArticleTags, ArticleFavorites)`.
3. **Add the write operations** inside `class ArticleRepository`:
   ```kotlin
   fun favorite(email: String, slug: String): Article = transaction {
       val (userId, articleId) = userAndArticle(email, slug)
       val already = !ArticleFavorites.select {
           (ArticleFavorites.user eq userId) and (ArticleFavorites.article eq articleId)
       }.empty()
       if (!already) ArticleFavorites.insert { it[user] = userId; it[article] = articleId }
       requireNotNull(loadBySlug(slug, email))
   }

   fun unfavorite(email: String, slug: String): Article = transaction {
       val (userId, articleId) = userAndArticle(email, slug)
       ArticleFavorites.deleteWhere { (ArticleFavorites.user eq userId) and (ArticleFavorites.article eq articleId) }
       requireNotNull(loadBySlug(slug, email))
   }

   /** Call inside a transaction. */
   private fun userAndArticle(email: String, slug: String): Pair<EntityID<Long>, EntityID<Long>> {
       val userId = Users.select { Users.email eq email }.singleOrNull()?.get(Users.id)
           ?: throw NotFoundException("User not found.")
       val articleId = Articles.select { Articles.slug eq slug }.singleOrNull()?.get(Articles.id)
           ?: throw NotFoundException("Article not found.")
       return userId to articleId
   }
   ```
   - The existence check and the write share one transaction, so a repeated favorite never hits the primary key.
   - `deleteWhere` needs `import org.jetbrains.exposed.sql.SqlExpressionBuilder.eq`, the same import `UserRepository.kt:9` uses for `unfollow` (`:124`).
4. **Fill the favorite fields in `toArticles`.**
   - Look up the viewer as an `EntityID`, so it compares against `ArticleFavorites.user` without a type mismatch:
     `val viewerId = viewerEmail?.let { e -> Users.select { Users.email eq e }.singleOrNull()?.get(Users.id) }`
     If you keep a `Long` for the `Follows` check, use `viewerId?.value` there.
   - After `ids` is computed, add:
   ```kotlin
   val favCount = ArticleFavorites.user.count()
   val counts = ArticleFavorites.slice(ArticleFavorites.article, favCount)
       .select { ArticleFavorites.article inList ids }
       .groupBy(ArticleFavorites.article)
       .associate { it[ArticleFavorites.article] to it[favCount] }
   val favoritedByViewer = if (viewerId == null) emptySet() else
       ArticleFavorites.select { (ArticleFavorites.user eq viewerId) and (ArticleFavorites.article inList ids) }
           .map { it[ArticleFavorites.article] }.toSet()
   ```
   - In the `Article(...)` constructor, set `favoritesCount = counts[row[Articles.id]] ?: 0L` and `favorited = row[Articles.id] in favoritedByViewer`. The fields are declared at `Article.kt:16-17`.
   - Bind `favCount` once and use the same instance in `slice` and in the row read. Exposed matches expressions by equality, and a single instance avoids surprises.
   - This grouped query selects only the grouped column and an aggregate, so H2 accepts it.
5. **Test the repository.** Add `src/test/kotlin/io/realworld/app/domain/repository/ArticleFavoritesRepositoryTest.kt` with an in-memory DB and UUID-named rows. It must show:
   - Favoriting twice leaves one row, with `favoritesCount` 1.
   - Unfavoriting twice raises no error, and the count is 0.
   - An unknown slug throws `NotFoundException`, and so does an unknown email.
   - After viewer A favorites, `favorited` is `true` for A, `false` for B, and `false` with a `null` viewer.
6. **Run** `just test only ArticleFavoritesRepositoryTest`, then `just test only ArticleSearchTest`. Search responses now carry real favorite fields, and nothing there should break.

**Error paths**

- **`ExposedSQLException` (primary key) on the second favorite:** the existence check is not in the same transaction as the insert.
- **H2 reports a column "must be in the GROUP BY list":** `slice` includes a column that is not grouped. Keep only `article` and the count.

## Done When

- [ ] All requirements met
- [ ] `just test only ArticleFavoritesRepositoryTest` passes: writes are idempotent, unknown slug or email throws, and `favorited` depends on the viewer. `just test only ArticleSearchTest` still passes
