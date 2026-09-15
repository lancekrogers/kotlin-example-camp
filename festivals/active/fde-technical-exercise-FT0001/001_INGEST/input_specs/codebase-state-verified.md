# Verified state of the codebase (source: security audit + re-verification on merged master)

> **Source documents:** `workflow/explore/ktor-realworld-repo-review/security/SECURITY_AUDIT.md`
> (appendix "non-security facts that change the plan") and `.../REMEDIATION.md`.
> Every claim below was **re-verified against `master` at commit `bf1435e`** on 2026-09-13,
> after the security remediation PR merged. Where the audit and the current code disagree,
> the current code wins and is noted.

## The single most important planning constraint

**All three features the brief offers are blocked by missing subsystems.** The brief states the
repo "implements … articles, comments, profiles, favorites, and pagination". It does not.

Verified on `bf1435e`:

- **Three tables exist, total:** `Users` and `Follows` (`UserRepository.kt:19,38`) and `Tags`
  (`TagRepository.kt:9`). There is no articles, comments, or favorites table.
- **Two services and two repositories exist:** `TagService`, `UserService`, `TagRepository`,
  `UserRepository`. There is no `ArticleService`, `ArticleRepository`, `CommentService`,
  `CommentRepository`, or `ProfileService`.
- **The article, comment, and profile controllers are stubs.** Every method body is commented
  out and returns an empty DTO — e.g. `ArticleController.findBy` returns
  `ArticlesDTO(listOf(), 1)` with the real implementation sitting in comments
  (`ArticleController.kt:11-76`, `CommentController.kt:9-30`, `ProfileController.kt:7-23`).

Mapping that onto the brief's three options:

| Brief option | Requires | Status |
|---|---|---|
| `GET /api/articles/feed/popular` — order by favorite count | articles + favorites tables, article read path | blocked |
| `GET /api/profiles/:username/stats` — article/comment/favorite counts | all three subsystems | blocked |
| `GET /api/articles/search?q=` — search titles and bodies | articles table, article read path | blocked |

The brief anticipates this kind of gap: *"If you encounter ambiguity, make a reasonable decision
and document it rather than waiting for perfect requirements."* It also permits a substitute:
*"You may propose a different feature of similar scope."* This was framed as the central decision
the festival had to make. **Superseded by user direction: all three.** The user ruled out a substitute
and chose to ship all three named features (`output_specs/constraints.md` C10), so what remains for PLAN
is how to build them, not which one.

## Test baseline

- **4 tests actually run.** Only `UserControllerTest` is enabled, and two of its six `@Test`
  methods are commented out (`UserControllerTest.kt:18-19, 58-59`). The enabled four are
  login, register, get-current-user, and update-user.
- **Four test classes are `@Ignore`d at class level:** `ArticleControllerTest.kt:18`,
  `CommentControllerTest.kt:14`, `ProfileControllerTest.kt:13`, `TagControllerTest.kt:14`.
  They target the stubbed controllers, so they cannot pass as written.
- Integration-test infrastructure does exist and works: `AppRule` boots the real server on a
  real port, and `HttpUtil` drives it over HTTP. This is a usable foundation, not a greenfield.
- **Consequence:** there is almost no regression net. "Existing behavior still works" (brief §2)
  cannot be demonstrated by the existing suite and has to be established.

## CI baseline

- `.github/workflows/gradle.yml` pins `java-version: '16'` and runs `./gradlew build`.
- The build now targets JVM 17, so this workflow **would fail** if it ran.
- **It has never run.** `gh run list` on the fork returns zero runs and the runs API reports
  `total_count: 0`, despite PR #1 merging into `master`. The workflow shows `active`, and the Actions
  permissions API reports `enabled: true` (checked at step 5), so repository-level Actions is not off.
  The cause is not yet diagnosed. An earlier draft asserted "GitHub disables Actions on new forks";
  that API result does not support it. A grader looking for CI evidence currently sees nothing at all.
- Brief §4 requires: build and test, JDK 17 **and** 21, test failures surfaced clearly, sensible
  Gradle caching. Bonus: run the bundled RealWorld API spec tests.

## Already delivered (do not redo)

Merged in PR #1 (`bf1435e`), 6 commits. Relevant to planning because it changes the starting line:

- **Toolchain modernized:** Gradle 4.10 → 8.14 (with `distributionSha256Sum`), Kotlin 1.3.+ →
  1.9.25, Exposed 0.14.1 → 0.41.1, all versions pinned in `gradle.properties`. This was a
  prerequisite for the JDK 17/21 matrix, since Gradle 4.10 cannot run on either.
- **`jcenter()` and `mavenLocal()` removed**; `buildscript` block replaced with the `plugins` DSL.
- **Auth rewritten:** hardcoded HMAC key and HMAC-as-password-hash replaced with bcrypt
  (cost 12) and an env-supplied `JWT_SECRET`; JWT now verifies issuer *and* audience.
- **Password material no longer echoed** in any response (separate `UserResponse` DTO).
- **Containerized:** `Dockerfile`, `compose.yaml`, and a modular `Justfile` + `.justfiles/`
  (`build`, `test`, `docker`, `security` modules). All build/test/run happens in Docker.
- Malformed JDBC URL fixed; `Server.createPgServer()` removed; `kls_database.db` gitignored.

**Still open from that work:** the CI workflow was deliberately left untouched, so it is stale
(JDK 16 vs a JVM 17 build). Brief §4 is the natural place to fix it.

## Standing constraints

- **H2 is in-memory** (`jdbc:h2:mem:realworld;DB_CLOSE_DELAY=-1`). State dies with the process.
  No migrations exist and none are needed; `SchemaUtils.create` always runs against an empty DB.
  Any feature must create its schema this way, not assume persistence.
- **Routing has no `/api` prefix.** `Router.kt` mounts `users`, `profiles`, `articles`, `tags`
  at the root, but the brief's endpoints are all `/api/…`. The existing tests are themselves
  inconsistent (`HttpUtil.kt:57` posts to `/users`, `HttpUtil.kt:70` to `/api/articles`).
  Shipping a `/api/...` route means resolving this deliberately.
- **The `articles` route is wrapped in mandatory `authenticate { }`** (`Router.kt:45`), so
  nested `authenticate(optional = true)` cannot loosen it. Public read endpoints are currently
  auth-gated. Fails closed, so it is a spec deviation rather than a vulnerability.
- Suggested time is ~90 minutes and the graders explicitly do not expect perfection. Scope
  discipline is itself being evaluated ("appropriately scoped", "trade-offs under the time
  constraint").

---

## Correction and major addition (found during INGEST step 2 READ, verified on `bf1435e`)

The "all three features are blocked" summary above is true as written, but it undersells what
exists. **The profiles/follows subsystem is roughly 90% complete** — the gap is one controller.

Verified layer by layer:

| Layer | State | Evidence |
|---|---|---|
| `Follows` table | **exists and works** | `UserRepository.kt:38-43`, created in `init` via `SchemaUtils.create(Follows)` |
| Repository methods | **fully implemented** | `findByUsername`, `findIsFollowUser`, `follow`, `unfollow` (`UserRepository.kt:103-134`) |
| Service methods | **fully implemented** | `getProfileByUsername`, `follow`, `unfollow` (`UserService.kt:45,65,71`) |
| Domain DTOs | **exist** | `ProfileDTO`, `Profile(username, bio, image, following)` (`Profile.kt:3-5`) |
| Routes | **already wired** | `Router.kt:29-40` mounts `profiles/{username}`, `follow` POST/DELETE |
| Controller | **stub — the only gap** | `ProfileController.kt:7-23`: reads the param, returns `Unit`, real calls commented out |

So `ProfileController` is three one-line bodies away from a working feature, and the service it
needs (`UserService`) is already injected elsewhere in the app.

### This does not unblock brief option 2 (corrected at step 5)

An earlier draft said follow-graph counts made option 2 "partially" shippable. They don't. Option 2
specifies `articlesCount`, `commentsCount`, and `favoritesCount`; `followersCount` and
`followingCount` are none of those, so shipping them would be a substitute feature wearing option 2's
URL. The user first directed that the submission ship one of the three named options, then revised
that to all three (**superseded by user direction: all three**, see `output_specs/constraints.md` C10).
Either way, this path is dropped. See §Feature sizing below. The profile layer and the `unfollow` bug below remain accurate
findings; none of the three named options wires `follow`/`unfollow`, so the bug stays unreachable.

### A real pre-existing bug in `unfollow` (candidate regression test target)

`follow` and `unfollow` write and delete **opposite** row orientations:

```kotlin
// follow(email = A, usernameToFollow = B)   →  UserRepository.kt:112-122
Follows.insert { row -> row[Follows.user] = userToFollow.id!!   // = B
                        row[follower]     = user.id!! }         // = A   → row (user=B, follower=A)

// unfollow(email = A, usernameToUnFollow = B)  →  UserRepository.kt:124-134
Follows.deleteWhere { Follows.user eq user.id!!                 // = A
                      and (Follows.follower eq userToUnfollow.id!!) }  // = B  → deletes (user=A, follower=B)
```

`follow` records "A follows B" as `(user=B, follower=A)`. `unfollow` deletes `(user=A, follower=B)`
— which is the row for "**B** follows **A**". Consequences:

1. Unfollowing does not remove the follow; `findIsFollowUser` keeps returning `true`.
2. If the follow is mutual, unfollowing silently destroys **the other person's** follow.

This is unreachable today only because the controller is stubbed, so nothing calls it. Wiring the
controller makes it reachable. It is a strong candidate for the "important edge cases you
considered" part of brief §2, and it is exactly the kind of defect a follow/unfollow round-trip
test catches.

> **Not yet fixed and not yet independently confirmed by execution.** The reading above is static.
> It should be proven with a test that follows, unfollows, and asserts `following == false`
> before any fix is claimed.


---

## Feature sizing for the three named options (INGEST step 5, verified on `bf1435e`)

### What exists for articles

- **Domain models are complete.** `Article.kt`: `Article(slug, title, description, body, tagList,
  createdAt, updatedAt, favorited, favoritesCount, author)`, `ArticleDTO`, and
  `ArticlesDTO(articles, articlesCount)`. `Comment.kt`: `Comment`, `CommentDTO`, `CommentsDTO`.
- **Routes exist for every article, comment, and favorite operation** (`Router.kt:43` onward).
- **The author's intended service API survives in comments** in `ArticleController.kt`: constructor
  injection of `ArticleService` (`:9`), `findBy(tag, author, favorited, limit, offset)` (`:17`),
  plus `findBySlug`, `create(email, article)`, `update(slug, article)`, `delete(slug)`,
  `favorite(email, slug)`, `unfavorite(email, slug)`.
- **No data layer ever existed.** `git log --all --diff-filter=D` finds no article or comment
  service/repository ever deleted. `5759ef1` (2019-03-31) added the controller and domain only.
- **Tags have no article association.** `Tags` is a unique-name table with only `findAll`
  (`TagRepository.kt:9`).
- **The ignored `ArticleControllerTest` (14 tests, written in `2351771`) is an executable spec** of
  the intended subsystem. Slugs derive from titles (`"slug test"` → `slug-test` in "favorite article
  by slug"), `tagList` round-trips on create ("create article"), author is populated, and every list
  test asserts `articles.size == articlesCount`.

### The shared foundation — every named option needs it

1. `Articles` table: unique slug, title, description, body, author → `Users`, createdAt, updatedAt.
2. An article↔tag join onto the existing `Tags` table, with get-or-create by name. Required because
   `tagList` is part of the response format and the author's create test asserts it round-trips.
   Side effect: `GET /tags` starts returning real data.
3. `ArticleRepository` + `ArticleService`, following the User/Tag layering and Kodein wiring.
4. `ArticleController.create` wired (route already exists), with slug-from-title generation and a
   collision rule for two articles with the same title.

### Beyond the foundation

| Option | Extra tables | Extra write paths needed to make it testable | Query | Edge cases worth testing |
|---|---|---|---|---|
| **Search** `GET /api/articles/search?q=` | none | none | title OR body, case-insensitive | blank/missing `q` → 422 via the existing `IllegalArgumentException` mapping; LIKE metacharacters (`q=%` must not match everything); no matches → empty list; title-only vs body-only match |
| **Popular** `GET /api/articles/feed/popular` | favorites join | wire `favorite` + `unfavorite` | LEFT JOIN favorites, GROUP BY, ORDER BY count DESC, `limit`/`offset` | deterministic tie-break (equal counts make offset paging unstable); zero-favorite articles included; offset past the end |
| **Stats** `GET /api/profiles/:username/stats` | favorites join + `Comments` | wire `favorite` + comment `add` | three counts | `favoritesCount` is ambiguous (favorites the user gave, or favorites their articles received); unknown username → 404 |

Search < Popular < Stats. Because the foundation is shared, each later feature reuses the earlier work.
**Superseded by user direction: all three.** This sizing first framed Popular as an optional stretch
after Search. The user has since decided to ship all three, and the same ascending cost now sets the
delivery order in `output_specs/requirements.md` R1: foundation → Search → Popular → User Activity.

### Applies to all three

- **Auth gating.** The whole `articles` route sits inside mandatory `authenticate` (`Router.kt:45`).
  Search and Popular are reads; mounting them inside that block would demand a token.
- **`favorited`/`favoritesCount` in the list format.** Under Search there is no favorites table, so
  these are truthfully `false`/`0`: no favorite can exist while the favorite endpoints are stubbed.
  Document this so it doesn't read as a bug.
- **`articlesCount` semantics conflict.** The author's tests and commented code use page size
  (`ArticlesDTO(articles, articles.size)`, `ArticleController.kt:18`); the RealWorld spec uses the
  total number of matches. Decide in PLAN.

### Verify during implementation, not assumed now

- That Ktor 1.2.3 routes the constant segment `search` ahead of `{slug}`. The existing `feed` route
  relies on the same rule, but it has never run because the controller is stubbed.
- What Exposed 0.41.1 offers on H2 for case-insensitive LIKE and escape characters.

---

## Route base: root vs `/api` (INGEST step 5, verified)

- **Root since the first router.** `AppConfig.kt:90-93` mounts routers directly on `routing`. The
  `5759ef1` diff shows the earliest Ktor router, `Routing.root()`, was already root-mounted.
- **`/api` exists only in never-run tests**, written in `2351771` (2019-04-01, "Add tests structure
  with ignored cases, futurely fix").
- **The author chose root when real code arrived.** In `6a09793` (2019-09-02, user features), six
  test paths changed from `/api/users…` and `/api/user` to `/users…` and `/user`
  (`UserControllerTest` ×4, `HttpUtil` ×2).
- **The bundled spec runner treats `/api` as part of the base URL.** `spec-api/run-api-tests.sh:6`
  defaults `APIURL` to `https://conduit.productionready.io/api`; all 31 Postman requests are
  `{{APIURL}}/users`, `{{APIURL}}/articles`, and so on. `README.md:95` runs it against this app with
  `APIURL=http://localhost:8080`.
- `.travis.yml:8` uses `APIURL=http://localhost:7000/api`, which matches neither this app's port nor
  its base. Likely carried over from another project; unverified.

**Cost of moving everything to `/api`:** changes the URL of every working endpoint and reverses
`6a09793`. Touches `AppConfig.kt`, `UserControllerTest.kt` ×4, `HttpUtil.kt` ×2,
`compose.yaml:13` (healthcheck), `README.md:95`, `.justfiles/docker.just` smoke recipe ×6
(`:83,93,99,102,107,112`), and `.justfiles/security.just:145`.

**Cost of staying at root:** no change to existing behavior. Reusing the author's article tests means
replacing `/api/articles` with `/articles` in their paths (and `HttpUtil.kt:70`). The brief's
`/api/articles/search` becomes `/articles/search` under this app's base URL, which has to be
documented.
