---
fest_type: task
fest_id: 01_diagnose_zero_ci_runs.md
fest_name: diagnose_zero_ci_runs
fest_parent: 01_ci_pipeline
fest_order: 1
fest_status: completed
fest_autonomy: low
fest_created: 2026-09-15T12:01:26.440806-06:00
fest_updated: 2026-09-15T14:57:50.268227-06:00
fest_tracking: true
---




# Task: Diagnose why the fork has zero CI runs

## Objective

Find out why `CI with Gradle` has never run on the fork, get the fork to a state where workflows run, and record the real cause with evidence.

## Requirements

- [ ] The cause is established from evidence (API output and what the Actions tab shows), not assumed (R10, D011).
- [ ] Any change to GitHub settings, such as enabling workflows on the fork, is made by the user or with the user's explicit authorization, and is recorded
- [ ] Raw command output and the conclusion are saved to `003_IMPLEMENT/01_ci_pipeline/results/01_zero_ci_runs.md`

## Implementation

**Context from planning (2026-09-15).** The fork `lancekrogers/kotlin-ktor-realworld-example-app` had `actions/runs` `total_count: 0`. The permissions API returned `{"enabled":true,"allowed_actions":"all"}`, default workflow permissions were `read`, the workflow `CI with Gradle` was `active`, and upstream `Rudge/kotlin-ktor-realworld-example-app` had 27 runs. PR #1 merged into the fork's `master` on 2026-09-13 without a run, even though `.github/workflows/gradle.yml:10-14` triggers on push and pull_request to `master`. The leading hypothesis is GitHub's per-fork workflow opt-in, which the permissions API does not report. It is a hypothesis, not a finding.

**Steps**

1. Re-capture the baseline (read-only) and paste the raw output into the results file:
   ```bash
   R=lancekrogers/kotlin-ktor-realworld-example-app
   gh api repos/$R --jq '{fork, parent: .parent.full_name, visibility, default_branch}'
   gh api repos/$R/actions/permissions
   gh api repos/$R/actions/permissions/workflow
   gh api repos/$R/actions/workflows --jq '.workflows[] | "\(.id)\t\(.state)\t\(.path)"'
   gh api repos/$R/actions/runs --jq .total_count
   ```
2. Ask the user to open https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions while signed in. They should report whether a banner says workflows are not being run on this forked repository, with a button to enable them, and give the banner's exact text. Do not try to enable anything through the API without the user's go-ahead.
3. Branch on what they find:
   - **Banner present.** The user clicks the enable button. Record that they did, and when. This confirms the hypothesis.
   - **No banner, and a workflow `state` is `disabled_manually` or `disabled_inactivity`.** With authorization, run `gh workflow enable <id> --repo $R` and record the output.
   - **Neither.** Record that the cause is still unknown, and list what was ruled out.
4. Do not push to `master` to test. The first real run comes from task 03's probe PR, which triggers `pull_request`. Add a `## Confirmation` section to the results file for task 03 to fill in with that run's URL.

**Error paths**

- `gh` returns 401 or 403: run `gh auth status` and ask the user to re-authenticate. Do not continue without API access.
- The user cannot check the Actions tab right now: `fest task block` this task with that reason. Task 03's evidence depends on runs happening, so do not start task 03 until this is resolved.

## Done When

- [ ] All requirements met
- [ ] `results/01_zero_ci_runs.md` holds the raw API output, what the Actions tab showed, the action taken and who took it, and a stated cause (or `unknown` with what was ruled out)
- [ ] Once task 03's run exists, its URL is recorded under `## Confirmation`, and R10 in `001_INGEST/output_specs/requirements.md` states the confirmed cause