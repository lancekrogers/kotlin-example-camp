---
fest_type: task
fest_id: 04_public_search_route.md
fest_name: public_search_route
fest_parent: 03_article_search
fest_order: 4
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.689855-06:00
fest_tracking: true
---

# Task: Expose GET /articles/search publicly

## Objective

Expose `GET /articles/search` as a public read, registered before the mandatory auth block, responding with `ArticlesDTO`.

## Requirements

- [ ] In `Routing.articles` (`Router.kt:43`), an `authenticate(optional = true)` block containing `get("search")` is the first child of `route("articles")`, placed before the `authenticate {` at `Router.kt:45` (D003).
- [ ] A code comment at the registration point explains the order dependence without referring to files outside the repository
- [ ] With a valid token the viewer's email is passed through, so `favorited` and `following` can reflect the viewer; anonymous callers pass `null`

## Implementation

**Why the order matters** (Ktor source at tag `1.2.3`):

- `authenticate` adds an `AuthenticationRouteSelector` of quality 1.0 that consumes no path segment (`ktor-auth/.../Authentication.kt:321-323`).
- A constant path segment such as `search` also has quality 1.0 (`ktor-server-core/.../routing/RouteSelector.kt:28`).
- Sibling routes of equal quality resolve in registration order (`RoutingResolve.kt:119-129`).
- The mandatory block contains `{slug}`, which matches `search`. If `search` were registered after that block, `GET /articles/search` would reach the authenticated article-get, and an anonymous caller would get 401.

**Steps**

1. In `src/main/kotlin/io/realworld/app/web/Router.kt`, change the start of `route("articles") {` (`:44`) inside `fun Routing.articles(...)` so the public block comes first:
   ```kotlin
   route("articles") {
       // Public reads first. Ktor resolves equal-quality sibling routes in registration order, and
       // the authenticate block below contains {slug}, which would otherwise match "search" and
       // demand a token. Keep this block above it.
       authenticate(optional = true) {
           get("search") { articleController.search(this.context) }
       }
       authenticate {
           // ...existing routes, unchanged...
   ```
2. In `ArticleController.kt`, add:
   ```kotlin
   suspend fun search(ctx: ApplicationCall) {
       val viewer = ctx.authentication.principal<User>()?.email
       ctx.respond(articleService.search(ctx.parameters["q"], ctx.parameters["limit"], ctx.parameters["offset"], viewer))
   }
   ```
   `ctx.parameters` includes query parameters; the author's stubs read them the same way (`ArticleController.kt:12-16`).
3. Try it against a container:
   1. Run `just docker up`.
   2. Create an article containing a distinctive word, using the calls from `02_article_foundation` task 05.
   3. `curl -s "http://localhost:18080/articles/search?q=<word>"` with no token should return 200 with an `articles` array.
   4. `curl -s -o /dev/null -w '%{http_code}\n' http://localhost:18080/articles/search` should print `422`.
   5. Run `just docker down`.

**Error paths**

- **An anonymous request returns 401, or an `article` object comes back instead of `articles`:** the public block is below the mandatory block, or `get("search")` ended up inside the mandatory block.
- **404:** the handler is not registered, or the request used `/api/articles/search`. D002 keeps routes at root.

## Done When

- [ ] All requirements met
- [ ] Against a running container, anonymous `GET /articles/search?q=<word>` returns 200 with an `articles` array containing the created article, and `GET /articles/search` with no `q` returns 422
