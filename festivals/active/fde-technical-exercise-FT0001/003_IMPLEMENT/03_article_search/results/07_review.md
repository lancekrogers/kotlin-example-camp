# Gate 07 results: code review of the article search slice

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`1e24ddef-eedc-4d1d-8561-10e72c9f7d20`, 08:22:17Z to 08:24:07Z). It received the committed slice diff, the commit list,
this gate's checklist, the sequence goal, the festival rules, decisions D002/D003/D004/D007/D008/D009 and every task's
evidence, plus eight specific questions (route ordering, injection and escaping, count semantics, validation,
author exposure, transactions, test strength, scope).

The diff it saw also contains `02_article_foundation`'s commits, because this branch is stacked on that slice while its
PR waits to merge. The brief told it to review only the search work, and its Scope section correctly separates the two.

**Verdict: APPROVE, no critical findings.**

## Suggestions and dispositions

| # | Suggestion | Disposition |
|---|---|---|
| S1 | `ArticleSearchTest` `body only match` and `case insensitive` assert `articlesCount >= 1`; with D009 isolation they can assert exactly 1. | **Accepted, gate 08.** Matches the orchestrator's own observation from task 05. |
| S2 | No test covers a non-numeric `offset`, though `Paging` implements the message. D004 asks for bad offset → 422 naming the parameter. | **Accepted, gate 08**, at both layers. The orchestrator flagged the same gap when verifying task 03. |
| S3 | The HTTP `bad paging` cases assert status only; a regression to a generic 422 message would still pass. | **Accepted, gate 08.** `PagingTest` asserts messages at the unit layer, so the wire should too. |
| S4 | `anonymous request is public` asserts 200 and a non-null `articles`, so a 200 with an empty list would pass. | **Accepted, gate 08.** This is the strongest finding of the five: it means the D003 regression guard could stay green while search stopped returning results. Assert a count. |
| S5 | `Math.toIntExact(page.total)` throws `ArithmeticException` → 500 for totals above `Int.MAX_VALUE`, unlike `require(...)` failures which map to 422. | **Deferred, documented.** `ArticlesDTO.articlesCount` is the original author's `Int` field, so the conversion has to happen somewhere. Reaching it needs more than 2.1 billion matching articles in an in-memory H2 database. The alternative, clamping the value, would report a false count, which is worse than failing loudly. Recorded here rather than changed. |

Plus the orchestrator's own finding from task 05 verification, also accepted for gate 08: `blank q` and `missing q`
each contain a discarded `UUID.randomUUID().toString().take(8)` statement whose value is never used. The reviewer's
Verified section read those as "isolation comments", which is exactly why dead code is worth removing: it reads as
intentional.

## Checklist (reviewer's result)

| Item | Result | Reason |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | pass | Literals, paging, anonymous access and no-match all covered |
| `require(...)` → 422, `NotFoundException` → 404 | pass | `ArticleService.kt:23`, `Paging.kt:10-14`, `ErrorExceptionMapping.kt:38-39` |
| Every wired handler calls `ctx.respond(...)` | pass | `ArticleController.kt:84` |
| Layering and Kodein wiring | pass | `ModulesConfig.kt:27-29` |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | No version changes in the search diff |
| Routes at root (D002); public reads before mandatory auth (D003) | pass | `Router.kt:45-50`, comment accurate |
| Authors are `Profile`s; no password in a response (D008) | pass | `ArticleRepository.kt:111`, leak test at `ArticleSearchTest.kt:161-166` |
| No commented-out code, debug output or stray files | n/a | The remaining stub comments belong to the foundation slice |
| No secrets or credentials | pass | Test fixtures only |
| CI actions pinned by SHA | n/a | No CI files in this slice |
| Changes match the sequence goal; out-of-scope stubs stay stubbed | pass | Only search is live |

## Facts the reviewer verified, worth keeping

- **Route ordering.** `authenticate(optional = true) { get("search") }` is the first child of `route("articles")`, before
  the mandatory block containing `{slug}`. It also names the residual fragility honestly: a future `get("search")` added
  inside the mandatory block, or a reordering, would break anonymous access; the comment mitigates but cannot enforce
  it. The HTTP test is what would catch such a regression.
- **No injection surface.** The term is never concatenated into SQL or into a path: `LikePattern.ofLiteral` escapes it,
  `q` travels as a Unirest query parameter, and `%`/`_` are proven literal at both the repository and HTTP layers.
- **Count semantics.** `total` is computed before `limit`/`offset`, the count and page queries share one `matches`
  predicate, and the empty case asserts `articlesCount == 0`.
- **Transactions.** The whole of `search` runs in one `transaction { }` with the pattern built inside it, which is the
  constraint task 01's probe established.
- **Probe alignment.** `LowerOnClobProbeTest` exercises the same expression as production against a body-only match, so
  the guard and the implementation cannot drift apart silently.
