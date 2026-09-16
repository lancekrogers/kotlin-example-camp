---
fest_type: task
fest_id: 04_article_repository_and_service_create.md
fest_name: article_repository_and_service_create
fest_parent: 02_article_foundation
fest_order: 4
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.566494-06:00
fest_updated: 2026-09-16T01:31:47.018452-06:00
fest_tracking: true
---


# Task: Add article create to the repository and service

## Objective

Give `ArticleRepository` create, find-by-slug and article mapping, and add `ArticleService.create` with validation, wired through Kodein.

## Requirements

- [ ] The mapping returns `Article` with its tags, `createdAt`/`updatedAt` as `Date`, and `author` as a `Profile` built only from `Users` `username`, `bio` and `image`. `following` is true only when the viewer follows the author (D008).
- [ ] `ArticleService.create(email, article)` rejects a blank title, description or body with `require(...)`, which becomes 422 through `ErrorExceptionMapping.kt:38`. It also trims tags, drops blank and duplicate tags, and throws `NotFoundException` when the author email is unknown
- [ ] The Kodein ARTICLE module (`ModulesConfig.kt:24`) binds `ArticleRepository` and `ArticleService`

## Implementation

**Steps**

1. **Add create, find and mapping.** Inside `class ArticleRepository` (task 02), add:
   ```kotlin
   fun create(authorEmail: String, article: Article, slugBase: String): Article = transaction {
       val authorRow = Users.select { Users.email eq authorEmail }.singleOrNull()
           ?: throw NotFoundException("Author not found.")
       val slug = uniqueSlug(slugBase) { candidate -> !Articles.select { Articles.slug eq candidate }.empty() }
       val now = System.currentTimeMillis()
       val articleId = Articles.insertAndGetId {
           it[Articles.slug] = slug
           it[title] = requireNotNull(article.title)
           it[description] = requireNotNull(article.description)
           it[body] = article.body
           it[author] = authorRow[Users.id]
           it[createdAt] = now
           it[updatedAt] = now
       }
       Tags.idsFor(article.tagList).forEach { tagId ->
           ArticleTags.insert { row -> row[ArticleTags.article] = articleId; row[ArticleTags.tag] = tagId }
       }
       requireNotNull(loadBySlug(slug, viewerEmail = authorEmail))
   }

   fun findBySlug(slug: String, viewerEmail: String?): Article? = transaction { loadBySlug(slug, viewerEmail) }

   /** Call inside a transaction. */
   internal fun loadBySlug(slug: String, viewerEmail: String?): Article? =
       (Articles innerJoin Users).select { Articles.slug eq slug }.singleOrNull()
           ?.let { toArticles(listOf(it), viewerEmail).single() }

   /** Maps article+author rows to domain articles, loading all tags in one query. Call inside a transaction. */
   internal fun toArticles(rows: List<ResultRow>, viewerEmail: String?): List<Article> {
       if (rows.isEmpty()) return emptyList()
       val ids = rows.map { it[Articles.id] }
       val tagsByArticle = (ArticleTags innerJoin Tags).select { ArticleTags.article inList ids }
           .groupBy({ it[ArticleTags.article] }, { it[Tags.name] })
       val viewerId = viewerEmail?.let { email -> Users.select { Users.email eq email }.singleOrNull()?.get(Users.id)?.value }
       return rows.map { row ->
           val authorId = row[Articles.author].value
           val following = viewerId != null &&
               !Follows.select { (Follows.user eq authorId) and (Follows.follower eq viewerId) }.empty()
           Article(
               slug = row[Articles.slug],
               title = row[Articles.title],
               description = row[Articles.description],
               body = row[Articles.body],
               tagList = tagsByArticle[row[Articles.id]].orEmpty().sorted(),
               createdAt = Date(row[Articles.createdAt]),
               updatedAt = Date(row[Articles.updatedAt]),
               author = Profile(row[Users.username], row[Users.bio], row[Users.image], following)
           )
       }
   }
   ```
   - **Join.** `Articles innerJoin Users` resolves through the single foreign key `Articles.author`. `Users.email` is used only inside filters, and the password column is never read.
   - **Follow direction.** It matches `follow()` (`UserRepository.kt:112-122`): `user` is the account being followed, and `follower` is the viewer.
   - **Favorites.** `favorited` and `favoritesCount` keep their defaults until `04_popular_articles`.
   - **Imports:**
     - `io.realworld.app.domain.Article`, `io.realworld.app.domain.Profile`
     - `io.realworld.app.domain.exceptions.NotFoundException`, `io.realworld.app.ext.uniqueSlug`
     - `java.util.Date`
     - `org.jetbrains.exposed.sql.ResultRow`, `select`, `insert`, `insertAndGetId`, `and`
     - `SqlExpressionBuilder.eq`, `SqlExpressionBuilder.inList`
2. **Create** `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt`, shaped like `UserService` (`UserService.kt:11`):
   ```kotlin
   class ArticleService(private val articleRepository: ArticleRepository) {
       fun create(email: String, article: Article): Article {
           val title = article.title?.trim()
           val description = article.description?.trim()
           require(!title.isNullOrEmpty()) { "Article title can't be blank." }
           require(!description.isNullOrEmpty()) { "Article description can't be blank." }
           require(article.body.isNotBlank()) { "Article body can't be blank." }
           val tags = article.tagList.map { it.trim() }.filter { it.isNotEmpty() }.distinct()
           val clean = article.copy(title = title, description = description, tagList = tags)
           return articleRepository.create(email, clean, title.toSlugBase())
       }
   }
   ```
3. **Wire Kodein.** In `ModulesConfig.kt:24`, add `bind() from singleton { ArticleRepository() }` and `bind() from singleton { ArticleService(instance()) }` to the ARTICLE module. Leave `ArticleController()` alone for now; task 05 gives it a constructor argument.
4. **Test the service directly.** Add `src/test/kotlin/io/realworld/app/domain/service/ArticleServiceTest.kt`, which calls the service without HTTP.
   - Setup: in-memory `DbConfig.setup(...)`, a real `ArticleRepository()`, and a uniquely named user created through `UserRepository().create(User(email = ..., username = ..., password = "x"))`.
   - A blank title, a blank description and a blank body each throw `IllegalArgumentException`.
   - Tags `[" x ", "x", "", "y"]` come back as `["x", "y"]`.
   - A second article with the same title gets a slug ending in `-2`.
5. **Run:** `just test only ArticleServiceTest`.

**Error paths**

- **`NoSuchElementException` from `single()` right after create:** the load ran outside the insert's transaction, or the join found no author row. Create and load must share one transaction.
- **An unknown author email creates an article anyway:** the `Users.select` result is not checked. It must throw `NotFoundException`.

## Done When

- [ ] All requirements met
- [ ] `just test only ArticleServiceTest` passes, covering blank-field rejection, tag normalization, and a `-2` slug on a duplicate title
- [ ] `ArticleRepository.kt` reads no password column, and `Users.email` appears only inside `select { }` filters