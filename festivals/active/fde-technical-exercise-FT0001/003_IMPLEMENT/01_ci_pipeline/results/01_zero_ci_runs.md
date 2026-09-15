# Task 01 results: why the fork has zero CI runs

## Baseline (read-only API, captured 2026-09-15T20:34:43Z)

```text
$ gh api repos/$R --jq '{fork, parent: .parent.full_name, visibility, default_branch}'
{"default_branch":"master","fork":true,"parent":"Rudge/kotlin-ktor-realworld-example-app","visibility":"public"}
$ gh api repos/$R/actions/permissions
{"enabled":true,"allowed_actions":"all","sha_pinning_required":false}
$ gh api repos/$R/actions/permissions/workflow
{"default_workflow_permissions":"read","can_approve_pull_request_reviews":false}
$ gh api repos/$R/actions/workflows --jq '.workflows[] | "\(.id)\t\(.state)\t\(.path)"'
357291028	active	.github/workflows/gradle.yml
$ gh api repos/$R/actions/runs --jq .total_count
0
```

`R=lancekrogers/kotlin-ktor-realworld-example-app`. Nothing changed since planning: Actions is enabled, the
only workflow is `active`, and there are still no runs.

## Actions tab

The user delegated this check ("You can do this check for me"), so the agent did it with browser automation
instead of asking.

- **claude-in-chrome** (the user's signed-in Chrome): not usable. The tool returned "Browser extension is not
  connected."
- **pinchtab**, default instance, 2026-09-15 ~20:50Z: loaded
  https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions but was **signed out** (header shows
  "Sign in" / "Sign up"). The page lists the `CI with Gradle` workflow, "0 workflow runs" and "There are no
  workflow runs yet." No fork opt-in banner, but GitHub shows that banner only to signed-in users with write
  access, so a signed-out view cannot rule it out.

- pinchtab's only other profile (`market-research`) has no running instance. The agent does not sign in to
  accounts, so no signed-in view of the Actions tab was available.

**What GitHub documents.** "When a public repository is forked, scheduled workflows are disabled by default."
([Disabling and enabling a workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)).
That covers `schedule` triggers only. This workflow has none, and the page does not describe a gate on
`push`/`pull_request` runs in forks. The fork opt-in banner remains an unconfirmed hypothesis.

## Action taken

No settings were changed. The workflow `state` is already `active`, so `gh workflow enable` has nothing to do,
and without a signed-in view there is no banner to click.

This task is blocked with that reason (`fest task blocked`). A signed-in browser is unavailable, so the
diagnosis moves to an empirical test: task 03's probe PR is the first push and PR since planning.
- If it creates a run, workflows do run on the fork, and the cause of the earlier zero runs is narrowed to
  whatever differed for PR #1.
- If it creates no run, the per-fork opt-in becomes the leading cause, and the user enables it from the
  Actions tab.

## Cause

**`unknown`, narrowed** (2026-09-15T21:00Z). Workflows run on the fork now (see Confirmation) with no settings
change, so the problem does not persist. The cause of the earlier zero runs is not proven.

**What differed.** PR #1 (`security/audit-remediation` → `master`, opened 2026-09-13T17:23:15Z, merged 19:10:36Z
as `bf1435e`) changed 30 files, none under `.github/`. Both its `pull_request` event and the merge push ran the
workflow file unchanged since `134bd8e` (2023-10-31, "Add github action"), and neither produced a run. PR #2 is
the fork's first push that changes `.github/workflows/gradle.yml`, and its run started 3 seconds after the PR
opened.

```text
$ gh api repos/$R/pulls/1 --jq '{created_at, merged_at, base: .base.ref, head: .head.label, merge_commit_sha}'
{"base":"master","created_at":"2026-09-13T17:23:15Z","head":"lancekrogers:security/audit-remediation","merge_commit_sha":"bf1435e0a20c2f0cc79836fabcd8dae1408e8222","merged_at":"2026-09-13T19:10:36Z"}
$ gh api repos/$R/pulls/1/files --paginate --jq '.[] | "\(.status) \(.filename)"' | wc -l
30
$ ... | grep -F ' .github/'
(no .github/ paths among PR #1 files)
$ git log --format='%h %ad %s' --date=iso-strict -3 master -- .github/workflows/gradle.yml
134bd8e 2023-10-31T20:38:10-03:00 Add github action
```

**Ruled out:**
- Actions disabled for the repository (`enabled: true`)
- a disabled workflow (`state: active`)
- restricted actions (`allowed_actions: all`)
- a permanent per-fork opt-in that needs a click: runs started without one

**Unverified candidate.** GitHub may hold workflows inherited from the parent on a fork until the fork changes
its own workflow files. The GitHub page cited above does not document this, so it stays a hypothesis. The
user may also have opened the Actions tab during this session, which this record cannot rule out.

**Consequence for the plan.** None. Every later slice's PR runs the new workflow, and task 03's run shows
checks are created on PRs to `master`.

## Confirmation

Workflows run on the fork. Task 03's probe PR produced the fork's first run within seconds:

- **PR:** https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/pull/2, created 2026-09-15T20:53:18Z,
  base `master`, head `ci/prove-red`
- **Run:** https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856, created
  2026-09-15T20:53:21Z

```text
$ gh run list --repo $R --branch ci/prove-red --limit 5 --json databaseId,event,status,conclusion,url,createdAt
[{"conclusion":"","createdAt":"2026-09-15T20:53:21Z","databaseId":35022352856,"event":"pull_request","status":"in_progress","url":"https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856"}]
$ gh api repos/$R/actions/runs --jq .total_count
1
$ gh pr checks 2 --repo $R
build and test (JDK 17)	pending	0	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856/job/104560931611
build and test (JDK 21)	pending	0	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856/job/104560931839
```

The agent changed no settings between the baseline and this run, so no per-fork opt-in click was needed.
That refutes the leading hypothesis from planning.
