# Task 03: Add GET /articles/feed/popular

## Changes

- `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` — added `popular(limit, offset, viewerEmail)` with LEFT JOIN ranking query using `COUNT(user)`, total count, and ordered page load via `toArticles`.
- `src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt` — added `popular` delegating to repository with `Paging.parse`.
- `src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt` — added suspend `popular` handler shaped like `search`.
- `src/main/kotlin/io/realworld/app/web/Router.kt` — registered `get("feed/popular")` inside the public `authenticate(optional = true)` block next to `search`.

### git diff --stat

```
 .../realworld/app/domain/repository/ArticleRepository.kt  | 15 +++++++++++++++
 .../io/realworld/app/domain/service/ArticleService.kt     |  6 ++++++
 src/main/kotlin/io/realworld/app/web/Router.kt            |  1 +
 .../io/realworld/app/web/controllers/ArticleController.kt |  5 +++++
 4 files changed, 27 insertions(+)
```

### git status --short --untracked-files=all

```
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
 M src/main/kotlin/io/realworld/app/domain/service/ArticleService.kt
 M src/main/kotlin/io/realworld/app/web/Router.kt
 M src/main/kotlin/io/realworld/app/web/controllers/ArticleController.kt
```

## Commands

### `just test all`

Exit code: **0**

```
BUILD SUCCESSFUL in 33s
5 actionable tasks: 4 executed, 1 up-to-date
```

All tests passed (see `just test census` for counts).

### `just test census`

Exit code: **0**

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=1   passed=1   failed=0   skipped=13
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=66 passed=66 failed=0 skipped=19

  WARNING: 19 test(s) skipped. A green build does not mean the application works.
```

### `just docker up`

Exit code: **0**

```
up at http://localhost:18080
```

### Manual container verification (three articles, two users favorite one, anonymous popular, limit=0)

Exit code: **0**

```
slugs: popular-test-1789548363-article-1 popular-test-1789548363-article-2 popular-test-1789548363-article-3
favorite popular-test-1789548363-article-1 by fav1: 200 body={"article":{"slug":"popular-test-1789548363-article-1","title":"Popular test 1789548363 article 1","description":"desc","body":"body 1","tagList":["t"],"createdAt":"2026-09-16T08:46:04.514+00:00","updatedAt":"2026-09-16T08:46:04.514+00:00","favorited":true,"favoritesCount":1,"author":{"username":"author_1789548363","bio":null,"image":null,"following":false}}}
favorite popular-test-1789548363-article-1 by fav2: 200 body={"article":{"slug":"popular-test-1789548363-article-1","title":"Popular test 1789548363 article 1","description":"desc","body":"body 1","tagList":["t"],"createdAt":"2026-09-16T08:46:04.514+00:00","updatedAt":"2026-09-16T08:46:04.514+00:00","favorited":true,"favoritesCount":2,"author":{"username":"author_1789548363","bio":null,"image":null,"following":false}}}
=== GET /articles/feed/popular (no token) status=200 ===
{"articles":[{"slug":"popular-test-1789548363-article-1","title":"Popular test 1789548363 article 1","description":"desc","body":"body 1","tagList":["t"],"createdAt":"2026-09-16T08:46:04.514+00:00","updatedAt":"2026-09-16T08:46:04.514+00:00","favorited":false,"favoritesCount":2,"author":{"username":"author_1789548363","bio":null,"image":null,"following":false}},{"slug":"popular-test-1789548363-article-3","title":"Popular test 1789548363 article 3","description":"desc","body":"body 3","tagList":["t"],"createdAt":"2026-09-16T08:46:04.553+00:00","updatedAt":"2026-09-16T08:46:04.553+00:00","favorited":false,"favoritesCount":0,"author":{"username":"author_1789548363","bio":null,"image":null,"following":false}},{"slug":"popular-test-1789548363-article-2","title":"Popular test 1789548363 article 2","description":"desc","body":"body 2","tagList":["t"],"createdAt":"2026-09-16T08:46:04.534+00:00","updatedAt":"2026-09-16T08:46:04.534+00:00","favorited":false,"favoritesCount":0,"author":{"username":"author_1789548363","bio":null,"image":null,"following":false}},{"slug":"popular-test-1789548350-article-3","title":"Popular test 1789548350 article 3","description":"desc","body":"body 3","tagList":["t"],"createdAt":"2026-09-16T08:45:51.512+00:00","updatedAt":"2026-09-16T08:45:51.512+00:00","favorited":false,"favoritesCount":0,"author":{"username":"author_1789548350","bio":null,"image":null,"following":false}},{"slug":"popular-test-1789548350-article-2","title":"Popular test 1789548350 article 2","description":"desc","body":"body 2","tagList":["t"],"createdAt":"2026-09-16T08:45:51.476+00:00","updatedAt":"2026-09-16T08:45:51.476+00:00","favorited":false,"favoritesCount":0,"author":{"username":"author_1789548350","bio":null,"image":null,"following":false}},{"slug":"popular-test-1789548350-article-1","title":"Popular test 1789548350 article 1","description":"desc","body":"body 1","tagList":["t"],"createdAt":"2026-09-16T08:45:51.427+00:00","updatedAt":"2026-09-16T08:45:51.427+00:00","favorited":false,"favoritesCount":0,"author":{"username":"author_1789548350","bio":null,"image":null,"following":false}}],"articlesCount":6}
first article: popular-test-1789548363-article-1 2 6 (expected popular-test-1789548363-article-1 with favoritesCount=2, articlesCount=3)
=== GET ?limit=0 status=422 ===
{"errors":{"body":["limit must be between 1 and 100."]}}
```

Note: `articlesCount` is 6 because the container H2 database retained articles from an earlier failed verification attempt in the same container session. The three newly created articles are present and the most-favorited (`favoritesCount=2`) ranks first.

## Done When

- [x] **All requirements met** — pass
  - D005 ordering: article with 2 favorites ranks first; zero-favorite articles included (articles 2 and 3 follow with `favoritesCount=0`).
  - H2 GROUP BY: ranking query slices only `id`, `createdAt`, and `COUNT(user)`; compiles and runs without SQL error.
  - `Paging.parse` used in service layer.
  - Route in public `authenticate(optional = true)` block before mandatory block.
- [x] **Against a running container: anonymous 200, most-favorited first, articlesCount equals total, limit=0 → 422** — pass
  - Evidence: `GET /articles/feed/popular` status=200, first slug `popular-test-1789548363-article-1` with `favoritesCount=2`; `?limit=0` status=422.

## Notes

- No anchor drift: all insertion points matched the task document.
- Added `import org.jetbrains.exposed.sql.selectAll` for `Articles.selectAll().count()` in `popular`.
- First container verification attempt failed to favorite because a bash array index (`SLUGS[0]`) was empty in the curl URL; re-ran with explicit slug variables and all checks passed.
- No dedicated popular integration tests exist yet (likely a later task in this sequence); container curl verification satisfies the task's manual check.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `ac466e74-11ed-4d78-a00b-00df4f754ae2`, 08:43:39Z to 08:46:36Z).
The subagent's own container run could not prove the total count, because articles from its earlier curls persisted in
the same container (it reported `articlesCount` 6 and said so). The orchestrator therefore ran an independent check in a
freshly started container holding exactly three articles, with two different users favoriting one of them.

```text
$ just docker up   # 3 users registered, 3 articles created by user a, users b and c favorite "pop two"
favorited slug = pop-two-1789548437 (by two different users)
b_favorites=200
c_favorites=200
== ANONYMOUS GET /articles/feed/popular (no Authorization header)
anon_popular=200
articlesCount = 3 | returned = 3
order (slug, favoritesCount):
    pop-two-1789548437 2
    pop-three-1789548437 0
    pop-one-1789548437 0
most-favorited first = True
zero-favorite articles included = 2
count equals total articles (3) = True
== paging and validation
limit_0=422
limit_1=200
   limit=1 -> returned 1 articlesCount 3
== the personal feed still resolves inside the mandatory block
feed_no_token=401
feed_with_token=404
== viewer-specific favorited flag
popular_as_b=200
   as user b: favorited = True count = 2
```

What that establishes, against D005, D007, D003 and the task's error paths:

- **The `COUNT(user)` choice is proven, not assumed.** The two unfavorited articles appear with `favoritesCount` 0 and
  rank *below* the favorited one. With `COUNT(*)` after the LEFT JOIN they would each have scored 1 and mixed in among
  favorited articles. This is the task's second error path, ruled out empirically.
- **Zero-favorite articles are included** (2 of 3 returned), which D005 requires: this is a ranking, not a filter.
- **Ranked order survives the second query.** The repository ranks ids, loads rows separately, then re-orders by the
  ranked id list. The response arrives in rank order rather than insertion order, so that mapping works.
- **`articlesCount` is the total number of articles** (3), including with `limit=1`, which returned a single article and
  still reported 3 (D007).
- **The route is genuinely public.** No `Authorization` header at all returns 200, because `get("feed/popular")` sits in
  the `authenticate(optional = true)` block that precedes the mandatory block (D003).
- **Validation reaches the wire:** `?limit=0` → 422, through the shared `Paging.parse`.
- **The viewer's own flag is right:** requesting as user b shows `favorited: true` with `favoritesCount: 2`.
- **The personal feed was not shadowed.** `GET /articles/feed` without a token is still 401, exactly as the task
  predicted from Ktor 1.2.3's resolver: the public block has no `feed` handler of its own, so that branch fails and the
  mandatory block's `get("feed")` handles the request. With a token it returns 404, which is the pre-existing stub
  behavior (the author's `feed` stub returns a DTO without calling `respond`), and that stub is out of this festival's
  scope.

Code checks:

- `slice` is limited to `Articles.id`, `Articles.createdAt` and the aggregate, grouped by the first two, which is what
  H2 accepts; the aggregate is bound once as `favCount` and reused in `slice` and `orderBy`.
- Ordering is `favCount DESC, createdAt DESC, id DESC` in one `orderBy` call, so ties break deterministically and pages
  are stable.
- The service reuses `Paging.parse` rather than re-validating, and the controller mirrors `search`'s shape, passing the
  viewer's email or null.
- **Suite unchanged at 66 ran / 19 skipped**, confirmed by an independent `cleanTest test --no-build-cache` run. This
  task adds no tests by design; task 04 adds the HTTP tests and enables the author's favorite and unfavorite tests.
