# D010: Enable the author's ignored tests slice by slice

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R2, R12, D002, D009

## Context

Four test classes are disabled at class level:

- `ArticleControllerTest.kt:18`
- `CommentControllerTest.kt:14`
- `ProfileControllerTest.kt:13`
- `TagControllerTest.kt:14`

They target stubs, so they cannot pass as written. They are also the best available spec of what the author
intended. Only `UserControllerTest` runs today, with 4 tests. `just test census` (`test.just:40`) exists
precisely to show what a green build is skipping.

Enabling any of these tests means dealing with three problems it inherits:

- **Wrong base path.** Its paths use `/api/...`, which D002 does not adopt. `HttpUtil.kt:70` does
  the same.
- **Username collisions.** `createUser()` defaults the username to `user_name_test`
  (`HttpUtil.kt:61`). The "unfavorite article by slug" test then registers a *different*
  email with that same username (`ArticleControllerTest.kt:213`). The users table declares
  `val username: Column<String> = varchar("username", 100).uniqueIndex()` (`UserRepository.kt:21`).
- **Leftover data.** Rows persist across test methods (D009), so earlier tests leave data behind.

Together, the last two mean a repeated username can make registration fail. That failure then shows up as
a misleading 401 further down the test.

## Options

### Option A: Enable the tests each slice makes passable; give every other test a reason
- **Pros:** the author's spec becomes regression coverage exactly when it becomes true. Anything still
  disabled says why.
- **Cons:** class-level `@Ignore` turns into one annotation per test method.

### Option B: Implement enough of every stub to enable everything
- **Pros:** full coverage.
- **Cons:** that means building the whole missing subsystem. D001 rules that out.

### Option C: Delete the ignored tests
- **Pros:** a clean suite.
- **Cons:** throws away the author's spec, and reads as hiding failures.

### Option D: Leave the class-level `@Ignore`
- **Pros:** zero effort.
- **Cons:** a green suite keeps hiding 21 disabled tests. R12 rejects this.

## Decision

**Option A.** Each slice replaces the class-level `@Ignore` on the classes it touches with method-level
`@Ignore("<endpoint> is still stubbed; out of scope per D001")` on the tests it still cannot pass. It then
enables the tests it can:

| Slice | Tests enabled |
|---|---|
| Article foundation | `create article` (`ArticleControllerTest.kt:124`), `get all tags` (`TagControllerTest.kt:21`) |
| Popular Articles | `favorite article by slug` (`:190`), `unfavorite article by slug` (`:210`) |
| User Activity | `add comment for article by slug` (`CommentControllerTest.kt:21`) |

`ProfileControllerTest` stays ignored in full, because profile get, follow, and unfollow stay stubbed.
Its class-level annotation gains a reason string.

When a test is enabled:

- its paths lose the `/api` prefix;
- colliding usernames, emails, and titles get unique suffixes, per the D009 isolation rule.

No assertion may be removed or loosened. If an enabled test needs an assertion changed, the change and its
reason go in the sequence's `results/`.

## Consequences

- Every slice records `just test census` output in its sequence `results/`. That makes the growth from 4
  running tests visible, and nothing is disabled silently.
- `AGENT_WORKLOG.md` reports the final ran, passed, and skipped numbers, not just "tests pass".
