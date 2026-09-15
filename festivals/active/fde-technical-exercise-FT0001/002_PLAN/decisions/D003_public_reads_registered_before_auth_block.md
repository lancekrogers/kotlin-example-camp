# D003: Public read endpoints are registered before the mandatory auth block

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R13, D002

## Context

`Router.kt:45` wraps the entire `articles` route in a mandatory `authenticate { }`. A route inside it
cannot be loosened by a nested `authenticate(optional = true)`, like the one at `Router.kt:50`. Search
and Popular are reads that must answer without a token.

Ktor 1.2.3 resolves routes as follows (upstream source at tag `1.2.3`):

- A constant path segment has quality 1.0 and a parameter segment 0.8
  (`ktor-server-core/.../routing/RouteSelector.kt:28,33`).
- `authenticate(…)` adds an `AuthenticationRouteSelector` of quality 1.0 whose `evaluate` returns
  `Constant` and consumes no path segment (`ktor-auth/.../auth/Authentication.kt:321-323`).
- Children are tried in registration order. A child whose quality ties the best match found so far is
  skipped unless its own selector quality is strictly higher
  (`ktor-server-core/.../routing/RoutingResolve.kt:119-129`).

Tracing `GET /articles/search` shows the consequence:

- **Search registered *after* the auth block:** the auth child is tried first. Inside it, `{slug}`
  matches `search` and the GET handler succeeds, giving best quality 1.0. The later `search` child ties
  at 1.0 and loses ("Lost in ambiguity tie"). The request reaches the article-get handler behind
  mandatory auth, so an anonymous caller gets **401**.
- **Search registered *before* the auth block:** `search` succeeds first, and the auth child loses the
  tie. The search handler runs.

`GET /articles/feed/popular` and `GET /profiles/{username}/stats` resolve correctly in either order,
because no route in the auth subtree can fully match them. Only Search depends on order. It depends on
it completely and without any error to show it.

`authenticate(optional = true)` lets the call continue with a null principal when no credentials are
sent. A request that sends *bad* credentials is still challenged (`Authentication.kt:282,297`).

## Options

### Option A: Register public reads first, inside `route("articles")`, wrapped in `authenticate(optional = true)`
- **Pros:** anonymous callers get 200. A signed-in viewer's principal is available, so `favorited` and
  `following` can reflect them. No existing route's auth changes.
- **Cons:** correctness depends on registration order. That needs a code comment and a test that pins it.

### Option B: Restructure `articles` so mandatory auth wraps only the write routes
- **Pros:** reads no longer depend on order.
- **Cons:** changes auth on routes whose controllers are stubbed and out of scope. R13 says to leave
  those routes as they are.

### Option C: Mount the new reads inside the mandatory block
- **Pros:** no routing change.
- **Cons:** requires a token for public reads, which violates R13.

## Decision

**Option A.**

- **`Routing.articles` (`Router.kt:43`).** Inside `route("articles")`, add `authenticate(optional = true)`
  as the **first** child, containing `get("search")` and `get("feed/popular")`. Put the existing
  `authenticate { }` block after it. A comment at the registration point says why the order matters and
  points to this decision.
- **`Routing.profiles` (`Router.kt:29`).** Add `get("stats")` inside the existing
  `authenticate(optional = true)` block (`Router.kt:31`). That block already precedes the
  mandatory `follow` block (`Router.kt:34`).

## Consequences

- **Pinning tests.** Each read endpoint gets two tests. The first is an anonymous request that returns
  200 with the endpoint's own response shape. The Search test asserts an `articles` list comes back,
  not an `article` or a 401, so a future reorder fails loudly.
- **Bad credentials.** A request with an invalid token gets 401 even on these public reads. That matches
  the RealWorld optional-auth behavior, and is documented in the README note from D002.
- **Slug shadowing.** An article whose slug is exactly `search` or `feed` can never be fetched with
  `GET /articles/{slug}`: the constant segment always wins. D009 reserves those slugs.
