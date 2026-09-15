# D012: Delivery through branches, fest commits, and a PR per slice

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R1, R6, C6, D001, D011

## Context

D001 splits the work into six slices. Each slice is finished before the next one starts.

Several facts constrain how that work gets committed and merged:

- **Commits.** The camp forbids raw `git commit`. While a festival is executing, commits go through
  `fest commit`, and the quality gate repeats that rule
  (`gates/implementation/QUALITY_GATE_FEST_COMMIT.md:35`).
- **Attribution.** No AI attribution in commits or PRs.
- **PR target.** In a fork, `gh pr create` targets the upstream parent by default. That happened with
  PR #1, so the target has to be pinned.
- **CI.** CI runs only on pushes to `master` and on PRs targeting it (D011). Per-slice CI evidence
  therefore needs a PR per slice.

## Options

### Option A: One branch and one PR for everything
- **Pros:** fewest operations.
- **Cons:** one giant diff with no per-feature CI evidence, and no point to stop at if time runs out.

### Option B: Commit straight to `master`
- **Pros:** fastest.
- **Cons:** nothing to review and no PR checks. The brief asks how the work was verified.

### Option C: One branch and one PR per slice, merged only when green
- **Pros:** each feature has a reviewable diff and its own CI run. The submission is always the last
  merged slice.
- **Cons:** more branch and PR operations.

## Decision

**Option C.**

- **Branches.** Each slice gets its own branch from the fork's current `master`: `ci/jdk-matrix`,
  `feat/article-foundation`, `feat/article-search`, `feat/popular-articles`, `feat/user-activity`,
  `ci/spec-tests`, `docs/submission`.
- **Commits.** Made with `fest commit` from the sequence's working directory. No AI co-author trailers or
  attribution.
- **PRs.** Opened ready for review with the target pinned:
  `gh pr create --repo lancekrogers/kotlin-ktor-realworld-example-app --base master`.
  The body states what changed and why, which decisions it implements, and its evidence (census counts and
  the CI run link).
- **Merging.** Only after both JDK jobs are green on the PR. Merging and pushing are outward-facing, so
  they happen only when the user has authorized execution. `camp fresh` then syncs local `master` before the
  next slice branches.
- **Camp submodule pointer.** Moved deliberately with `camp refs-sync` after merges, never as a side effect.

## Consequences

- The fork's PR list becomes the per-feature evidence trail the walkthrough can show.
- If a slice's PR cannot go green, the next slice does not start. The blocker is recorded in that
  sequence's `results/` and in `AGENT_WORKLOG.md`.
