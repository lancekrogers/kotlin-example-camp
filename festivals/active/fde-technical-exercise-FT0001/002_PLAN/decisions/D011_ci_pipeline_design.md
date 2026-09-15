# D011: CI pipeline design, spec-test job, and the zero-run cause

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R3, R9, R10, C4, C7

## Context

`.github/workflows/gradle.yml` triggers on push and pull request to `master` (`gradle.yml:10`). It is
out of date:

- it uses `actions/checkout@v3` (`:25`);
- it sets `java-version: '16'` (`:29`), but the build targets JVM 17, so that job would fail;
- it uses `gradle/gradle-build-action@v2` (`:32`).

It has also never run. The fork shows zero runs; see the R10 evidence in `../inputs/gaps.md`.

The brief (§4) asks for CI that builds and tests, runs on JDK 17 and 21, surfaces test failures clearly,
and makes sensible use of Gradle caching. The bonus is running the bundled RealWorld spec tests.

That spec runner has two problems of its own:

- it calls `npx newman` without a version (`spec-api/run-api-tests.sh:11`);
- its default `APIURL` is a remote host (`run-api-tests.sh:6`).

Current releases, checked 2026-09-15:

| Tool | Version |
|---|---|
| `actions/checkout` | v7.0.1 |
| `actions/setup-java` | v6.0.1 |
| `gradle/actions` | v6.3.0 |
| `mikepenz/action-junit-report` | v6.5.0 |
| `dorny/test-reporter` | v3.0.0 |
| `newman` (npm) | 6.2.2 |

PR #1 treated floating versions in the build as a supply-chain risk. The same rule applies to CI.

## Options

### Test surfacing
- **Raw Gradle log only.** Rejected: failures are buried in thousands of lines.
- **`dorny/test-reporter`.** Workable.
- **`mikepenz/action-junit-report`.** Chosen: it annotates failing tests on the PR check and writes a job
  summary with passed, failed, and skipped counts.
- **Gradle build scans.** Rejected: an external service with its own terms, for an exercise.

### Caching
- **Hand-written `actions/cache` keys.** Rejected: easy to get wrong.
- **`gradle/actions/setup-gradle`.** Chosen: it manages the Gradle cache and writes it only from the
  default branch.

### Spec-test failures
- **Fail the job on any failure.** Rejected: it would stay red forever, because list, feed, update,
  delete, comment list/delete, and profiles all stay stubbed (D001).
- **Mark the job `continue-on-error`.** Rejected: it would then prove nothing.
- **Compare against a checked-in expected-failures manifest.** Chosen: the job fails on an *unexpected*
  failure (a regression) and on an *unexpected* pass (a stale manifest).

## Decision

**Workflow.** Replace the contents of `.github/workflows/gradle.yml`, keeping the path so the workflow
keeps its identity.

- **Triggers:** push and pull request on `master`, plus `workflow_dispatch` for manual reruns and the
  zero-run diagnosis.
- **Permissions:** `contents: read` and `checks: write`; the report action needs the second to create its
  check run.

**Job `test`.**

- **Matrix:** a `java` dimension with values 17 and 21, and `fail-fast: false`, so a failure on one JDK
  never hides the other result.
- **Steps:** checkout, then setup-java (Temurin, version from the matrix), then setup-gradle, then
  `./gradlew build`, then action-junit-report with `if: always()`.
- **On failure:** upload `build/reports/tests` as an artifact named with the JDK version.

**Pinning.** Every third-party action is pinned by full commit SHA, with the release tag in a comment.

- The CI task resolves each SHA with `gh api repos/<owner>/<repo>/git/ref/tags/<tag>`, dereferencing
  annotated tags.
- It also confirms whether setup-gradle v6.3.0 validates the Gradle wrapper JAR checksum by default. If it
  does not, the task adds `gradle/actions/wrapper-validation`.

**Job `spec`** (a separate slice, after `test`):

1. Build the image. Start `app` with a generated `JWT_SECRET` (`compose.yaml:11`) and wait for its
   healthcheck (`compose.yaml:12`).
2. Run `npx --yes newman@6.2.2` with `APIURL=http://localhost:8080` set explicitly, so the script's
   remote default can never be used, and with the JSON reporter enabled.
3. Run a small checked-in script that compares the failed request names against
   `spec-api/expected-failures.txt`.

`run-api-tests.sh` is changed to pin the same newman version, so local runs match CI.

**Zero-run diagnosis.** This is the first task in the CI slice:

1. Check the fork's Actions tab for GitHub's per-fork workflow opt-in banner. If it is there, enabling it
   is a human click.
2. Confirm the fix with a `workflow_dispatch` run or a push.
3. Record the actual cause in the sequence's `results/` and correct R10.

## Resolved after step 6 approval (facts, no design change)

- **Wrapper validation is already covered.** `gradle/actions` `setup-gradle/action.yml` at `v6.3.0` declares
  `validate-wrappers` with default `true` (lines 200-203). The action validates every Gradle wrapper jar in the
  repository and fails the job if a checksum is invalid, so no separate `wrapper-validation` step is added.
- **Caching needs no `cache-read-only` setting.** Its default is `false` on the default branch and `true`
  elsewhere (same file, lines 25-28), which already matches this decision, so the workflow leaves it unset.
- **Report action inputs are verified.** `mikepenz/action-junit-report/action.yml` at `v6.5.0` declares
  `report_paths`, `check_name`, `include_passed`, `include_skipped`, `detailed_summary`,
  `fail_on_failure` and `require_tests` (lines 14, 41, 64, 68, 101, 48, 56).
- **Current releases of the actions added by the spec and artifact steps:** `actions/upload-artifact` v7.0.1
  and `actions/setup-node` v7.0.0 (GitHub releases API, 2026-09-15).

## Consequences

- A CI failure on one JDK can be reproduced locally with `just test jdk 21` (`test.just:29`).
- A deliberate-failure check proves the pipeline goes red with a readable annotation. It uses a throwaway
  branch containing one failing test, deleted afterwards, and the evidence is recorded rather than
  assumed.
- Updating a pinned action SHA becomes a deliberate change, reviewed like any other.
