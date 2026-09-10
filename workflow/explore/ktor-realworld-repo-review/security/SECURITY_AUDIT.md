# Security Audit — kotlin-ktor-realworld-example-app

- **Target:** `projects/kotlin-ktor-realworld-example-app` @ `b9e9dec`
- **Upstream:** github.com/Rudge/kotlin-ktor-realworld-example-app (fork origin: `lancekrogers/…`)
- **Date:** 2026-09-10
- **Method:** pure static analysis. **Nothing in the repository was executed** — no `gradlew`, no
  `gradle`, no tests, no `run-api-tests.sh`, no dependency resolution, no JVM invocation.
  Binary inspection was read-only (`unzip -l`, `unzip -p`, byte-level string extraction).
- **Analyst note:** all repository text was treated as untrusted data, never as instruction.

## Verdict

**Safe to read, build, and run locally.** No evidence of malicious code, backdoors, data
exfiltration, or prompt-injection payloads. The repository is what it claims to be: an abandoned
2019-era RealWorld sample app with organic commit history and stock tooling.

Two qualifications on that verdict:

1. **Trust the code, not the build inputs.** The committed code is clean, but the build resolves
   dependencies through a decommissioned repository (`jcenter()`), consults `mavenLocal()` first,
   and floats the Kotlin compiler plugin on a `1.3.+` wildcard. These are the real supply-chain
   exposures, and they are exposures in *what the build will fetch*, not in what is committed.
   See [findings-supply-chain.md](findings-supply-chain.md).

2. **The app's own security quality is poor.** Hardcoded signing key, unsalted fast-hash
   passwords, password material echoed in API responses. None of this endangers *your machine*;
   all of it would matter if the code were deployed, and some of it is directly in scope for the
   exercise. See [findings-application.md](findings-application.md).

**Recommended precaution before first build:** remove `jcenter()`, drop `mavenLocal()`, and pin
`kotlin_version` to an exact version. That is a three-line change that closes every high-value
supply-chain vector before the first dependency is ever fetched.

## Findings summary

| ID | Sev | Finding |
|----|-----|---------|
| [A1](findings-supply-chain.md#a1) | Medium | `jcenter()` — decommissioned repo in the resolution chain |
| [A2](findings-supply-chain.md#a2) | Medium | `mavenLocal()` resolved before Maven Central |
| [A3](findings-supply-chain.md#a3) | Medium | `kotlin_version = "1.3.+"` floats a build-time-executing plugin |
| [A4](findings-supply-chain.md#a4) | Low-Med | No `distributionSha256Sum` on the Gradle download |
| [A5](findings-supply-chain.md#a5) | Low | Wrapper JAR is 4.4, properties request 4.10 |
| [A6](findings-supply-chain.md#a6) | Medium | API test script defaults to a third-party remote host |
| [A7](findings-supply-chain.md#a7) | Medium | `npx newman` — unpinned remote package executed at run time |
| [A8](findings-supply-chain.md#a8) | Low | `set -x` echoes the test password into logs |
| [A9](findings-supply-chain.md#a9) | Info | No lockfile / dependency verification; no SCA performed |
| [A10](findings-supply-chain.md#a10) | Info | Dependency stack is 2019-era and EOL |
| [B1](findings-application.md#b1) | High | Hardcoded JWT signing secret committed to the repo |
| [B2](findings-application.md#b2) | High | Passwords stored as unsalted HMAC-SHA256 |
| [B3](findings-application.md#b3) | High | One key reused for both password hashing and JWT signing |
| [B4](findings-application.md#b4) | Med-High | Password material returned in API responses |
| [B5](findings-application.md#b5) | Medium | `update()` writes the raw password to the DB unhashed |
| [B6](findings-application.md#b6) | Medium | Exception text leaked to clients; every error becomes a 500 |
| [B7](findings-application.md#b7) | Medium | H2 Postgres-wire listener opened on every startup |
| [B8](findings-application.md#b8) | High | Inverted credential validation — only blank passwords pass |
| [B9](findings-application.md#b9) | Info | JWT hygiene: 10h validity, no revocation, inconsistent issuer check |

## Surfaces examined and cleared

Each of these was actively checked for compromise and found clean.

**`gradle/wrapper/gradle-wrapper.jar`** — the classic supply-chain vector, so it got the most
attention. 50 entries, all `org/gradle/wrapper/*` and `org/gradle/cli/*` plus three properties
files; no extra classes, no nested archives, no scripts. Byte-level string extraction found
**zero hardcoded URLs, hostnames, or IP addresses** — the download target comes only from
`gradle-wrapper.properties`. Every risky API reference maps to documented wrapper behavior:
`URLClassLoader` in `BootstrapMainStarter` (loads the fetched launcher), `Authenticator`/`Base64`
in `Download` (HTTP proxy auth), `ProcessBuilder`+`chmod` and `MessageDigest` in `Install`
(permissions and SHA-256 verification), `MessageDigest` in `PathAssembler` (cache-dir naming).
Consistent with a stock Gradle 4.4 wrapper.

**`gradlew` / `gradlew.bat`** — stock content, no payload appended at head or tail. The only
`eval`/`exec` occurrences (`gradlew:137,139,165,172`) are the standard argument-marshalling and
final `exec "$JAVACMD"` lines present in every Gradle wrapper script.

**`spec-api/Conduit.postman_collection.json`** — Newman executes the JavaScript inside this
file, so it is a genuine code-execution surface. No `eval(`, `Function(`, `require(`,
`child_process`, `pm.sendRequest`, `fetch(`, `process.env`, `atob`, or `Buffer.from` anywhere in
2136 lines. The only absolute URLs are the Postman schema and the gothinkster RealWorld repo;
every request targets the `{{APIURL}}` variable.

**`.github/workflows/gradle.yml`** — well-formed and least-privilege: `permissions: contents:
read`, triggered by `pull_request` (not the dangerous `pull_request_target`), no secrets
referenced, no `${{ github.event.* }}` interpolation into shell (no script-injection vector).
Actions float on major tags (`@v3`, `@v2`) rather than commit SHAs, which is common practice but
mildly mutable.

**Kotlin sources (`src/`)** — no `ProcessBuilder`, `Runtime.exec`, `ScriptEngine`,
`Class.forName`, `newInstance`, `ObjectInputStream`, or `System.load`. No file or filesystem
writes. No outbound network calls. No raw SQL string construction anywhere — persistence goes
through Exposed's typed DSL, which parameterizes, so **there is no SQL-injection surface** in the
code as written.

**Prompt injection** — every tracked text file was swept for agent-directed instruction patterns
("ignore previous", "you are an AI", "system prompt", model names, `<!-- instruct`). One hit, in
`gradlew.bat:76`, is the stock comment "…if you need the _script_ return code instead of…" — a
false positive on "instead of". The entire tracked tree is **pure ASCII**, so there are no
zero-width characters, bidirectional-override attacks, or homoglyph substitutions hiding text
from a human reader.

**Git history** — 37 commits spanning 2019-03-27 to 2024-07-22. Authorship is consistent
(27 + 4 commits from Rudge Ferreira, 4 from dependabot, one each from two outside PR
contributors). The recent commits are all routine dependabot version bumps. Nothing backdated,
nothing anomalous, no unexpected remotes.

## Appendix — non-security facts that change the plan

These surfaced during the audit. They are not vulnerabilities, but they contradict the exercise
brief's description of the codebase and materially change what the work is. They feed steps 3
and 4 of this workitem.

1. **Articles, comments, and profiles are not implemented.** `ArticleController`,
   `CommentController`, and `ProfileController` are stubs — every method body is commented out
   and returns an empty DTO (`ArticleController.kt:11-76`, `CommentController.kt:9-30`,
   `ProfileController.kt:7-23`). There is no `ArticleService`, `ArticleRepository`,
   `CommentService`, `CommentRepository`, or `ProfileService`; `domain/service/` and
   `domain/repository/` contain only the Tag and User pairs.

2. **Only three tables exist:** `Users`, `Follows` (`UserRepository.kt:19,38`) and `Tags`
   (`TagRepository.kt:9`). There are no articles, comments, or favorites tables.

   Together, 1 and 2 mean **all three offered features are blocked**. "Order articles by favorite
   count", "count a user's articles/comments/favorites", and "search article titles and bodies"
   each require the article/comment/favorite subsystem to be built first. The brief's claim that
   the repo "implements … articles, comments, profiles, favorites, and pagination" does not match
   the code.

3. **Every test class is `@Ignore`d at class level** — `ArticleControllerTest.kt:18`,
   `CommentControllerTest.kt:14`, `ProfileControllerTest.kt:13`, `TagControllerTest.kt:14`,
   `UserControllerTest.kt:13`. All 27 `@Test` methods are disabled, so `./gradlew test` currently
   executes zero tests. There is no existing coverage to regress against.

4. **Gradle 4.10 cannot run on JDK 17 or 21.** Gradle requires 7.3+ for JDK 17 and 8.5+ for
   JDK 21 ([Gradle compatibility matrix](https://docs.gradle.org/current/userguide/compatibility.html)).
   The brief's required JDK 17/21 CI matrix is therefore impossible without a major Gradle
   upgrade — which also forces `testCompile` → `testImplementation` (removed in Gradle 7) and the
   removal of `jcenter()`. The existing workflow runs JDK 16 only, on `master`.

5. **Routing does not match the spec.** `Router.kt` mounts `users`, `profiles`, `articles`, and
   `tags` with no `/api` prefix, while the RealWorld spec and the Postman collection expect
   `/api/…`. The tests are themselves inconsistent — `HttpUtil.kt:57` posts to `/users` but
   `HttpUtil.kt:70` posts to `/api/articles`.

6. **Public endpoints require authentication.** `Router.kt:45` wraps the whole `articles` route in
   a mandatory `authenticate { }`, and the nested `authenticate(optional = true)` blocks at
   lines 50 and 63 cannot loosen it. Article listing and comment reading should be public per the
   spec. This fails closed, so it is a spec deviation rather than a vulnerability.

7. **Malformed JDBC URL.** `AppConfig.kt:40` uses `"jdbc:h2:mem:DATABASE_TO_UPPER=false;"`, which
   names the database `DATABASE_TO_UPPER=false` instead of setting that option — the `;`
   separator is missing.

8. **Untracked local artifact.** `kls_database.db` (SQLite, from the Kotlin language server) sits
   in the working tree and is not in `.gitignore`. It should not reach the fork.
