# Context

## The repository

- **Upstream:** `github.com/Rudge/kotlin-ktor-realworld-example-app`
- **Fork (the submission):** `github.com/lancekrogers/kotlin-ktor-realworld-example-app`
- **Local:** `projects/kotlin-ktor-realworld-example-app` (camp submodule), on `master` at `bf1435e`
- **Stack:** Kotlin 1.9.25, Ktor 1.2.3, Exposed 0.41.1, H2 (in-memory), Kodein DI, java-jwt,
  HikariCP, JUnit 4, Unirest (test client), Gradle 8.14
- A [RealWorld](https://codebase.show/projects/realworld) implementation — the reference spec
  defines canonical request/response shapes. A Postman collection is bundled in the repo, which is
  what R9's bonus spec tests would drive.

### Architecture as it actually is

```
web/Router.kt          → mounts users, user, profiles/{username}, articles, tags  (no /api prefix)
web/controllers/       → UserController  ✔ working
                         TagController   ✔ working
                         ProfileController  ✖ stub — bodies commented out
                         ArticleController  ✖ stub — intended service API left in comments
                         CommentController  ✖ stub
domain/                → Article, Comment, Profile, Tag models + DTOs ✔ complete
domain/service/        → UserService ✔ (incl. getProfileByUsername, follow, unfollow), TagService ✔
domain/repository/     → UserRepository ✔ (Users + Follows, incl. follow/unfollow/findIsFollowUser)
                         TagRepository  ✔ (Tags: unique name, findAll only)
                         — no article/comment/favorite repository, and there never was one
test/…/controllers/    → UserControllerTest ✔ 4 enabled
                         Article/Comment/Profile/TagControllerTest ✖ @Ignore — the author's spec
                         for the unbuilt subsystem, written against /api paths
spec-api/              → Postman collection (31 requests) + newman runner; APIURL carries the base
config/AppConfig.kt    → Ktor module wiring, StatusPages, in-memory H2 datasource
```

The layering is conventional and consistent where it is implemented. New work should follow
controller → service → repository and mount in `Router.kt` the same way the existing routes do.

## Prior art in this camp

### Security audit and remediation — merged as PR #1 (`bf1435e`, 6 commits)

Full record: `workflow/explore/ktor-realworld-repo-review/security/` —
`SECURITY_AUDIT.md` (19 findings), `findings-supply-chain.md` (A1–A10),
`findings-application.md` (B1–B9), `go-no-go.md`, `REMEDIATION.md`.

Started from the user's instruction to treat the repo as untrusted third-party code: static
analysis only, nothing executed, report before acting. No malicious code was found. What changed:

- Toolchain: Gradle 4.10 → 8.14 (+`distributionSha256Sum`), Kotlin 1.3.+ → 1.9.25,
  Exposed 0.14.1 → 0.41.1, all versions pinned. **This is why R3's JDK 17/21 matrix is possible at
  all** — Gradle 4.10 cannot run on either JDK.
- Supply chain: `jcenter()` (decommissioned) and `mavenLocal()` removed; `buildscript` block
  replaced with the `plugins` DSL, eliminating dynamic versions resolved at configuration phase.
- Auth: committed HMAC key and HMAC-as-password-hash replaced with bcrypt (cost 12, SHA-512
  pre-hash); `JWT_SECRET` from the environment; JWT now verifies issuer **and** audience.
- Password material no longer echoed in responses (separate `UserResponse` DTO).
- Containerization: `Dockerfile`, `compose.yaml`, modular `Justfile` + `.justfiles/`
  (`build`, `test`, `docker`, `security`).

**Deliberately left undone:** the CI workflow, still pinned to JDK 16 against a JVM 17 build. R3 is
where that gets fixed. Note master's CI was already failing before the PR, for worse reasons.

### Explore workitem — `workflow/explore/ktor-realworld-repo-review/`
WI-5205b8, still active. Steps 1–2 (audit, go/no-go) are done; steps 3–4 (`THE_ASK.md`,
`CODEBASE.md`) were never written, because this festival supersedes them — the INGEST output specs
now carry that material. `camp fresh` correctly declined to auto-promote it.

### Explore workitem — `workflow/explore/interview-exercise-brief/`
WI-da13e4. Landed the PDF → markdown conversion now serving as `input_specs/exercise-brief.md`.

### Agent usage record — `input_specs/agent-usage-record.md`
Raw material for R4, captured from the audit sessions because it exists nowhere else. Includes the
three agent mistakes worth reporting: the `@JsonProperty(WRITE_ONLY)` change that broke all four
tests by stripping passwords from outgoing *requests*, the justfile `root :=` misresolution that made
three security greps silently pass against an empty tree, and a confidently wrong claim about
Gradle 4.10 on JDK 16.

## Key references

- Exercise brief (transcribed): `input_specs/exercise-brief.md`; original PDF at
  `docs/interview-exercise.pdf`
- Verified codebase state: `input_specs/codebase-state-verified.md`
- [Gradle/JDK compatibility matrix](https://docs.gradle.org/current/userguide/compatibility.html) —
  the authority behind C4 (≥7.3 for JDK 17, ≥8.5 for JDK 21)
- [RealWorld API spec](https://realworld-docs.netlify.app/specifications/backend/endpoints/) —
  canonical endpoint shapes, relevant to R9 and R11

## Decisions at the INGEST step 5 checkpoint

**Features (R1, R7, C10).** User direction: ship all three named features. Delivery order: foundation →
Search → Popular → User Activity, each slice green and committed before the next begins.

**Route base (R11).** Recommendation, pending approval: keep root, and document how the brief's `/api/…`
maps onto it. Evidence is in `input_specs/codebase-state-verified.md` §Route base.

**Open semantics (R15).** Search's match rule, Popular's tie-break, User Activity's `favoritesCount`, and
the meaning of `articlesCount`. Proposals are in R15; the binding records get written in
`002_PLAN/decisions/`.
