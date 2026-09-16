# Gate 06 results: code review of the popular articles slice

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`c9a25e11-e673-4d36-a077-d3fcf718698b`, 08:54:04Z to 08:55:14Z). It received the committed diff, the commit list, this
gate's checklist, the sequence goal, the festival rules, decisions D002/D003/D005/D007/D008/D009/D010 and every task's
evidence, plus eight specific questions (ranking, idempotence, count semantics, route placement, author exposure,
validation, test strength, performance and scope).

The diff also contains the foundation and search slices, because this branch is stacked on both while their PRs wait to
merge. The brief told it to review only the popular work, and its Scope section attributes changes correctly.

**Verdict: APPROVE, no critical findings.**

## Suggestions and dispositions

| # | Suggestion | Disposition |
|---|---|---|
| S1 | The favorite and unfavorite endpoints return an article but have no raw-JSON `assertNoAuthorSecrets` test, unlike popular. | **Accepted, gate 07.** This is a festival-rule gap, not a preference: `FESTIVAL_RULES.md` requires a raw-JSON leak test on *every* endpoint returning an article or comment (D008). Popular has one; these two do not. |
| S2 | `PopularArticlesTest` exercises only `limit=0`; add a bad-`offset` case for parity with search. | **Accepted, gate 07.** `Paging.parse` is shared, so the risk is low, but popular's paging contract should be pinned at its own endpoint. |
| S3 | `following` is one `Follows` query per row in `toArticles`, while the favorite lookups are batched. | **Accepted, gate 07.** A bounded N+1 (up to `limit` extra queries per page, so up to 100), and this slice already reworked `toArticles`, which makes it the right moment. Batching it also retires the deferral recorded in `02_article_foundation/results/04_article_repository_and_service_create.md`, where the inline query was kept for efficiency over `findIsFollowUser`: one batched query beats both. |
| S4 | The author's two enabled tests assert response shape but not `favorited`/`favoritesCount`. | **Deferred deliberately.** Adding assertions would not weaken them, and the reviewer is right that it would align them with D005. But the line this festival has held is that D010 means *enable* the author's tests and fix their data isolation, not author their assertions. `PopularArticlesTest.double favorite` and `ArticleFavoritesRepositoryTest` already assert those exact values. Keeping the author's tests as the author wrote them is worth more than duplicate coverage. |
| S5 | `Math.toIntExact(page.total)` would surface as a 500 above `Int.MAX_VALUE` articles. | **Deferred, same reason as before.** Already recorded in `03_article_search/results/07_review.md`: `ArticlesDTO.articlesCount` is the original author's `Int`, and clamping would report a false count, so failing loudly is the better trade. |

## Checklist (reviewer's result)

| Item | Result | Reason |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | pass | Favorites, ranking, paging and the enabled author tests match tasks 01-04 |
| `require(...)` → 422, `NotFoundException` → 404 | pass | `ErrorExceptionMapping.kt:34-39` |
| Every wired handler calls `ctx.respond(...)` | pass | `ArticleController.kt:70,77,87` |
| Layering and Kodein wiring | pass | Controller → service → repository |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | None in this slice |
| Routes at root (D002); public reads before mandatory auth (D003) | pass | `Router.kt:45-51` |
| Authors are `Profile`s; no password in a response (D008) | pass (code); test gap → S1 | `ArticleRepository.kt:168` |
| No commented-out code, debug output or stray files | pass | This slice *removed* the favorite/unfavorite stub comments |
| No secrets or credentials | pass | Test fixtures only |
| CI actions pinned by SHA | n/a | No CI files in this slice |
| Changes match the sequence goal; out-of-scope stubs stay stubbed | pass | Other article endpoints remain stubbed with reasons |

## Facts the reviewer verified, worth keeping

- **Ranking:** `ArticleFavorites.user.count()` after the LEFT JOIN (not `COUNT(*)`), the aggregate bound once and reused
  in `slice` and `orderBy`, order `favCount DESC, createdAt DESC, id DESC`, all articles included, and rank preserved
  through `rankedIds.mapNotNull { rowsById[it] }` because `toArticles` maps in input order.
- **Idempotence:** the existence check and the insert share one transaction; repeat unfavorite is a `deleteWhere` no-op.
  It also names the residual risk honestly: two *concurrent* first-favorites could still race to a primary-key
  violation, which would surface as a 500. Not reachable from the single-threaded tests, and not a behavior this
  exercise's scope requires handling, but recorded here rather than left implicit.
- **Count semantics:** `Articles.selectAll().count()`, unchanged at an offset past the end.
- **Route placement:** popular sits in the public block; the personal feed still resolves inside the mandatory block, as
  task 03's container evidence shows (anonymous feed → 401).
- **Test strength:** relative order only, via `assertRelativeOrder` and `fullWalk`, with UUID-scoped data and no global
  totals; the author's tests kept their original assertions.
