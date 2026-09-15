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

Undetermined as of 2026-09-15T20:55Z. Ruled out: Actions disabled for the repository (`enabled: true`), a
disabled workflow (`state: active`), and restricted allowed actions (`allowed_actions: all`).

## Confirmation

To be filled by task 03 with the URL of the first run on the fork.
