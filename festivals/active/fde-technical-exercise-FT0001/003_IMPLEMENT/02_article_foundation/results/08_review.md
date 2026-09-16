# Gate 08 results: code review of the article foundation slice

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`7b974c80-1772-4ac2-b1b7-5287d7950fdd`, 07:42:06Z to 07:44:11Z). It received the full committed slice diff
(`git diff master...feat/article-foundation`), the commit list, this gate's checklist, the sequence goal, the festival
rules, decisions D001-D012 and every task's evidence. Seven specific questions were put to it (transaction boundaries,
follow direction, author exposure, validation mapping, tag handling, test isolation and D010, scope).

**Verdict: REQUEST_CHANGES**, on two counts, both of which the orchestrator had already flagged from task 06's
verification.

## Critical findings and dispositions

| # | Finding | Disposition |
|---|---|---|
| C1 | `CommentControllerTest.kt:14` has a bare `@Ignore`, silently disabling all three comment tests. Census shows `ran=0 skipped=3` with no reason in source. Breaks `FESTIVAL_RULES.md` "No silent skips" and D010. | **Accepted, fixed in gate 09.** Add a class-level reason naming the stubbed endpoints and the slice that enables them (`05_user_activity`). |
| C2 | `ProfileControllerTest.kt:13` has the same bare `@Ignore`. D010 requires the class-level annotation to carry a reason. | **Accepted, fixed in gate 09.** Its reason must also state that no slice in this festival enables it, since profile get/follow/unfollow stay stubbed. |

## Suggestions and dispositions

| # | Suggestion | Disposition |
|---|---|---|
| S1 | `ArticleRepository.kt:48,:71,:80`: `Users.select { }` selects every column, including the password hash, though only `id`, `username`, `bio` and `image` are used. Project the columns instead. | **Deferred with reason.** The reviewer itself concluded this is not a real defect: `Profile` cannot carry a secret, and `assertNoAuthorSecrets` proves the response is clean. It is in-memory hygiene, not a leak. Recorded as a hardening opportunity; changing three queries now would widen this slice's diff without changing behavior. The orchestrator raised the same point independently in `results/04_article_repository_and_service_create.md`. |
| S2 | `ArticleControllerTest.kt` `create article` still uses `HttpUtil.createUser()`'s default `user@valid_user_mail.com`. UUID-suffixed users would fit D009 better. | **Deferred with reason.** This is the original author's test, and D010 says to enable it, not rewrite it. It passes because no other enabled test claims that email, and re-registration with the same password still logs in. Revisit if a later slice enables a test that reuses that email with a different password, which is exactly the failure task 06's error path describes. |
| S3 | `ArticleRepository.kt:90` sorts `tagList` on read; a future multi-tag test asserting input order could break. | **Accepted as documentation, no code change.** The sort is deliberate, so tag order is stable across reads. Recorded here as the contract: tests asserting multiple tags must compare order-agnostically or expect alphabetical order. The author's own default tags (`["dragons","training"]`) are already alphabetical. |
| S4 | `ArticleController.kt:20-79` still contains the original commented-out stub blocks, which the gate's "no commented-out code" item flags. | **Deferred with reason.** These are the original author's comments on handlers that task 05 explicitly required to stay stubbed. Later slices implement `GET /articles` (search), favorites and comments, and will remove the comments they replace. Deleting them now would touch out-of-scope handlers for no behavioral gain. |
| S5 | No HTTP-level 422 for a blank `description`; only the service unit test covers it. | **Accepted, added in gate 09.** The sequence goal claims "422 on invalid input", and the HTTP layer currently proves that only for a blank title and a missing `body`. An eight-line test closes the gap at the boundary that actually ships. |

## Checklist (reviewer's result, with orchestrator notes)

| Item | Result | Note |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | pass | Create works; 401/422 covered; persistence probe recorded |
| `require(...)` → 422, `NotFoundException` → 404 | pass | `ArticleService.kt:11-13`, `ArticleRepository.kt:49`, `ErrorExceptionMapping.kt:34-39` |
| Every wired handler calls `ctx.respond(...)` | pass | Only `create` was wired; `ArticleController.kt:49` |
| Layering and Kodein wiring match users and tags | pass | `ModulesConfig.kt:26-29` |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | Jackson config inside the existing `ContentNegotiation`; no version changes |
| Routes at root (D002); public reads before mandatory auth (D003) | pass / n/a | Test paths now root; `Router.kt` untouched; this slice adds no public read |
| Authors are `Profile`s; no password column in a response (D008) | pass | `Article.kt:18`, `Comment.kt:12`, `ArticleRepository.kt:93`, leak test at `ArticleCreateTest.kt:130` |
| No commented-out code, debug output or stray files | fail | S4: pre-existing stub comments on out-of-scope handlers; deferred with reason |
| No secrets or credentials | pass | Test fixtures only |
| CI actions pinned by SHA | n/a | No CI files in this diff |
| Changes match the sequence goal; out-of-scope stubs stay stubbed | pass | Only `create` is live |

## Facts the reviewer verified (excerpts worth keeping)

- **Follow direction is correct in both directions.** `Follows.user` is the followed account and `Follows.follower` the
  viewer (`UserRepository.kt:38-42`); `follow()` inserts in that orientation (`:116-118`) and `findIsFollowUser` reads it
  the same way (`:106-108`). `toArticles` matches (`ArticleRepository.kt:83-84`), and an anonymous viewer yields
  `viewerId == null` → `following = false`.
- **The internal helpers are never reached outside a transaction.** `loadBySlug` and `toArticles` are called only from
  `create`'s transaction (`:64`) and from `findBySlug`'s own (`:67`); a grep found no other callers.
- **The 422 path is genuinely reachable and no 500 risk was found** on the documented validation paths. The 401 comes
  from Ktor's mandatory `authenticate` (`Router.kt:45-66`) before the handler runs.
- **Tag sorting conflicts with no enabled test.** `create article` asserts a single tag, and the default fixture tags are
  already alphabetical.
- **The persistence probe fails safely.** If `a_writes_row` fails, `b_row_survives_new_app` fails on a missing slug
  rather than passing vacuously.
