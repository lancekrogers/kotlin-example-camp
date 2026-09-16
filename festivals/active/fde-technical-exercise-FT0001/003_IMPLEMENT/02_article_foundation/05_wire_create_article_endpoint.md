---
fest_type: task
fest_id: 05_wire_create_article_endpoint.md
fest_name: wire_create_article_endpoint
fest_parent: 02_article_foundation
fest_order: 5
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.586709-06:00
fest_updated: 2026-09-16T01:35:37.941731-06:00
fest_tracking: true
---


# Task: Wire POST /articles

## Objective

Make `POST /articles` create an article for the signed-in user and respond with `ArticleDTO`, turning malformed bodies into 422 and dates into ISO-8601 strings.

## Requirements

- [ ] `ArticleController` takes `ArticleService` in its constructor, and `create` responds with `ctx.respond(ArticleDTO(created))`, following `UserController.kt:17`. The route stays inside the mandatory `authenticate` block (`Router.kt:45`, `:66`).
- [ ] A body Jackson cannot map, for example one missing `body`, returns 422, not 500
- [ ] Dates serialize as ISO-8601 strings
- [ ] The other stubbed `ArticleController` methods still compile and stay stubbed

## Implementation

**Steps**

1. **Controller.** In `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt`, the class currently reads `class ArticleController {` (`:8`), with the intended signature commented out at `:9`.
   - Change the declaration to `class ArticleController(private val articleService: ArticleService) {`.
   - Replace the stub `create` (`:40`) with:
   ```kotlin
   suspend fun create(ctx: ApplicationCall) {
       val email = ctx.authentication.principal<User>()?.email
       require(!email.isNullOrBlank()) { "User not logged." }
       val article = runCatching { ctx.receive<ArticleDTO>().article }
           .getOrElse { throw IllegalArgumentException("Article is invalid.") }
       requireNotNull(article) { "Article is invalid." }
       ctx.respond(ArticleDTO(articleService.create(email, article)))
   }
   ```
   - **Why `runCatching`.** Ktor 1.2.3's Jackson converter throws a Jackson mapping exception when a non-null field such as `Article.body` (`Article.kt:12`) is missing. That exception is not an `IllegalArgumentException`, so without this it would reach the catch-all 500 handler (`ErrorExceptionMapping.kt:41`).
   - **Principal lookup.** It mirrors `UserController.kt:42`.
   - **Imports:** `io.ktor.auth.authentication`, `io.ktor.response.respond`, `io.realworld.app.domain.User`, `io.realworld.app.domain.service.ArticleService`.
   - **Route.** `Router.kt:66` calls `articleController.create(this.context)` and ignores the result, which is correct now that the controller responds itself.
2. **Kodein.** In `ModulesConfig.kt`, change the ARTICLE module's controller binding to `ArticleController(instance())`.
3. **ISO-8601 dates.** The `jackson { }` block in `src/main/kotlin/io/realworld/app/config/AppConfig.kt:71-74` is empty. Replace it with:
   ```kotlin
   install(ContentNegotiation) {
       jackson {
           disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
           dateFormat = StdDateFormat().withColonInTimeZone(true)
       }
   }
   ```
   Imports: `com.fasterxml.jackson.databind.SerializationFeature` and `com.fasterxml.jackson.databind.util.StdDateFormat`. User responses carry no dates (`User.kt:62`), so nothing already shipped changes.
4. **Try it in a container.**
   ```bash
   just docker up
   B=http://localhost:18080
   ```
   - Register and log in exactly as the smoke recipe does (`.justfiles/docker.just:80-98`), which gives you `$token`.
   - Create an article:
   ```bash
   curl -s -X POST $B/articles -H 'Content-Type: application/json' -H "Authorization: Token $token" \
     -d '{"article":{"title":"Hello World","description":"d","body":"b","tagList":["x"]}}'
   curl -s -o /dev/null -w '%{http_code}\n' -X POST $B/articles -H 'Content-Type: application/json' \
     -H "Authorization: Token $token" -d '{"article":{"title":"t","description":"d"}}'
   just docker down
   ```
   - Expect the first call to return 200 with `"slug":"hello-world"`, `"tagList":["x"]`, an ISO `createdAt`, and an `author` object without `password`, `email` or `token`.
   - Expect the second call to print `422`.

**Error paths**

- **404 on `POST /articles`:** the route was not reached. Confirm `post { articleController.create(this.context) }` is still inside `route("articles")`.
- **401 with a fresh token:** the container generates an ephemeral signing key each time it starts, so log in again after `just docker up`.
- **`withColonInTimeZone` does not compile:** check the Jackson version that `ktor-jackson` 1.2.3 pulls in, using `just deps`. That method needs Jackson 2.9.1 or later.

## Done When

- [ ] All requirements met
- [ ] Against a running container: `POST /articles` with a token returns 200 with a slug, tags, ISO-8601 dates and a secret-free author; without a token it returns 401; with a payload missing `body` it returns 422