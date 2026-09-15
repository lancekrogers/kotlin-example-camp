# D001: Ship all three named features as ordered slices

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R1, R7, R14, C5, C10

## Context

The brief says "Choose one of the following." (`docs/interview-exercise.md:46`) and suggests about 90
minutes (`:22`). The user decided to ship all three: Popular Articles (`:48`), User Activity
(`:60`), and Article Search (`:80`). That decision is recorded as C10.

All three read articles, and no article data layer has ever existed. `git log --all --diff-filter=D` finds
no article or comment service or repository, and `5759ef1` added only the controller stub and domain
models. So every option needs the same foundation first. What remains to decide is the order and the
point at which work is allowed to stop.

## Options

### Option A: Ship one feature (Search)
- **Pros:** smallest diff; closest to "Choose one".
- **Cons:** contradicts the user's decision (C10).

### Option B: Ship all three, built together
- **Pros:** one pass over shared code.
- **Cons:** nothing is shippable until everything is. A timeout leaves three half-built features and no
  submission.

### Option C: Ship all three as ordered slices, each finished before the next starts
- **Pros:** the fork can be submitted after every slice. Each feature gets its own review and CI
  evidence. Scope can be cut at a slice boundary without leaving broken code.
- **Cons:** some ordering constraints must be respected (below), and several PRs replace one.

## Decision

**Option C.** The slice order is:

1. **CI pipeline** (R3, R10). Every later slice then gets JDK 17 and 21 evidence on its own PR, and the
   zero-run cause is found before any feature depends on CI.
2. **Article foundation** (R14): articles, tags on articles, and create.
3. **Article Search**, which adds one query.
4. **Popular Articles**, which adds favorites and a sorted, paginated query.
5. **User Activity**, which adds comments and counts favorites *given*. Those favorites exist only after
   slice 4, so this order is forced.
6. **RealWorld spec-test CI job** (R9, bonus). It runs last because each earlier slice makes more of the
   collection pass.

A slice is finished when its sequence's quality gates (testing, review, iterate, `fest commit`) pass and
its PR is green on both JDKs (D011, D012). The next slice starts from the merged result.

**Cutoff rule:** if time runs out, the submission is the last merged slice. `AGENT_WORKLOG.md` then says
which slices shipped.

## Consequences

- The departure from "Choose one" must be explained in the repo, in `README.md` and `AGENT_WORKLOG.md`,
  as R7 requires. It must not be left for a grader to infer.
- Slice 5 depends on slice 4 and slice 3 on slice 2. Reordering them is not free.
- The scope tension against the brief's guidance is recorded in `../../001_INGEST/output_specs/constraints.md`
  (Known tensions).
