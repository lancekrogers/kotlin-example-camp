---
fest_type: task
fest_id: 03_prove_ci_fails_red.md
fest_name: prove_ci_fails_red
fest_parent: 01_ci_pipeline
fest_order: 3
fest_status: pending
fest_autonomy: low
fest_created: 2026-09-15T12:01:26.48538-06:00
fest_tracking: true
---

# Task: Prove CI turns red with a readable annotation

## Objective

Prove the new pipeline turns red, with a per-test annotation, when a test fails; then remove the probe without merging it.

## Requirements

- [ ] The evidence comes from a real GitHub run of the new workflow, not from reading the workflow file (D011 consequence).
- [ ] The probe never merges: its PR is closed and its branch deleted after the evidence is captured
- [ ] Pushing the probe branch and opening its PR happen only with the user's authorization

## Implementation

**Steps**

1. Start from the committed `ci/jdk-matrix` branch (task 02):
   `git switch -c ci/prove-red ci/jdk-matrix`
2. Add `src/test/kotlin/io/realworld/app/CiRedProbeTest.kt`:
   ```kotlin
   package io.realworld.app

   import org.junit.Assert.assertEquals
   import org.junit.Test

   class CiRedProbeTest {
       @Test
       fun `ci red-path probe - must fail`() {
           assertEquals("CI must report this failure as an annotation", 1, 2)
       }
   }
   ```
3. Confirm locally that exactly this test fails: `just test only CiRedProbeTest` (`.justfiles/test.just:19`) must exit non-zero with `expected:<1> but was:<2>`.
4. With the user's authorization, commit, push and open a PR clearly marked as a probe. Pin the PR to the fork, because a fork's PRs otherwise target upstream (D012):
   ```bash
   fest commit -m "test: CI red-path probe (never merged)"
   git push -u origin ci/prove-red
   gh pr create --repo lancekrogers/kotlin-ktor-realworld-example-app --base master --head ci/prove-red \
     --title "CI red-path probe (do not merge)" \
     --body "Proves a failing test turns the check red with an annotation. Closed after evidence is captured."
   ```
5. When the run finishes, capture the evidence into `results/03_red_probe.md`:
   ```bash
   R=lancekrogers/kotlin-ktor-realworld-example-app
   gh pr checks <pr-number> --repo $R
   run=$(gh run list --repo $R --branch ci/prove-red --limit 1 --json databaseId --jq '.[0].databaseId')
   gh run view "$run" --repo $R --json conclusion,url,jobs --jq '{conclusion, url, jobs: [.jobs[] | {name, conclusion}]}'
   head=$(git rev-parse HEAD)
   for id in $(gh api "repos/$R/commits/$head/check-runs" --jq '.check_runs[] | select(.name | startswith("JUnit")) | .id'); do
     gh api "repos/$R/check-runs/$id/annotations" --jq '.[] | "\(.path):\(.start_line) \(.annotation_level) \(.message)"'
   done
   ```
   Expected: both `build and test (JDK 17)` and `build and test (JDK 21)` conclude `failure`, and at least one annotation names `CiRedProbeTest`. Copy the run URL into task 01's `## Confirmation` section; it proves workflows now run on the fork.
6. Clean up and record it:
   ```bash
   gh pr close <pr-number> --repo $R --delete-branch
   git switch ci/jdk-matrix
   git branch -D ci/prove-red
   ```

**Error paths**

- **The PR gets no run at all.** Task 01's cause is not resolved. Stop and return to task 01.
- **The run is green.** Tests are not running, or the report path is wrong. Read the job log (`gh run view "$run" --repo $R --log`) before changing anything, and fix the workflow on `ci/jdk-matrix`, not on the probe branch.
- **The run is red but has no annotations.** Check the JUnit check run's summary text. If failures appear in the summary but not as annotations, record that, then compare the inputs against `mikepenz/action-junit-report` v6.5.0 `action.yml` (`annotate_only`, `check_annotations`).

## Done When

- [ ] All requirements met
- [ ] `results/03_red_probe.md` shows both JDK jobs concluded `failure`, with at least one annotation naming `CiRedProbeTest`, and the run URL is recorded in task 01's `## Confirmation`
- [ ] The probe PR is closed, `ci/prove-red` is deleted locally and on the fork, and `CiRedProbeTest.kt` does not exist on `ci/jdk-matrix`
