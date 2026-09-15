# Gaps and Questions: 002_PLAN step 2

Sources:
- `../001_INGEST/output_specs/`, approved at INGEST step 5 by the approval judge.
- Reading done at this step against the project at `bf1435e`.
- The pinned upstream sources of Ktor 1.2.3 and Exposed 0.41.1.

Project `file:line` anchors below were computed from the live files when this document was written.

## How gaps are resolved here

This step's checkpoint says to present critical gaps to the user. The user's standing direction for
this festival is different: keep running the `fest next` loop until the festival is promoted to
ready, and let the approval judge decide checkpoints.

So each open product question below is settled by a recorded decision (`../decisions/D###`). Each
decision carries its evidence and the options it rejected, and is flagged **agent-decided under
delegation**. The PLAN presentation lists all of them, so the user can overrule any one before
execution starts. Nothing in this list blocks planning.

## Process blockers (cannot be resolved by an agent)

- [ ] **Gap:** INGEST gate step 3 and PLAN gate step 2 are `operator_attestation` checkpoints.
  - **Impact:** fest refuses to let a judge or an agent clear them ("checkpoint class
    operator_attestation cannot be auto-judged"). Those phases cannot close until the user approves
    them in a real terminal.
  - **Resolution:** do all other planning work, then hand both attestations to the user with the exact
    commands.
- [ ] **Gap:** The walkthrough recording (R5) is human work.
  - **Impact:** the festival cannot complete without it. Planning is unaffected.
  - **Resolution:** a human action item in the delivery phase, followed by the step that adds its link
    to the worklog.
- [ ] **Gap:** Nobody has worked out why the fork has zero workflow runs (R10).
  - **Impact:** a committed workflow proves nothing if it never runs. Evidence so far:
    - the fork is public;
    - the permissions API reports `enabled: true`, with default workflow permissions `read`;
    - the workflow is `active`;
    - upstream `Rudge/kotlin-ktor-realworld-example-app` has 27 runs, but the fork has 0, even though
      PR #1 merged into `master`.

    That pattern fits GitHub's per-fork workflow opt-in, which the permissions API does not report.
    It is still a hypothesis.
  - **Resolution:** the first task of the CI sequence diagnoses it. If it is the fork opt-in, turning it
    on is a human click in the GitHub Actions tab. Owned by D011.

## Decisions needed (resolved in `../decisions/`)

- [ ] **Route base, root vs `/api` (R11)** → D002.
- [ ] **How the three new read endpoints stay public (R13)** → D003.

  New finding at this step: in Ktor 1.2.3, when two sibling routes match with equal quality, the one
  registered first wins (`RoutingResolve.kt:119-129`).
  - `authenticate` adds a selector of quality 1.0 that consumes no path segment
    (`Authentication.kt:321-323`).
  - A constant path segment is also quality 1.0 (`RouteSelector.kt:28`).

  So a `get("search")` registered *after* the `authenticate { }` block at `Router.kt:45` loses
  the tie. `GET /articles/search` would then resolve to the authenticated `{slug}` article-get
  instead, and an anonymous caller gets a 401. Registration order is therefore part of the design, and
  a test must pin it.
- [ ] **Search semantics (R15)** → D004.

  Exposed 0.41.1 can build a pattern that treats the search term as literal text:
  - `LikePattern.ofLiteral` escapes the escape character plus the dialect's special characters
    (`SQLExpressionBuilder.kt:150-178`).
  - On H2 those are the defaults, `%` and `_` (`vendors/Default.kt:669`; the H2 override is commented
    out at `vendors/H2.kt:215`).
  - `like(LikePattern)` emits an `ESCAPE` clause (`SQLExpressionBuilder.kt:409-410`, `Op.kt:479-486`).
  - `lowerCase()` exists (`SQLExpressionBuilder.kt:20`).

  **Open:** Exposed maps `text()` to `TEXT` (`vendors/Default.kt:63`), which is a CLOB on H2. Nobody
  has checked whether `LOWER` works on a CLOB column. The search task must prove it with a test before
  relying on it.
- [ ] **Popular: ordering, tie-break, pagination, favorite idempotency (R15)** → D005.

  Exposed provides the pieces the query needs:
  - `count()` (`SQLExpressionBuilder.kt:58`);
  - `groupBy` (`Query.kt:164`);
  - `orderBy` over several `Pair<Expression, SortOrder>` (`AbstractQuery.kt:46-48`);
  - `limit(n: Int, offset: Long)` (`AbstractQuery.kt:41`).

  Favoriting the same article twice must not surface a primary-key violation as a 500.
- [ ] **User Activity: what each count means (R15)** → D006.
- [ ] **`articlesCount`: page size or total matches (R15)** → D007.
- [ ] **Article and comment authors must not be `User` objects** → D008.

  New finding at this step: `Article.kt:18` and `Comment.kt:12` declare
  `val author: User?`, and `User` carries `email`, `token`, and the `password` hash (`User.kt:44`).
  Filling those fields from the database would put password hashes into every article and comment
  response. That is the same kind of leak PR #1 closed for `/users`.
- [ ] **Slug generation and collisions (R14)** → D009.
- [ ] **What to do with the four `@Ignore`d author test classes (R12)** → D010.
- [ ] **CI design: matrix, caching, failure reporting, the spec-test job, and the zero-run cause
  (R3, R9, R10)** → D011.

  New findings at this step:
  - `spec-api/run-api-tests.sh:11` runs `npx newman` unpinned, so it fetches whatever version npm
    serves at run time. The current version is 6.2.2.
  - The script's default `APIURL` is `https://conduit.productionready.io/api`
    (`run-api-tests.sh:6`). CI must never hit that remote host by accident.
  - The existing workflow pins `java-version: '16'` (`gradle.yml:29`).
- [ ] **Delivery: branches, commits, PRs, and the per-slice cutoff (R1, C6)** → D012.

## Non-critical gaps (handled in task documents, no decision needed)

- [ ] **Gap:** The stubbed article handlers return DTOs but never send them.
  - **Impact:** `ArticleController.kt:11` returns `ArticlesDTO`, and `Router.kt:64` and
    `Router.kt:59` call these methods without using the result. Ktor then falls through to a 404.
    The working pattern calls `respond` inside the controller (`UserController.kt:17`).
  - **Resolution:** every handler a task wires calls `ctx.respond(...)`, as `UserController` does.
- [ ] **Gap:** New services need Kodein wiring.
  - **Impact:** `ModulesConfig.kt:25`, `:28` and `:31` register the article, profile
    and comment controllers with no dependencies.
  - **Resolution:** each slice's service task adds its registrations, following the USER and TAG
    modules.
- [ ] **Gap:** Validation failures must map to 422.
  - **Impact:** already handled. `require(...)` throws `IllegalArgumentException`, which is mapped to 422
    at `ErrorExceptionMapping.kt:38`.
  - **Resolution:** new validation uses `require`, not a new exception type.
- [ ] **Gap:** Join tables need a composite key pattern.
  - **Impact:** favorites and article tags each need a table keyed on two columns.
  - **Resolution:** follow `Follows` (`UserRepository.kt:43`) and the unique-name `Tags` table
    (`TagRepository.kt:9`).
- [ ] **Gap:** The `unfollow` row-orientation bug (R8, P2).
  - **Impact:** it stays unreachable, because none of the three named features wires follow/unfollow.
  - **Resolution:** recorded in `AGENT_WORKLOG.md` as found and deferred; not planned as work.
