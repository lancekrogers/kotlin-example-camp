# Gate 06 results: findings addressed

No code changed in this iteration. Every finding was either refuted with evidence or deferred or accepted with
a reason. Nothing is waiting on a fix.

## From testing (gate 04, `results/04_testing.md`)

| # | Finding | How it was caught | Disposition |
|---|---|---|---|
| T1 | Four test classes (`ArticleControllerTest`, `CommentControllerTest`, `ProfileControllerTest`, `TagControllerTest`) are `@Ignore`d at class level with no reason string (D010, "No silent skips") | `just test census` plus a source check by the gate subagent | **Deferred to `02_article_foundation`.** This predates the CI slice, which changed no Kotlin. D010 enables the author's tests slice by slice, and that sequence's goal requires "the remaining ignored tests carry reasons; census recorded". |
| T2 | `just build matrix` restores `:test` from the Gradle build cache (`org.gradle.caching=true`), so it does not prove the tests re-ran | Orchestrator reading the task 02 output (`results/02_workflow.md`, "Build caveat") | **Mitigated.** Gate 04 added `cleanTest test --no-build-cache` on JDK 17 and JDK 21. Both show `> Task :test` executing (not `FROM-CACHE`), 4 passed and 21 skipped. |
| T3 | The gate lists `just test all`, and the subagent ran the no-cache runs instead | Subagent report | **Accepted.** `just test all` is `gradle test` on JDK 17 with the cache on. The no-cache runs on both JDKs are a strict superset. `just gate` (which includes `just test all`) runs before commit; see `results/07_just_gate.txt`. |

## From code review (gate 05, `results/05_review.md`)

| # | Finding | Disposition |
|---|---|---|
| R1 | "upload-artifact needs `actions: write`" (critical) | **Refuted.** Run 35022352856 uploaded `test-reports-jdk17` and `test-reports-jdk21`, with both upload steps `success`. No change, because adding it would widen the token. |
| R2 | `timeout-minutes` | Deferred: not in D011; jobs take about 1 minute; GitHub's default cap is 360 minutes |
| R3 | `concurrency` group | Deferred: not in D011; low PR volume |
| R4 | Read-only token on third-party fork PRs | Accepted as documented behavior |
| R5 | `require_tests` doubles a compile failure | Accepted, as intended by task 02's error path |
| R6 | Local proof used containerized `gradle`, not `./gradlew` | Accepted: CI ran `./gradlew build` on both JDKs |

## What the agents got wrong, and how it was caught

- **The review subagent overstated a requirement.** It asserted that `actions: write` was needed without evidence. The
  orchestrator checked the claim against the run's artifacts API and the action's README at the pinned tag before
  acting, so no unnecessary permission was added.
- **The task 02 subagent over-reported.** It reported "Build succeeded … matrix OK" without noting that JDK 21 tests
  came from cache. The orchestrator caught this from the raw output tail and added the no-cache runs to the gate.

## Definition of done

- [x] All critical findings fixed. None remain; R1 was refuted.
- [x] Tests pass after the changes. No changes were made; the no-cache runs on both JDKs passed, and `just gate` is recorded in `results/07_just_gate.txt`.
- [x] Code review findings addressed or explicitly deferred with a reason.
- [x] Ready to commit.
