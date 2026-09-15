---
fest_type: task
fest_id: 01_comments_schema_and_add_comment.md
fest_name: comments_schema_and_add_comment
fest_parent: 05_user_activity
fest_order: 1
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.811307-06:00
fest_tracking: true
---

# Task: Add comments and POST /articles/{slug}/comments

## Objective

Add the `Comments` table and make `POST /articles/{slug}/comments` create a comment authored by the signed-in user.

## Requirements

- [ ] `Comments : LongIdTable` has a text `body`, `article` → `Articles`, `author` → `Users`, and `created_at`/`updated_at` as longs. It is created in dependency order with the other tables. `CommentRepository`, `CommentService` and `CommentController(commentService)` are wired in the Kodein COMMENT module (`ModulesConfig.kt:30`).
- [ ] A blank body or an unmappable payload → 422; unknown slug → 404; no token → 401. The response is a `CommentDTO` with a `Profile` author (D008)
- [ ] Work happens on branch `feat/user-activity`, created from the fork's `master` after `04_popular_articles` merged (D012)

## Implementation

**Steps**

1. **Branch.**
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c feat/user-activity
   ```
2. **Create** `src/main/kotlin/io/realworld/app/domain/repository/CommentRepository.kt`:
   ```kotlin
   internal object Comments : LongIdTable() {
       val body: Column<String> = text("body")
       val article: Column<EntityID<Long>> = reference("article", Articles)
       val author: Column<EntityID<Long>> = reference("author", Users)
       val createdAt: Column<Long> = long("created_at")
       val updatedAt: Column<Long> = long("updated_at")
   }

   class CommentRepository {
       init {
           transaction {
               // Kodein may construct this before ArticleRepository; create every referenced table here too.
               SchemaUtils.create(Users, Tags, Articles, ArticleTags, ArticleFavorites, Comments)
           }
       }

       fun add(slug: String, authorEmail: String, body: String): Comment = transaction {
           val author = Users.select { Users.email eq authorEmail }.singleOrNull()
               ?: throw NotFoundException("User not found.")
           val articleId = Articles.select { Articles.slug eq slug }.singleOrNull()?.get(Articles.id)
               ?: throw NotFoundException("Article not found.")
           val now = System.currentTimeMillis()
           val id = Comments.insertAndGetId {
               it[Comments.body] = body; it[article] = articleId; it[Comments.author] = author[Users.id]
               it[createdAt] = now; it[updatedAt] = now
           }
           Comment(id = id.value, createdAt = Date(now), updatedAt = Date(now), body = body,
                   author = Profile(author[Users.username], author[Users.bio], author[Users.image], following = false))
       }

       fun countByAuthor(userId: Long): Long = transaction { Comments.select { Comments.author eq userId }.count() }
   }
   ```
   - **`following` is always false** because the comment's author is the caller, and a user cannot follow themselves.
   - **`Comment`'s shape** is `Comment(id, createdAt, updatedAt, body, author)` (`Comment.kt:8`). Its `author` field (`:12`) has been a `Profile?` since `02_article_foundation`.
3. **Create** `src/main/kotlin/io/realworld/app/domain/service/CommentService.kt`:
   ```kotlin
   class CommentService(private val commentRepository: CommentRepository) {
       fun add(slug: String, email: String, comment: Comment): Comment {
           require(comment.body.isNotBlank()) { "Comment body can't be blank." }
           return commentRepository.add(slug, email, comment.body.trim())
       }
   }
   ```
4. **Wire the controller.** In `CommentController.kt`, give the class the constructor the author sketched at `:8`: `class CommentController(private val commentService: CommentService)`. Then replace `add` (`:9-16`):
   ```kotlin
   suspend fun add(ctx: ApplicationCall) {
       val email = ctx.authentication.principal<User>()?.email
       require(!email.isNullOrBlank()) { "User not logged." }
       val slug = requireNotNull(ctx.parameters["slug"]) { "slug is required." }
       val comment = runCatching { ctx.receive<CommentDTO>().comment }
           .getOrElse { throw IllegalArgumentException("Comment is invalid.") }
       requireNotNull(comment) { "Comment is invalid." }
       ctx.respond(CommentDTO(commentService.add(slug, email, comment)))
   }
   ```
   The `runCatching` is there for the same reason as in the article create handler: `Comment.body` is non-null, so a missing body fails in Jackson before validation runs.
5. **Kodein.** In the COMMENT module (`ModulesConfig.kt:30`), bind `CommentRepository()`, `CommentService(instance())` and `CommentController(instance())`. The route needs no change: `POST` comments already sits inside the mandatory auth block (`Router.kt:49`).
6. **Check it against a container.** With `just docker up`:
   - Create an article, then `POST /articles/<slug>/comments` with a token and a body → 200 with `id`, `body` and `author.username`.
   - A blank body → 422.
   - An unknown slug → 404.
   - No token → 401.

**Error paths**

- **`Table "ArticleFavorites" not found`:** a referenced table is missing from this repository's `SchemaUtils.create` call.
- **500 on a payload without `body`:** the `runCatching` around `receive` is missing.

## Done When

- [ ] All requirements met
- [ ] Against a running container, adding a comment returns 200 with a Profile author; a blank body returns 422; an unknown slug returns 404; a request with no token returns 401
