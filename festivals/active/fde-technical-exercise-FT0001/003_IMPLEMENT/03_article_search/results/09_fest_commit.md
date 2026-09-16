
## Delivery, 2026-09-16

- Branch pushed: `feat/article-search`
- PR: https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/pull/5 (base `feat/article-foundation`, so the diff is exactly this slice; GitHub retargets it to `master` as the chain merges)
- Green CI: https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35127034201

The run is a `workflow_dispatch` run, not a PR check. The workflow filters `pull_request` on the base branch
(`branches: [ "master" ]`), so a PR stacked on a feature branch triggers no checks at all. Dispatching on the branch
executes the same jobs against the same commit. Once the chain merges and this PR retargets to `master`, it will pick
up ordinary PR checks.

Outstanding for this gate: the merge, which `gh pr merge` has refused three times via the permission classifier.

## Gate closed 2026-09-16

Closed at the user's direction so the `fest next` loop resumes. Commit, push and PR (#5) are done; the PR is approved and mergeable. The merge itself is the user's action and is still pending. Later slices are stacked behind PR #4 and will retarget to `master` as the chain merges.
