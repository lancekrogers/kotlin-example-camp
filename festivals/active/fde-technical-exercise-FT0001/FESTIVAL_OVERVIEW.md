# Festival Overview: fde-technical-exercise

## Problem Statement

**Current State:** The fork (security-remediated in PR #1, `bf1435e`) implements users, a follow graph and tags. The brief describes articles, comments, favorites and pagination, but none of those exist: the controllers are stubs, the article data layer was never built, and 4 tests run. CI pins JDK 16 and has never run on the fork.

**Desired State:** All three named features are shipped on a minimal article foundation, each as a reviewed PR that is green on JDK 17 and 21. The bundled spec tests run in CI, and the submission documents explain the scope, the decisions, and how agent work was verified.

**Why This Matters:** This is an interview submission scored on engineering judgment, verification, and agentic workflow rather than box count. How the gap between the brief and the codebase is handled is the highest-signal part of it.

## Scope

### In Scope

- CI on the fork: JDK 17/21 matrix, caching, annotated failures, spec-test job
- Article foundation: articles, tags on articles, slugs, create, authors as Profiles
- Article Search, Popular Articles (with favorites), User Activity (with comments)
- Tests for each feature, enabling the author's tests that become passable
- README API notes, `AGENT_WORKLOG.md`, the walkthrough recording and its link

<!-- Add more items as needed -->

### Out of Scope

- The rest of the article subsystem: list, feed, get-by-slug, update, delete, comment list/delete, profile get/follow/unfollow
- Moving routes under `/api` (D002)
- Fixing the `unfollow` row-orientation bug (R8, deferred and recorded)
- Toolchain upgrades (Ktor stays 1.2.3) and further security work

<!-- Add more items as needed -->

## Planned Phases

### 001_INGEST

The brief transcribed verbatim; codebase facts re-verified; purpose, requirements, constraints and context approved.

### 002_PLAN

Gap analysis, decisions D001-D012, the festival structure and implementation plan, then scaffolding and validation.

### 003_IMPLEMENT

Seven slices in order: CI pipeline, article foundation, Article Search, Popular Articles, User Activity, spec-test CI job, submission docs.

### 004_DELIVER

Walkthrough recording and link, logged-out access check, camp submodule sync.

<!-- Add more phases as they're planned -->

## Notes

Decisions were made by the planning agent under the user's delegation, with the approval judge deciding checkpoints; the user can overrule any D###. The brief says "Choose one"; shipping all three is a recorded departure (D001). Open until their tasks run: LOWER on H2 CLOB, the H2 GROUP BY form, row persistence across tests, and the fork's zero-run cause. Operator attestations and outward-facing actions need the user.