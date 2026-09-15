# D008: Article and comment authors are `Profile`s, never `User`s

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R14, R2; continuity with the security remediation in PR #1

## Context

`Article.kt:18` and `Comment.kt:12` declare `val author: User? = null`. `User`
(`User.kt:39`) carries three sensitive fields:

- `email` (`:41`)
- `token` (`:42`)
- `password` (`:44`), which holds the bcrypt hash once a user is loaded from the database

Nothing fills `author` today, because the article and comment layers are stubs. The first repository that
maps an author row into `User` will serialize the email and password hash into every article and comment
response.

PR #1 closed exactly this class of leak for `/users`, and wrote down why it used a separate response type
(`User.kt:52`). The work log also records the failed alternative, a Jackson `WRITE_ONLY` annotation
that broke the API's own test client (`agent-usage-record.md:65`).

The RealWorld spec defines an article's `author` as a profile: `username`, `bio`, `image`, `following`.

## Options

### Option A: Keep `User`, clear `password` when mapping
- **Pros:** smallest change.
- **Cons:** still leaks `email` and `token`. It is also only safe while every future query remembers to
  clear the field.

### Option B: Jackson annotations on `User`
- **Pros:** one place to change.
- **Cons:** the same approach that already failed here (see `agent-usage-record.md:65`).

### Option C: Make `author` a `Profile`
- **Pros:** sensitive fields cannot be serialized, because the type has none. It matches the spec.
- **Cons:** `Article` and `Comment` change their field type.

## Decision

**Option C.**

- `Article.author` and `Comment.author` become `Profile?` (`Profile.kt:5`).
- Repositories build the `Profile` from author columns only. Author `password`, `email`, and `token` are
  never selected into it.
- `following` is true only when a signed-in viewer follows the author, using
  `UserRepository.findIsFollowUser` (`UserRepository.kt:103`). It is false for anonymous
  callers.

## Consequences

- The author's ignored tests read `author?.username` (for example `ArticleControllerTest` "get all
  articles by author"). That still compiles and passes with a `Profile`.
- Every slice that returns an article or a comment gets a leak test. It fetches the **raw JSON string**
  and asserts that `password`, `email`, and `token` appear nowhere under `author`.
- Checking raw JSON rather than a deserialized object matters: the test client's own model would silently
  drop unknown fields.
