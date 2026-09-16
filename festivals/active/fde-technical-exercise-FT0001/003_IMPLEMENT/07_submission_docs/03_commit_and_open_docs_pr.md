---
fest_type: task
fest_id: 03_commit_and_open_docs_pr.md
fest_name: commit_and_open_docs_pr
fest_parent: 07_submission_docs
fest_order: 3
fest_status: completed
fest_autonomy: low
fest_created: 2026-09-15T12:03:15.7589-06:00
fest_updated: 2026-09-16T12:42:11.719937-06:00
fest_tracking: true
---


# Task: Commit the docs, open the PR, and merge when green

## Objective

Commit the docs with `fest commit`, open the PR against the fork, and merge it once green. This sequence has no quality gates, so this task carries them.

## Requirements

- [ ] Before committing, `just gate` passes (`Justfile:38-55`) and the user has reviewed the README examples and the work log wording (C6, D012).
- [ ] The commit is made with `fest commit` and has no AI attribution. The PR is created with `--repo lancekrogers/kotlin-ktor-realworld-example-app --base master`, and is merged only after the `test` and `spec` jobs are green and the user authorizes it
- [ ] After the merge, `camp fresh` syncs local `master`

## Implementation

**Why this task exists.** The sequence name ends in `_docs`, and `fest.yaml` excludes that pattern from automatic quality gates (`fest.yaml:45`, `:49`). Other sequences get review, commit and PR steps from those gates. Here they are done explicitly.

**Steps**

1. **Self-review.** Run `git diff master -- README.md AGENT_WORKLOG.md`, and check that every link resolves.
2. **User review.** Ask the user to review `AGENT_WORKLOG.md`, which is written in their voice, and apply their edits.
3. **Gate.** Run `just gate`. It must end with `=== gate: PASSED ===`.
4. **Commit.** `fest commit -m "docs: API additions, CI notes and agent work log"`
5. **Push and open the PR** (needs the user's authorization):
   ```bash
   git push -u origin docs/submission
   gh pr create --repo lancekrogers/kotlin-ktor-realworld-example-app --base master --head docs/submission \
     --title "Docs: API additions, CI, and agent work log" \
     --body "<what changed and why; decisions D001, D002, D006, D007; links to the evidence>"
   ```
6. **Wait for checks, then merge.** Watch with `gh pr checks <n> --repo lancekrogers/kotlin-ktor-realworld-example-app --watch`. When the checks are green and the user authorizes, merge using the same merge style as the earlier slices.
7. **Sync.** Run `camp fresh` in `projects/kotlin-ktor-realworld-example-app`.

**Error paths**

- **`just gate` fails in `security secrets`:** something secret-like was added to the docs, such as a real token in an example. Replace it with a placeholder.
- **The PR targets `Rudge/...`:** `--repo` was omitted. Close that PR and recreate it with the repo pinned.

## Done When

- [ ] All requirements met
- [ ] The docs PR is merged into the fork's `master` with both CI jobs green, the commit carries no AI attribution, and local `master` matches the fork after `camp fresh`