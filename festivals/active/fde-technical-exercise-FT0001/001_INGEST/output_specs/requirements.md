# Requirements

Traceability: **B§n** = `input_specs/exercise-brief.md` section n. **CS** =
`input_specs/codebase-state-verified.md`. **AU** = `input_specs/agent-usage-record.md`.
**UD** = user direction given at the INGEST step 5 checkpoint.

Priorities: **P0** the submission is incomplete without it. **P1** materially moves a graded axis.
**P2** worth doing only if time and risk allow. IDs are stable identifiers, not an ordering.

## P0 — required for a complete submission

### R1 — Ship all three named features · B§1, UD
Add all three of the brief's named endpoints — Article Search, Popular Articles, and User Activity — so
each "feel[s] native to the existing application", follows existing conventions where they make sense,
and preserves existing behavior.

*Acceptance:* each endpoint reachable on a running container; each follows the established
controller → service → repository layering, Kodein wiring, and `Router.kt` mounting style; the four
currently enabled tests still pass unmodified.

*Delivery order:* foundation (R14) → Search → Popular → User Activity. Each slice ends green on JDK 17
and 21 and committed before the next starts, so the fork is submittable after every slice. If time runs
out, the submission is the last completed slice, not three half-built features.

*Blocked-by:* R7 (decision record), R11 (route base), R14 (foundation).

### R2 — Prove it works · B§2
Tests sufficient for a reader to understand intended behavior, which edge cases were considered,
whether the implementation works, and whether existing behavior still works.

*Acceptance:* a reader can derive each endpoint's contract from test names and assertions alone;
at least one test covers each documented edge case; suite green on JDK 17 and 21; the pre-existing
four tests unmodified and still passing.

*Note:* C3 means the inherited suite provides almost no regression signal — 4 tests. Claiming
"existing behavior still works" therefore requires saying **what that claim actually rests on**.

### R3 — Useful CI · B§4
GitHub Actions pipeline that builds and tests the project, runs against **JDK 17 and JDK 21**,
surfaces test failures clearly, and makes sensible use of Gradle caching.

*Acceptance:* a matrix run over both JDKs; test reports/annotations visible from the run summary
without downloading artifacts; Gradle cache demonstrably hit on a second run; the workflow
replaces the stale JDK-16 one (which would fail against the JVM 17 build — CS).

### R4 — Agent work log · B§5
`AGENT_WORKLOG.md` at the repo root. Short, not a transcript. Must cover all six points: harnesses
and models used, a few representative instructions, where agents materially changed the approach,
how their work was verified, something they got wrong or caused a reconsideration, and what to do
differently next time.

*Acceptance:* all six points present; raw material drawn from AU; the mistake described is real and
specific, not a hedge. The brief's stated interest — "the gap between asking an agent to do
something and building a workflow where you can trust the result" — is addressed directly.

### R5 — Walkthrough recording · B§6
5–10 minute recording (screen + voice-over preferred) showing what was built and how agents fit the
workflow. Link added to `AGENT_WORKLOG.md`.

*Acceptance:* runs 5–10 minutes; covers the build and the agent workflow; link resolves for a
logged-out viewer.

### R6 — Submission accessible · B§Submission
The fork contains implementation, tests, CI config, `AGENT_WORKLOG.md`, and the recording link, and
graders can reach both repo and recording.

*Acceptance:* verified from outside the account, not from a logged-in browser tab.

## P1 — high signal on the graded axes

### R7 — Document the decision to ship all three · B§1, UD
The brief says "Choose one of the following." The user has decided to ship all three (C10). That departs
from the brief's instruction and stretches its ~90 minute guidance, so the reasoning belongs in the repo
rather than being left for a grader to infer.

*Acceptance:* a decision record in `002_PLAN/decisions/` that states the brief asked for one, and why all
three: they share one article foundation that has to be built regardless, so each further feature costs
a slice rather than a subsystem. It also gives the delivery order and the per-slice cutoff that keeps
the fork submittable. Referenced from `AGENT_WORKLOG.md`.

### R14 — Build the article foundation the three features share · CS
No article repository or service has ever existed (CS). Build what the three features read, following
the author's intended design: the service API left in `ArticleController.kt` comments and the behavior
specified by the ignored `ArticleControllerTest`.

*Scope by slice:*
- **Foundation, for Search:** `Articles` table; tag association onto the existing `Tags` table;
  `ArticleRepository` and `ArticleService`; create with slug-from-title and a stated collision rule.
- **Popular adds:** a favorites join with a composite key, matching the `Follows` pattern
  (`UserRepository.kt:38-43`); `favorite`/`unfavorite` wired (routes exist). From this slice on,
  `favorited` and `favoritesCount` are real in every article response, including Search results.
- **User Activity adds:** a `Comments` table and comment `add` wired (route exists).

*Acceptance:* per slice, the author's tests that slice makes passable are enabled (paths per R11) and
passing — at minimum `create article`, `favorite article by slug`, `unfavorite article by slug`, and
`add comment for article by slug`. `GET /tags` returns tags from created articles. Controller methods
no feature needs (update, delete, get-by-slug, list, personal feed, comment list and delete) stay
stubbed, and that is documented.

### R15 — Settle each feature's open semantics · B§1, CS
Each feature leaves a question the brief does not answer. Decide each in `002_PLAN/decisions/` and pin
it with a test. Proposals:

- **Search:** case-insensitive substring match on title or body; blank or missing `q` → 422 through the
  existing `IllegalArgumentException` mapping; LIKE metacharacters in `q` matched literally; `limit` and
  `offset` supported with the same defaults as `findBy`, 20 and 0 (`ArticleController.kt:15-16`).
- **Popular:** a deterministic tie-break for equal favorite counts (newest first, then id), since without
  one offset pagination can repeat or skip articles; zero-favorite articles included; bounds on
  `limit`/`offset`.
- **User Activity:** what `favoritesCount` counts. The brief files this endpoint under "User Activity",
  which reads as things the user *did*: articles written, comments written, favorites given. Proposal:
  favorites given, not favorites received on the user's articles. Unknown username → 404.
- **All article lists:** `articlesCount` means page size in the author's tests and commented code
  (`ArticleController.kt:18`), but total matches in the RealWorld spec.

*Acceptance:* each decision recorded with its evidence and covered by at least one test.

### R11 — Decide the route base for the new endpoints · CS
The brief writes every endpoint as `/api/…`. This app is root-mounted (`AppConfig.kt:90-93`) and always
has been. The author moved the user tests from `/api` to root in `6a09793` when implementing them. The
bundled spec runner treats `/api` as part of `APIURL`.

*Acceptance:* the choice is recorded with its evidence. The README and `AGENT_WORKLOG.md` state how
the brief's path maps onto this app.

*Recommendation, pending user approval:* **keep root.** Moving to `/api` changes the URL of every
working endpoint, contradicting B§1's "preserve existing behavior", and touches at least 16 lines
across production config, the passing tests, compose, the README, and the justfiles (CS §Route base).

### R13 — The new endpoints must be publicly readable · CS (promoted from P2)
`Router.kt:45` wraps the whole `articles` route in mandatory `authenticate`, and nested
`authenticate(optional = true)` blocks cannot loosen it. Search (`articles/search`) and Popular
(`articles/feed/popular`) are reads under `articles`; mounted inside that block, they would demand a
token. The existing `feed` is a personal feed and rightly needs auth; Popular is not personal. User
Activity mounts under `profiles/{username}`, which already uses optional auth.

*Acceptance:* tests show all three endpoints return 200 without a token. With a token, `favorited` in
Search and Popular results reflects the viewer; without one it is `false`. Auth on the still-stubbed
article routes is left as-is, and that is documented.

### R9 — RealWorld API spec tests · B§4 bonus
Run the bundled Postman collection (`spec-api/`, 31 requests) against the running application.

*Acceptance:* runs in CI against a live container with `APIURL` set for this app's base. Requests for
endpoints that stay stubbed — list, get-by-slug, update, delete, personal feed, comment list and delete,
and profiles — are expected to fail, and are listed rather than hidden, so the job's meaning stays honest.

### R10 — CI must be visible · CS
The fork has **zero workflow runs**, even though PR #1 merged into `master` and `gradle.yml` triggers on
push and pull request to `master`. The Actions permissions API reports `enabled: true` and the workflow
reads `active`, so repository-level Actions is not switched off. The cause is not yet diagnosed; one
plausible cause is GitHub's per-fork opt-in on the Actions tab, which that API may not reflect. A
grader looking for CI evidence currently sees nothing.

*Acceptance:* the zero-run cause identified and recorded, and at least one completed run visible on the
fork's Actions tab. If the cause is the fork opt-in, enabling it is a human click in the GitHub UI.

*Diagnosed in 003_IMPLEMENT (2026-09-15):* **not the fork opt-in.** With no settings changed, probe PR #2 got
run 35022352856 three seconds after opening, and the run completed. The exact cause of the earlier zero runs
is unproven. The one recorded difference is that PR #1 changed no `.github/` files, while PR #2 was the fork's
first push to change the workflow file. Evidence: `003_IMPLEMENT/01_ci_pipeline/results/01_zero_ci_runs.md`.

## P2 — only if time and risk allow

### R8 — `unfollow` row-orientation bug · CS (demoted from P1)
`follow` inserts `(user=followed, follower=me)`; `unfollow` deletes `(user=me, follower=unfollowed)` —
the opposite orientation (`UserRepository.kt:112-122` vs `124-134`). Unfollow therefore does not
remove the follow, and on a mutual follow it deletes the other person's.

*Why demoted:* none of the three named options wires `follow`/`unfollow`, so the bug stays
unreachable over HTTP. Record it in `AGENT_WORKLOG.md` as found and deferred. If it is fixed, a test
that fails before the fix comes first — it is still only a static reading.

### R12 — The four `@Ignore`d test classes · CS
`ArticleControllerTest`, `CommentControllerTest`, `ProfileControllerTest`, `TagControllerTest` are
disabled at class level and target stubs. The author's article suite is the best available spec for
R14: enable the tests the three features make passable, and put a stated reason on whatever stays
ignored. Leaving them silently disabled while claiming a green suite is not acceptable.

## Out of scope

The rest of the article subsystem beyond R14: update, delete, get-by-slug, list, personal feed, and
comment list/delete. Also out: full RealWorld conformance, moving every route to `/api` (unless R11 is
decided otherwise), and further security work (shipped in PR #1, `bf1435e`).
