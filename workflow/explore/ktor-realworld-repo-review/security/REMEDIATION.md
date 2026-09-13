# Remediation record

Companion to [SECURITY_AUDIT.md](SECURITY_AUDIT.md). Records what was changed, and — separately —
what was actually *verified* versus merely written.

- **Decision:** [go-no-go.md](go-no-go.md) — GO, containerized, full security remediation
- **Date:** 2026-09-10
- **Environment:** every build, test, and run happened inside Docker. Nothing was executed on the host.
- **Toolchain after remediation:** Gradle 8.14, Kotlin 1.9.25, JDK 17 (also verified on JDK 21)

## Status

| ID | Finding | Status |
|----|---------|--------|
| A1 | `jcenter()` in resolution chain | Fixed — removed; `buildscript` block eliminated entirely via the `plugins` DSL |
| A2 | `mavenLocal()` before Central | Fixed — removed |
| A3 | `kotlin_version = "1.3.+"` floating | Fixed — all versions pinned in `gradle.properties` |
| A4 | No distribution checksum | Fixed — `distributionSha256Sum` pinned to the published Gradle 8.14 sum |
| A5 | Wrapper JAR 4.4 vs 4.10 drift | Fixed — wrapper regenerated at 8.14 |
| A6 | Test script defaults to remote host | Open — deferred, see below |
| A7 | Unpinned `npx newman` | Open — deferred, see below |
| A8 | `set -x` leaks password | Open — deferred, see below |
| A9 | No SCA performed | Still open — unchanged; needs dependency resolution |
| A10 | 2019-era EOL stack | Partly — Gradle/Kotlin/Exposed current; Ktor 1.2.3 unchanged |
| B1 | Hardcoded JWT signing secret | Fixed — loaded from `JWT_SECRET`, ephemeral random fallback, 32-char minimum |
| B2 | Unsalted HMAC password storage | Fixed — bcrypt cost 12, per-hash salt, SHA-512 pre-hash for >72-byte inputs |
| B3 | One key for hashing and signing | Fixed — bcrypt carries its own salt; the JWT key is now unrelated |
| B4 | Password material in responses | Fixed — separate `UserResponse` DTO |
| B5 | `update()` wrote raw password | Fixed — hashing moved into `UserService`; the repository never sees a raw password |
| B6 | Exception text leaked; all errors 500 | Fixed — `ErrorExceptionMapping` maps 401/404/422, generic 500, detail logged server-side |
| B7 | H2 Postgres listener on startup | Fixed — removed; container now listens on 8080 only |
| B8 | Inverted credential validation | Fixed — six conditions negated; update validates only supplied fields |
| B9 | JWT hygiene | Fixed — verifier asserts issuer *and* audience; `decodeJWT` reuses it |

## Verified by execution

Each of these was observed against the running container, not inferred from the source.

- **Registration works at all.** `POST /users` with a real password returns 200. Before B8 was
  fixed, the validator required a *blank* password, so no valid registration could succeed.
- **No password material in any response.** Register, login, and `GET /user` all return exactly
  `{email, token, username, bio, image}` — the spec's shape. Confirmed by grepping the response
  bodies.
- **A token forged with the old committed key is rejected.** Signed `{iss: ktor-realworld,
  aud: ktor-audience, email: jake@jake.jake}` with `"something-very-secret-here"` and presented
  it: **401**. That exact token was full account takeover before B1.
- **Secret loading behaves at both edges.** Unset `JWT_SECRET` → ephemeral key plus a warning on
  stderr. `JWT_SECRET=tooshort` → hard startup failure, `IllegalArgumentException: JWT_SECRET
  must be at least 32 characters; got 8`.
- **Password rotation works.** `PUT /user` with a new password → 200; login with the new password
  → 200; login with the old password → 401. Before B5 the update path stored plaintext, which
  locked the account out permanently since `authenticate` compares against a hash.
- **Status codes are correct.** Wrong password → 401 `{"errors":{"body":["email or password
  invalid!"]}}`. Blank password → 422. Missing token → 401. All three were 500 before.
- **No stray listener.** `/proc/net/tcp` inside the container shows port 8080 only; 5435 is gone.
- **Builds green on both required JDKs.** `gradle clean build` succeeds under `gradle:8.14-jdk17`
  and `gradle:8.14-jdk21`.

## Two findings that only surfaced by running it

**The app was already broken before any of this.** With `jcenter()` removed, `exposed:0.14.1`
turns out not to exist on Maven Central — it was only ever published to JCenter, so the original
build is no longer reproducible from any trustworthy source. Worse, once it did resolve, the app
crashed on startup: `NoSuchMethodError: org.h2.jdbc.JdbcConnection.getSession()`. Exposed 0.17.x
expects H2 1.4.x; dependabot had bumped H2 to 2.2.224 (commit `9d1f1bc`) with nothing to catch
the incompatibility. Exposed was moved to 0.41.1 (modular artifacts, H2 2.x support), which
required migrating `LongIdTable` to `org.jetbrains.exposed.dao.id`, replacing
`Column.primaryKey()` with a table-level `PrimaryKey`, and importing `SqlExpressionBuilder.eq`
for `deleteWhere`.

Nobody noticed because every test was disabled and CI could not run — Gradle 4.10 cannot start on
the JDK 16 the workflow requests.

**The existing tests asserted the vulnerability.** `UserControllerTest` contained
`assertEquals(response.body.user?.password, userDTO.user?.password)` (register must echo the
plaintext back) and `assertNotNull(response.body.user?.password)` (`GET /user` must return the
password). Enabling them unchanged would have actively blocked the B4 fix. Both now assert the
opposite, and are the regression guard for it.

Worth recording as a method note: the first attempt at B4 used
`@JsonProperty(access = WRITE_ONLY)` on `User.password`. It compiled, and it looked right. But
`User` is shared between requests and responses, so the annotation also stripped the password
when the *test client* serialized a registration body — every request silently arrived with no
password. Only running the tests caught it. That is why the final fix is a separate
`UserResponse` type: it cannot be defeated by a shared class or forgotten on a new endpoint.

## Deliberately not done

- **A6 / A7 / A8 (`spec-api/run-api-tests.sh`).** Untouched. The script is only reachable by
  running it deliberately, and under the containerized decision it cannot reach the host. Fix
  before wiring the RealWorld spec tests into CI: default `APIURL` to localhost, pin the newman
  version, drop `set -x`.
- **Ktor 1.2.3 → 2.x/3.x.** Ktor 1.2.3 is on Maven Central and compiles cleanly under Kotlin
  1.9.25, so it does not block the build. The upgrade is a wide mechanical rename across every
  controller and would swamp the diff. Scoped, deferred, recorded here.
- **Re-enabling the other 21 tests.** `ArticleControllerTest` (14), `CommentControllerTest` (3),
  `ProfileControllerTest` (3), and `TagControllerTest` (1) all exercise controllers that are
  stubs returning empty DTOs. They cannot pass until that subsystem exists, so they stay
  `@Ignore`d rather than being force-fixed into a false green.
- **The `/api` prefix and the public-route auth nesting** (appendix items 5 and 6). Both are spec
  deviations rather than vulnerabilities and both change routing behaviour, which belongs with the
  feature work, not a security pass.

## Honest limits

- **Test coverage is 4 of 25.** The build is green, but 21 tests are skipped. A green build here
  is *not* evidence the application works; it is evidence that what runs, passes.
- **No SCA / CVE scan** (A9, unchanged). The dependency tree has not been checked against any
  vulnerability database.
- **The verification above is a manual smoke run**, not an automated suite. Only the four user
  tests are repeatable in CI. Article, comment, profile, and favourite behaviour is unverified
  because it is unimplemented.
- **bcrypt cost 12 was not benchmarked** on target hardware; it is a reasonable default, not a
  measured one.
