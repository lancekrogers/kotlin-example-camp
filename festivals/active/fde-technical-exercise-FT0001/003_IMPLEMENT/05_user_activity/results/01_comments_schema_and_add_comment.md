# Results: 01_comments_schema_and_add_comment

## Changes

- `src/main/kotlin/io/realworld/app/domain/repository/CommentRepository.kt` — Added `Comments` table, schema bootstrap, `add`, and `countByAuthor`.
- `src/main/kotlin/io/realworld/app/domain/service/CommentService.kt` — Added comment validation and repository delegation for `add`.
- `src/main/kotlin/io/realworld/app/web/controllers/CommentController.kt` — Wired `CommentService`; implemented authenticated `add` with validation and `runCatching` around receive.
- `src/main/kotlin/io/realworld/app/config/ModulesConfig.kt` — Bound `CommentRepository`, `CommentService`, and `CommentController` in the COMMENT module.

```
 .../kotlin/io/realworld/app/config/ModulesConfig.kt  |  6 +++++-
 .../app/web/controllers/CommentController.kt         | 20 ++++++++++++--------
 2 files changed, 17 insertions(+), 9 deletions(-)
```

```
 M src/main/kotlin/io/realworld/app/config/ModulesConfig.kt
 M src/main/kotlin/io/realworld/app/web/controllers/CommentController.kt
?? src/main/kotlin/io/realworld/app/domain/repository/CommentRepository.kt
?? src/main/kotlin/io/realworld/app/domain/service/CommentService.kt
```

## Commands

### `just build gradle compileKotlin` — exit 0

```
> Task :compileKotlin
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/CommentController.kt:31:13 Variable 'slug' is never used
w: file:///app/src/main/kotlin/io/realworld/app/web/controllers/CommentController.kt:32:13 Variable 'id' is never used

BUILD SUCCESSFUL in 6s
```

### `just docker down` — exit 0

```
stopped
```

### `just docker up` — exit 0

```
up at http://localhost:18080
```

### Container verification (fresh container, `B=http://localhost:18080`) — all exit 0

```
=== register ===
HTTP_CODE:200
{"user":{"email":"comment_test_1789549892@valid_email.com","token":"...","username":"comment_user_1789549892","bio":null,"image":null}}

=== create article ===
HTTP_CODE:200
{"article":{"slug":"comment-test-article","title":"Comment test article",...}}

SLUG=comment-test-article

=== add comment (expect 200) ===
HTTP_CODE:200
{"comment":{"id":1,"createdAt":"2026-09-16T09:11:32.661+00:00","updatedAt":"2026-09-16T09:11:32.661+00:00","body":"Very carefully.","author":{"username":"comment_user_1789549892","bio":null,"image":null,"following":false}}}
id= 1 body= Very carefully. author.username= comment_user_1789549892

=== blank body (expect 422) ===
HTTP_CODE:422
{"errors":{"body":["Comment body can't be blank."]}}

=== unknown slug (expect 404) ===
HTTP_CODE:404
{"errors":{"body":["Article not found."]}}

=== no token (expect 401) ===
HTTP_CODE:401
```

### `just test all` — exit 0

```
BUILD SUCCESSFUL in 45s
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

- [x] **All requirements met** — pass. `Comments` table with body/article/author/timestamps; `CommentRepository`/`CommentService`/`CommentController(commentService)` wired in COMMENT module (`ModulesConfig.kt:36-39`); blank body → 422, unknown slug → 404, no token → 401; response is `CommentDTO` with `Profile` author (`following=false`).
- [x] **Against a running container, adding a comment returns 200 with a Profile author; a blank body returns 422; an unknown slug returns 404; a request with no token returns 401** — pass. Container curl output above: 200 with `id`, `body`, `author.username`; 422 for blank body; 404 for unknown slug; 401 without token.

## Notes

- Branch/camp-fresh step skipped per orchestrator instruction (already on `feat/user-activity`).
- `CommentController.kt:31-32` unused-variable warnings remain in stubbed `delete`/`findBySlug` methods; task did not ask to change those.
- `CommentControllerTest` remains `@Ignore` (enabled in a later slice per D001); not modified in this task.
- Manual curl scripts must not use shell variable name `USERNAME` on macOS — it expands to the login name and causes duplicate-username 500 on second register; used `UNAME` instead.
- No file:line anchor drift encountered for the anchors cited in the task.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `e565494c-0e58-45f1-ab19-1aa016b12fe7`, 09:09:41Z to 09:13:22Z).
The subagent ran its own container check; the orchestrator ran an independent one with a different user and a payload
shaped to probe the failure modes the task names.

```text
$ just docker up   # then register, login, create an article
slug=comment-target-1789550004
== add a comment with a token
add_comment=200
{"comment":{"id":1,"createdAt":"2026-09-16T09:13:25.078+00:00","updatedAt":"2026-09-16T09:13:25.078+00:00","body":"First comment from the orchestrator.","author":{"username":"cmt_1789550004","bio":null,"image":null,"following":false}}}
== blank body (expect 422)
blank_body=422
{"errors":{"body":["Comment body can't be blank."]}}
== payload with no body field (expect 422, not 500)
missing_body=422
== unknown slug (expect 404)
unknown_slug=404
== no token (expect 401)
no_token=401
== response shape
   keys = ['author', 'body', 'createdAt', 'id', 'updatedAt']
   id = 1 | body = 'First comment from the orchestrator.'
   author keys = ['bio', 'following', 'image', 'username']
   createdAt ISO = True
   raw contains password/token = False
```

What that establishes, against the task's Done When and its error paths:

- **200 with a `Profile` author.** The author object carries exactly `username`, `bio`, `image` and `following`, and the
  raw body contains no `password` or `token` (D008). Dates serialize as ISO strings, inherited from the foundation
  slice's Jackson configuration.
- **422 on a blank body**, with a message naming the field, through `require(...)` in the service.
- **422, not 500, on a payload without `body`.** This is the task's second error path: `Comment.body` is non-null, so
  Jackson fails before validation runs, and the `runCatching` around `receive` converts that into an
  `IllegalArgumentException`. A 500 here would have meant the wrapper was missing.
- **404 on an unknown slug** and **401 with no token**, the latter from the mandatory auth block that already contained
  the comments POST route, so no routing change was needed.

Code checks:

- **The schema trap is avoided.** `CommentRepository`'s `init` calls
  `SchemaUtils.create(Users, Tags, Articles, ArticleTags, ArticleFavorites, Comments)` — every table it references,
  which is what the task's first error path warns about, since Kodein may construct this repository before
  `ArticleRepository`.
- `Comments` matches the requirement: `text` body, references to `Articles` and `Users`, and `created_at`/`updated_at`
  as `long` columns, so no new dependency is needed.
- The Kodein COMMENT module now binds `CommentRepository`, `CommentService(instance())` and
  `CommentController(instance())`, replacing the bare `CommentController()`.
- `following = false` is hardcoded in the mapping with the reason given: the comment's author is the caller, and a user
  cannot follow themselves. That holds for this endpoint; the comment-list endpoint stays stubbed.
- **Suite unchanged at 83 ran / 17 skipped**, confirmed by an independent `cleanTest test --no-build-cache` run
  (100 defined). This task adds no tests by design.

**Observations for later tasks in this slice:**

1. `countByAuthor(userId: Long)` is present and unused so far — task 02's profile stats consume it. Nothing tests it
   yet, so task 03's stats tests need to cover it, including the zero case for a user who has written no comments.
2. Comments have no raw-JSON leak test at the HTTP layer yet. The rule (D008) applies to comment-returning endpoints,
   so task 03 must add one when it enables the author's add-comment test.
