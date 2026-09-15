# EXTRACT — rough notes (INGEST step 3)

Sources: `input_specs/exercise-brief.md` (B§n = brief section n),
`input_specs/codebase-state-verified.md` (CS), `input_specs/agent-usage-record.md` (AU).

## Purpose (→ purpose.md)

End goal: the fork at `lancekrogers/kotlin-ktor-realworld-example-app` is submission-ready for the
AmEx FDE exercise — a shipped feature, tests that justify confidence, useful CI, a work log, and a
walkthrough recording.

Why it matters: this is an interview submission. The graders state they evaluate **overall
engineering approach, not box count** (B§What We Care About). Six named axes: Engineering,
Verification, Agentic Engineering, Judgment, Communication, FDE Mindset. Judgment is explicitly
about what was investigated vs delegated vs verified, and trade-offs under time pressure.

Secondary purpose (user's, not the brief's): the festival itself is the artifact — `fest gif`
replays the run to show thought process to the interviewer. So **the planning record is part of
the submission story**, which raises the bar on decision documentation.

"Done" = all six brief deliverables present in the fork + graders can access repo and recording.

## Requirements (→ requirements.md, with P0/P1/P2)

P0 — without these the submission is incomplete:

- R1 (B§1) Ship one feature (superseded by user direction: all three, see Addendum 2). Must "feel native to the existing application", follow existing
  conventions where sensible, preserve existing behavior.
- R2 (B§2) Add tests establishing: intended behavior, edge cases considered, implementation works,
  **existing behavior still works**.
- R3 (B§4) GitHub Actions CI that builds and tests, runs **JDK 17 and JDK 21**, surfaces test
  failures clearly, makes sensible use of Gradle caching.
- R4 (B§5) `AGENT_WORKLOG.md` — short, not a transcript. Must cover: harnesses/models used,
  representative instructions, where agents changed the approach, how work was verified,
  **something they got wrong**, what to do differently.
- R5 (B§6) 5–10 min walkthrough recording, link added to `AGENT_WORKLOG.md`.
- R6 (B§Submission) Fork accessible to graders, contains implementation + tests + CI + worklog +
  recording link.

P1 — materially strengthens the graded axes:

- R7 (CS) Decide and **document** the feature choice given all three offered options are blocked
  by missing subsystems. The brief pre-authorizes this: "propose a different feature of similar
  scope" + "make a reasonable decision and document it rather than waiting for perfect
  requirements". This is the single highest-signal judgment artifact in the submission.
- R8 (CS) Fix the `unfollow` row-orientation bug, with a regression test that fails before the fix.
  Unproven statically — must be demonstrated by test first.
- R9 (B§4 bonus) Run the bundled RealWorld API spec tests against the app.
- R10 (CS) CI must actually be **visible** — zero runs on the fork, cause undiagnosed (corrected at
  step 5: the permissions API shows Actions enabled); a green badge nobody can see satisfies nothing.

P2 — do only if time and risk allow:

- R11 (CS) Resolve the `/api` prefix inconsistency (routes mount at root; brief endpoints are
  `/api/…`; existing tests disagree with each other).
- R12 (CS) Decide what to do with the four `@Ignore`d test classes — they target stubs and cannot
  pass as written. Deleting, fixing, or leaving them are all defensible; silence is not.
- R13 (CS) `articles` route is wrapped in mandatory `authenticate` so public reads are gated.

## Constraints (→ constraints.md)

Hard:
- C1 Only three tables exist (`Users`, `Follows`, `Tags`). No articles/comments/favorites. All
  three brief-offered features depend on the missing ones. **This is the defining constraint.**
- C2 H2 is in-memory; no persistence, no migrations, schema via `SchemaUtils.create`.
- C3 Regression net is 4 enabled tests (only `UserControllerTest`; two of its six are commented
  out). "Existing behavior still works" cannot be shown by the existing suite.
- C4 Ktor 1.2.3 / Exposed 0.41.1 / Kotlin 1.9.25 / Gradle 8.14, JVM 17 target. Gradle ≥7.3 needed
  for JDK 17 and ≥8.5 for JDK 21 — already satisfied, and was a prerequisite for R3.
- C5 Suggested ~90 minutes. Graders do not expect perfection; scope discipline is graded.

Process:
- C6 Camp rules: never raw `git commit` — `fest commit` while executing a festival,
  `camp p commit` in `projects/*`, `camp commit` for camp root. No AI attribution in commits/PRs.
- C7 All build/test/run happens in Docker (decision already taken and implemented).
- C8 Agent harnesses must be used and the usage documented (B§3) — this is graded, not optional.

## Context (→ context.md)

- Prior art in this camp: security audit + remediation, merged as PR #1 (`bf1435e`), 6 commits.
  Full record in `workflow/explore/ktor-realworld-repo-review/security/`. Modernized the toolchain,
  replaced HMAC-as-password-hash with bcrypt, removed the committed JWT key, containerized, added
  the modular justfile. See CS "Already delivered (do not redo)".
- The CI workflow was deliberately left stale by that PR (JDK 16 vs JVM 17 build) — R3 is where it
  gets fixed. Worth noting master's CI was already red before the PR, for worse reasons.
- AU holds the raw material for R4, including three agent mistakes worth reporting honestly
  (the `WRITE_ONLY` serialization break, the justfile `root :=` silent false pass, and a
  confidently wrong claim about Gradle 4.10 on JDK 16).
- Upstream: `Rudge/kotlin-ktor-realworld-example-app`. RealWorld spec defines the canonical API
  shapes; a Postman collection is bundled in the repo (relevant to R9).
- Open question for the user, not resolvable from documents: **which feature** to ship (R7).
  Leading candidate on the evidence — wire `ProfileController` and add
  `GET /profiles/{username}/stats` over the existing follow graph, since repository, service,
  DTOs, and routes all already exist and only the controller body is missing.

## Addendum — user direction at step 5 PRESENT

- The user rejected the substitute candidate above: ship **one of the three named options**.
  Recorded as C10 and folded into R7. The "leading candidate" line above is superseded.
- Further reading done in response. The domain models exist, the data layer never did, the ignored
  `ArticleControllerTest` is the author's intended spec, and the route-base evidence favors root.
- Requirement changes: new R14 (shared article foundation); R8 demoted to P2 (no named option wires
  follow/unfollow); R13 promoted to P1 (the new read endpoint must not sit behind mandatory auth); R11
  now carries a recommendation (root).
- Details: `input_specs/codebase-state-verified.md` §Feature sizing and §Route base.

## Addendum 2 — user revised the direction: all three features

- Asked which single option to recommend, the user decided instead to ship all three in this festival,
  reasoning that the extra work is small. C10 and R1 are rewritten accordingly; the Search-only
  recommendation above is superseded.
- Structure added so a three-feature scope stays shippable: delivery order foundation → Search →
  Popular → User Activity, each slice green and committed before the next (R1). R7 is now the decision
  record for departing from "Choose one". R14 grows by slice (favorites for Popular, comments for User
  Activity). The new R15 collects each feature's open semantics. R13 now covers all three endpoints.
- New tension recorded in constraints: "choose one" and ~90 minutes versus shipping three.
