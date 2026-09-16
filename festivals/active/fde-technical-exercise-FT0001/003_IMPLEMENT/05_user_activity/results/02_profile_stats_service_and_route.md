# Results: 02_profile_stats_service_and_route

## Changes

- `src/main/kotlin/io/realworld/app/domain/Profile.kt` — Added `ProfileStats` and `ProfileStatsDTO`.
- `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt` — Added `countByAuthor` and `countFavoritesBy`.
- `src/main/kotlin/io/realworld/app/domain/service/ProfileStatsService.kt` — New service resolving username to counts via user, article, and comment repositories.
- `src/main/kotlin/io/realworld/app/web/controllers/ProfileController.kt` — Injected `ProfileStatsService`; added `stats` handler; stubbed `get`/`follow`/`unfollow` unchanged.
- `src/main/kotlin/io/realworld/app/config/ModulesConfig.kt` — Bound `ProfileStatsService` and updated `ProfileController(instance())` in PROFILE module.
- `src/main/kotlin/io/realworld/app/web/Router.kt` — Registered `get("stats")` inside the optional-auth block.

```
 src/main/kotlin/io/realworld/app/config/ModulesConfig.kt       |  4 +++-
 src/main/kotlin/io/realworld/app/domain/Profile.kt             |  5 ++++-
 .../io/realworld/app/domain/repository/ArticleRepository.kt    |  4 ++++
 src/main/kotlin/io/realworld/app/web/Router.kt                 |  1 +
 .../io/realworld/app/web/controllers/ProfileController.kt      | 10 +++++++++-
 5 files changed, 21 insertions(+), 3 deletions(-)
```

```
 M src/main/kotlin/io/realworld/app/config/ModulesConfig.kt
 M src/main/kotlin/io/realworld/app/domain/Profile.kt
 M src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt
 M src/main/kotlin/io/realworld/app/web/Router.kt
 M src/main/kotlin/io/realworld/app/web/controllers/ProfileController.kt
?? src/main/kotlin/io/realworld/app/domain/service/ProfileStatsService.kt
```

## Commands

### `just build gradle compileKotlin` — exit 0

```
> Task :compileKotlin

BUILD SUCCESSFUL in 6s
2 actionable tasks: 2 executed
```

### `just docker down` — exit 0

```
stopped
```

### `just docker up` — exit 0

```
up at http://localhost:18080
```

### Container verification (fresh container, `B=http://localhost:18080`) — exit 0

```
=== register main user ===
{"user":{"email":"stats_user_1789550244@valid_email.com","token":"...","username":"stats_user_1789550244","bio":null,"image":null}}
HTTP_CODE:200

=== register other user ===
{"user":{"email":"other_user_1789550244@valid_email.com","token":"...","username":"other_user_1789550244","bio":null,"image":null}}
HTTP_CODE:200

=== other user creates article ===
{"article":{"slug":"other-article","title":"Other article",...}}
HTTP_CODE:200
OTHER_SLUG=other-article

=== new user stats (expect 0/0/0) ===
{"stats":{"articlesCount":0,"commentsCount":0,"favoritesCount":0}}
HTTP_CODE:200
ok zeros

=== anonymous stats request (expect 200) ===
{"stats":{"articlesCount":0,"commentsCount":0,"favoritesCount":0}}
HTTP_CODE:200
ok anonymous 200

=== main user creates own article ===
HTTP_CODE:200

=== main user comments on other article ===
HTTP_CODE:200

=== main user favorites other article ===
HTTP_CODE:200

=== stats after activity (expect 1/1/1) ===
{"stats":{"articlesCount":1,"commentsCount":1,"favoritesCount":1}}
HTTP_CODE:200
ok 1/1/1

=== unknown username (expect 404) ===
{"errors":{"body":["Profile not found."]}}
HTTP_CODE:404
ok 404

CONTAINER VERIFICATION: PASSED
```

### `just test all` — exit 0

```
BUILD SUCCESSFUL in 48s
5 actionable tasks: 3 executed, 2 up-to-date
```

Test counts from `build/test-results/test/*.xml` via `just test census`:

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleFollowingMappingTest ran=1   passed=1   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=6   passed=6   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=83 passed=83 failed=0 skipped=17
```

### `just test census` — exit 0

Same output as above (run immediately after `just test all`).

## Done When

- [x] **All requirements met** — pass. `ProfileStats`/`ProfileStatsDTO` in `Profile.kt`; `ProfileStatsService` computes articles authored, comments written, and favorites given (D006); unknown username → 404 via `NotFoundException`; zero activity → 200 with zeros; route in optional-auth block (D003); `ProfileController` gains only `stats`; stubs unchanged; Kodein binds `ProfileStatsService` and `ProfileController(instance())`.
- [x] **Against a running container, a new user's stats are all zero; after one article, one comment and one favorite of another user's article, they are `1/1/1`; an unknown username returns 404; a request with no token returns 200** — pass. Container curl output above.

## Notes

- Branch/camp-fresh step skipped per orchestrator instruction (already on `feat/user-activity`).
- No file:line anchor drift for cited anchors (`Profile.kt:8`, `Router.kt:31`, `ModulesConfig.kt:27`, `UserRepository.kt:62`).
- `CommentRepository.countByAuthor` is used but has no automated test coverage yet; task 03 is expected to add stats tests including the zero-comment case.
- `ProfileControllerTest` remains `@Ignore` (enabled in a later slice per D001); not modified in this task.
- Container paths use `http://localhost:18080` without an `/api` prefix (matches `AppConfig` routing and `just docker smoke`).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `df29f519-5852-47a4-83d0-6f248b91bf4a`, 09:14:55Z to 09:18:46Z).
The orchestrator ran an independent container check built specifically to distinguish D006's two readings of
`favoritesCount`, which a single-user test cannot tell apart.

```text
$ just docker up
== stats for a brand-new user (expect 0/0/0)
new_user_stats=200
{"stats":{"articlesCount":0,"commentsCount":0,"favoritesCount":0}}
beta_creates_article=200
alpha_creates_article=200
alpha_comments_on_beta=200
alpha_favorites_beta=200
== alpha stats (expect 1 article / 1 comment / 1 favorite GIVEN)
alpha_stats=200
{"stats":{"articlesCount":1,"commentsCount":1,"favoritesCount":1}}
== beta stats — the decisive check: beta GAVE no favorites but RECEIVED one
beta_stats=200
{"stats":{"articlesCount":1,"commentsCount":0,"favoritesCount":0}}
   D006 satisfied (favorites = GIVEN): True
   would indicate RECEIVED instead: False
== unknown username (expect 404)
unknown_user=404
== anonymous access (expect 200, not 401)
anonymous=200
```

**The D006 semantics are proven, not assumed.** Alpha wrote one article, commented on Beta's article and favorited
Beta's article. Beta wrote one article and favorited nothing, but Beta's article *received* one favorite. Alpha's
`favoritesCount` is 1 and Beta's is 0, so the field counts favorites **given**, which is what D006 specifies. Under the
"received" reading those two values would have been swapped. A test where one user favorites one article — the obvious
shape — produces identical numbers under both readings and would have proven nothing.

The rest of the Done When:

- **A brand-new user returns `0/0/0` with a 200**, not a 404: absence of activity is not absence of a profile.
- **`1/1/1` after one article, one comment on another user's article, and one favorite of another user's article.**
- **An unknown username returns 404**, from `NotFoundException("Profile not found.")` in the service.
- **Anonymous access returns 200.** `get("stats")` sits inside the existing `authenticate(optional = true)` block next
  to `get { profileController.get(...) }`, not in the mandatory `follow` block — the task's first error path, which
  would have produced a 401.

Code checks:

- `ProfileStats(articlesCount, commentsCount, favoritesCount)` and `ProfileStatsDTO` live in `Profile.kt` as specified.
- `ArticleRepository.countByAuthor` counts `Articles.author eq userId`; `countFavoritesBy` counts
  `ArticleFavorites.user eq userId` — the "given" direction, matching the container evidence.
- `ProfileStatsService` resolves the user through the existing `UserRepository.findByUsername`, so an unknown username
  fails before any counting.
- Kodein binds `ProfileStatsService(instance(), instance(), instance())` across the USER, ARTICLE and COMMENT modules,
  and `ProfileController(instance())`; the stubbed `get`, `follow` and `unfollow` are untouched.
- **Suite unchanged at 83 ran / 17 skipped**, confirmed by an independent `cleanTest test --no-build-cache` run.

**Still uncovered by automated tests, for task 03:** `CommentRepository.countByAuthor` and both new
`ArticleRepository` counts are exercised only by this container run. Task 03's stats tests must cover them, including
the zero case and, critically, the given-versus-received distinction at the test level so it survives future edits.
