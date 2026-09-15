# D006: User Activity counts

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R1, R2, R15; brief `docs/interview-exercise.md:60-65`

## Context

The brief titles this feature "User Activity" and asks `GET /api/profiles/:username/stats` to return
`articlesCount`, `commentsCount`, and `favoritesCount`. `articlesCount` and `commentsCount` have one
plausible reading each. `favoritesCount` has two: favorites the user **gave**, or favorites the user's
articles **received**.

`Article.favoritesCount` already means favorites an article *received*, so the same name on a profile
invites confusion either way.

## Options

### `favoritesCount` = favorites the user gave (articles they favorited)
- **Pros:** matches the feature's title: every count is something the user *did*.
- **Cons:** the name differs in meaning from `Article.favoritesCount`.

### `favoritesCount` = favorites received across the user's articles
- **Pros:** same meaning as the article field.
- **Cons:** that measures reputation, not activity. The brief's other two counts are both things the user
  did, so this one would be the odd one out.

## Decision

**Favorites given.** All three counts are the user's own activity:

| Field | Counts |
|---|---|
| `articlesCount` | articles the user authored |
| `commentsCount` | comments the user wrote, on any article |
| `favoritesCount` | articles the user has favorited |

- **Endpoint.** `GET /profiles/{username}/stats`, public (D003, inside the existing optional-auth block).
  It returns `{"stats": {"articlesCount": n, "commentsCount": n, "favoritesCount": n}}` and has no
  viewer-specific fields.
- **Errors.** Unknown username → **404** via `NotFoundException`. A real user with no activity → 200 with
  three zeros.
- **Comments.** They need a `Comments` table and a working `POST /articles/{slug}/comments`, which keeps
  its existing mandatory-auth route. A blank body → 422; an unknown slug → 404. The comment response's
  author is a `Profile` (D008).
- **Code placement.**
  - `ProfileStats(articlesCount: Long, commentsCount: Long, favoritesCount: Long)` and
    `ProfileStatsDTO(stats: ProfileStats)` go in `Profile.kt` (`Profile.kt:5`).
  - A `ProfileStatsService` takes the user, article, and comment repositories.
  - `ProfileController` gains a `stats` handler and a constructor dependency. Its binding at
    `ModulesConfig.kt:28` changes accordingly.
  - Its stubbed `get`, `follow`, and `unfollow` stay stubbed.

## Consequences

- `README.md` documents the three meanings in one line each, including the naming difference from
  `Article.favoritesCount`.
- **Tests must cover:**
  - zeros for a new user;
  - each count rising after the matching action;
  - favoriting *someone else's* article raising the favoriter's `favoritesCount` but not the author's;
  - a comment on another user's article counting for the commenter;
  - unknown username → 404;
  - an anonymous request → 200.
