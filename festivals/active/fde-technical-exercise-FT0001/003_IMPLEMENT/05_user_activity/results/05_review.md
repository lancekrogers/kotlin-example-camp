# Gate 05 results: code review of the user activity slice

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`53e32e27-2614-471c-8f14-73e378abece0`, 09:26:08Z to 09:28:07Z). It received the committed diff, the commit list, this
gate's checklist, the sequence goal, the festival rules, decisions D002/D003/D006/D008/D009/D010 and every task's
evidence, plus eight specific questions.

The diff contains four slices, because this branch is stacked on popular-articles, search and foundation while their
PRs wait to merge. The reviewer attributed changes correctly and listed explicitly what it excluded from judgment.

**Verdict: APPROVE, no critical findings.**

## Suggestions and dispositions

| # | Suggestion | Disposition |
|---|---|---|
| S1 | `CommentCreateTest` has no `missing body returns 422` test, though `ArticleCreateTest` has the equivalent. | **Accepted, gate 06.** A real gap: the `runCatching` path that turns a Jackson mapping failure into 422 instead of 500 is proven only by container evidence in `results/01`. Without an automated test, removing that wrapper would still pass the suite. |
| S2 | The three counts each open their own transaction; one read transaction would give snapshot consistency. | **Deferred, with a layering reason.** Wrapping them would require importing Exposed's `transaction` into `ProfileStatsService`, and no service in this codebase touches the database API — that boundary is the point of the controller → service → repository split the review itself checks. For three read-only counters in a single-process app with an in-memory database, snapshot consistency is not worth breaking it. Recorded rather than changed. |
| S3 | D006's consequences ask for the three count meanings to be documented, including that a profile's `favoritesCount` is favorites *given*, unlike `Article.favoritesCount`. No README change here. | **Deferred to the right place:** `07_submission_docs` task 01 (`01_readme_api_notes.md`) owns the README API notes. Noted there rather than done here, so the doc slice has a concrete item instead of a vague one. |
| S4 | `CommentController.delete`'s stub leaves unused locals (compiler warnings). | **Deferred.** Cosmetic, and the stub is out of scope per D001. It disappears when a later slice implements the endpoint. |

## Checklist (reviewer's result)

| Item | Result | Reason |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | pass | Comment POST, stats GET, and the 422/404/401 paths, all tested |
| `require(...)` → 422, `NotFoundException` → 404 | pass | `CommentService.kt:8`, `ProfileStatsService.kt:15`, `ErrorExceptionMapping.kt:34-39` |
| Every wired handler calls `ctx.respond(...)` | pass | `CommentController.kt:19`, `ProfileController.kt:30` |
| Layering and Kodein wiring | pass | `ModulesConfig.kt:34-41` |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | None in this slice |
| Routes at root (D002); public reads before mandatory auth (D003) | pass | `Router.kt:31-34` |
| Authors are `Profile`s; no password in a response (D008) | pass | `CommentRepository.kt:43-44`, leak test at `CommentCreateTest.kt:70-75` |
| No commented-out code, debug output or stray files | fail (pre-existing) | Stub comments in `ProfileController` and `CommentController` for endpoints D001 leaves stubbed; no new commented-out code |
| No secrets or credentials | pass | UUID-suffixed test users only |
| CI actions pinned by SHA | n/a | No CI files in this slice |
| Changes match the sequence goal; out-of-scope stubs stay stubbed | pass | `CommentController.findBySlug`/`delete` and the profile stubs unchanged |

## Facts the reviewer verified, worth keeping

- **D006 semantics confirmed in code and test.** `countFavoritesBy` counts `ArticleFavorites.user eq userId` — rows where
  this user favorited something, not favorites their articles received — and `ProfileStatsTest` asserts V=1 with W=0,
  which is the assertion that can actually fail if the sides are ever swapped.
- **No missing-table path.** `CommentRepository`'s `init` creates all six tables it references; `SchemaUtils.create` is
  idempotent, and `ProfileStatsService` pulls all three repositories, so every init runs before stats are served.
- **Error mapping table** built from the code: blank body → 422, payload missing `body` → 422 (not 500), unknown slug →
  404, unknown username → 404, comment POST without a token → 401 from Ktor before the handler, stats without a token →
  200.
- **`following = false` on a comment's author is correct for this endpoint**: the author is the signed-in caller, and a
  self-follow is not meaningful. The viewer-specific `following` rule applies where an author is returned to a
  different viewer.
- **The author's add-comment test kept its assertions**; only its setup changed, to an isolated user and article.
