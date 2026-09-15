# Gate 05 results: code review of the CI change

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`3eb18959-7abe-41ae-a28e-4ad7da52e186`, 20:58:12Z to 20:59:09Z). It received `git diff master...ci/jdk-matrix`
(workflow only), the gate checklist, D011, task 02, and the evidence in `results/02_workflow.md` and
`results/03_red_probe.md`. The orchestrator then verified each finding against primary evidence.

## Findings and verification

### Critical 1: "`upload-artifact@v7.0.1` requires `actions: write`": **refuted, no change**

The reviewer claimed that the failure-artifact step cannot upload with only `contents: read` and `checks: write`.
The probe run shows otherwise. In both JDK jobs the build failed, `if: failure()` fired, and the upload
succeeded:

```text
$ gh api repos/$R/actions/runs/35022352856/artifacts --jq '{total_count, artifacts: [.artifacts[] | {name, size_in_bytes, expired, created_at}]}'
{"artifacts":[{"created_at":"2026-09-15T20:54:33Z","expired":false,"name":"test-reports-jdk21","size_in_bytes":14627},{"created_at":"2026-09-15T20:54:29Z","expired":false,"name":"test-reports-jdk17","size_in_bytes":14563}],"total_count":2}
$ gh run view 35022352856 --repo $R --json jobs --jq '<steps per build job>'
build and test (JDK 17): ... Build and test=failure, Publish test results=success, Upload test reports=success, ...
build and test (JDK 21): ... Build and test=failure, Publish test results=success, Upload test reports=success, ...
```

`actions/upload-artifact` `README.md` at `v7.0.1` has no token-permission requirement. Its only "Permission" section
is about file modes inside the zip ("Permission Loss"), and `action.yml` declares no token input. Adding
`actions: write` would widen the `GITHUB_TOKEN` for no benefit, against the least-privilege rule.

### Suggestions

| # | Suggestion | Disposition | Reason |
|---|---|---|---|
| S1 | Add `actions: write` | Rejected | Same as Critical 1: refuted by run 35022352856's artifacts. |
| S2 | Add `timeout-minutes` to the `test` job | Deferred | Not part of D011. The jobs took 1m8s and 1m12s, and GitHub caps jobs at 360 minutes by default. Revisit if a hang is ever seen. |
| S3 | Add a `concurrency` group | Deferred | Not part of D011. One maintainer and one PR per slice leave few superseded runs to cancel. |
| S4 | PRs from other people's forks get a read-only token, so JUnit check runs may not be created | Accepted as documented behavior | The fork takes no third-party PRs for this exercise. The build step still fails the job, and logs and artifacts still carry the failure. |
| S5 | A compile failure also fails the JUnit step through `require_tests: true` | Accepted | Intended: task 02's error path treats "No test results found" as `require_tests` doing its job. |
| S6 | The local proof used containerized `gradle`, not `./gradlew` | Accepted | CI run 35022352856 executed `./gradlew build` on both JDKs. The local no-cache runs in gate 04 add fresh test execution on both JDKs. |

## Checklist

| Item | Result | Reason |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | **pass** | The reviewer marked this fail only because of Critical 1, which is refuted. Triggers, permissions, matrix, `fail-fast`, steps, report path and artifact all match D011, and run 35022352856 showed the red path, the annotations and the artifacts working. |
| Validation via `require(...)` → 422; `NotFoundException` → 404 | n/a | Workflow-only change |
| Handlers call `ctx.respond(...)` | n/a | No handlers |
| Controller → service → repository layering, Kodein wiring | n/a | No application code |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | `build.gradle` and `gradle.properties` unchanged |
| Routes at root (D002); public reads before mandatory auth (D003) | n/a | No routes |
| Authors are `Profile`s; no password column in responses (D008) | n/a | No queries |
| No commented-out code, debug output or stray files | pass | Only `.github/workflows/gradle.yml` changed. The old boilerplate comments were removed. |
| No secrets or credentials | pass | None in the workflow |
| Every CI action pinned by SHA; no unpinned runtime fetches | pass | 5/5 pins re-resolved and matched (`results/02_workflow.md`). The wrapper distribution is checksum-pinned (`gradle-wrapper.properties` `distributionSha256Sum=61ad310d…`), and setup-gradle validates wrapper jars by default. |
| Changes match the sequence goal | pass | Workflow-only. The `spec` job is left to `06_spec_api_ci`. |

## Facts checked by the orchestrator

- **Bytecode target vs. runtime JDK.** `build.gradle:37-38` sets `sourceCompatibility`/`targetCompatibility` to `VERSION_17` and `build.gradle:43` sets
  `jvmTarget = JvmTarget.JVM_17`. The project declares no toolchain, so each matrix leg runs Gradle and the tests
  on its own JDK and compiles JVM 17 bytecode. That tests the exercise's requirement that 17 bytecode runs on JDK 21.
- **Wrapper pin.** `gradle/wrapper/gradle-wrapper.properties` pins `gradle-8.14-bin.zip` with `distributionSha256Sum`.
- **No stray changes.** The repository was unchanged after the review (`git status` clean on `ci/jdk-matrix`).

**Critical issues:** none remain.
**Suggestions:** S2 and S3 deferred with reasons; S4–S6 accepted.
