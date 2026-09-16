# Task 01: readme_api_notes — evidence

## Changes

| File | Description |
|---|---|
| `README.md` | Added `# API additions` and `# CI` sections; updated `# Development` recipe list; replaced stale `# Getting started` with Docker-only workflow |

```
 README.md | 148 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++----
 1 file changed, 139 insertions(+), 9 deletions(-)
```

```
 M README.md
```

## Commands

### `just docker up` — exit 0

```
up at http://localhost:18080
```

### API capture curls (setup + three endpoints + errors) — exit 0

```
=== SETUP ===
register_a=200
register_b=200
create1=200
create2=200
favorite=200
comment=200
USER_A=readme_a_1789553063 USER_B=readme_b_1789553063 SLUG2=popular-readme-1789553063-favorite-target
=== SEARCH SUCCESS ===
{"articles":[{"slug":"zephyrine-readme-1789553063-article","title":"Zephyrine readme 1789553063 article","description":"desc","body":"Body with zephyrine keyword 1789553063","tagList":["readme"],"createdAt":"2026-09-16T10:04:24.310+00:00","updatedAt":"2026-09-16T10:04:24.310+00:00","favorited":false,"favoritesCount":0,"author":{"username":"readme_a_1789553063","bio":null,"image":null,"following":false}}],"articlesCount":1}
HTTP 200
=== SEARCH MISSING Q ===
{"errors":{"body":["q is required."]}}
HTTP 422
=== SEARCH LIMIT 101 ===
{"errors":{"body":["limit must be between 1 and 100."]}}
HTTP 422
=== POPULAR SUCCESS ===
{"articles":[{"slug":"popular-readme-1789553063-favorite-target","title":"Popular readme 1789553063 favorite target","description":"desc","body":"body","tagList":[],"createdAt":"2026-09-16T10:04:24.334+00:00","updatedAt":"2026-09-16T10:04:24.334+00:00","favorited":false,"favoritesCount":1,"author":{"username":"readme_b_1789553063","bio":null,"image":null,"following":false}},{"slug":"zephyrine-readme-1789553063-article","title":"Zephyrine readme 1789553063 article","description":"desc","body":"Body with zephyrine keyword 1789553063","tagList":["readme"],"createdAt":"2026-09-16T10:04:24.310+00:00","updatedAt":"2026-09-16T10:04:24.310+00:00","favorited":false,"favoritesCount":0,"author":{"username":"readme_a_1789553063","bio":null,"image":null,"following":false}}],"articlesCount":2}
HTTP 200
=== POPULAR LIMIT 0 ===
{"errors":{"body":["limit must be between 1 and 100."]}}
HTTP 422
=== STATS USER A ===
{"stats":{"articlesCount":1,"commentsCount":1,"favoritesCount":1}}
HTTP 200
=== STATS USER B ===
{"stats":{"articlesCount":1,"commentsCount":0,"favoritesCount":0}}
HTTP 200
=== STATS UNKNOWN ===
{"errors":{"body":["Profile not found."]}}
HTTP 404
=== INVALID TOKEN SEARCH ===

HTTP 401
=== AUTH SEARCH WITH TOKEN ===
{"articles":[{"slug":"zephyrine-readme-1789553063-article","title":"Zephyrine readme 1789553063 article","description":"desc","body":"Body with zephyrine keyword 1789553063","tagList":["readme"],"createdAt":"2026-09-16T10:04:24.310+00:00","updatedAt":"2026-09-16T10:04:24.310+00:00","favorited":false,"favoritesCount":0,"author":{"username":"readme_a_1789553063","bio":null,"image":null,"following":false}}],"articlesCount":1}
HTTP 200
```

### Additional error captures — exit 0

```
=== BLANK Q ===
{"errors":{"body":["q is required."]}}
HTTP 422
=== NEGATIVE OFFSET ===
{"errors":{"body":["offset must not be negative."]}}
HTTP 422
=== INVALID TOKEN STATS ===

HTTP 401
```

### `just docker down` — exit 0

```
stopped
```

### `just security secrets` — exit 0

```
  ok    no hardcoded credentials in src/main
```

### `just test all` — exit 0

```
BUILD SUCCESSFUL in 2s
5 actionable tasks: 1 executed, 4 up-to-date
```

### `just test census` — exit 0

```
  PagingTest                 ran=8   passed=8   failed=0   skipped=0
  ArticleFavoritesRepositoryTest ran=5   passed=5   failed=0   skipped=0
  ArticleFollowingMappingTest ran=1   passed=1   failed=0   skipped=0
  ArticleSchemaTest          ran=1   passed=1   failed=0   skipped=0
  ArticleSearchRepositoryTest ran=8   passed=8   failed=0   skipped=0
  LowerOnClobProbeTest       ran=1   passed=1   failed=0   skipped=0
  ArticleServiceTest         ran=5   passed=5   failed=0   skipped=0
  SlugTest                   ran=7   passed=7   failed=0   skipped=0
  ArticleControllerTest      ran=6   passed=6   failed=0   skipped=11
  ArticleCreateTest          ran=9   passed=9   failed=0   skipped=0
  ArticleSearchTest          ran=11  passed=11  failed=0   skipped=0
  CommentControllerTest      ran=1   passed=1   failed=0   skipped=2
  CommentCreateTest          ran=5   passed=5   failed=0   skipped=0
  PopularArticlesTest        ran=11  passed=11  failed=0   skipped=0
  ProfileControllerTest      ran=0   passed=0   failed=0   skipped=3  <-- entire class disabled
  ProfileStatsTest           ran=5   passed=5   failed=0   skipped=0
  TagControllerTest          ran=1   passed=1   failed=0   skipped=0
  UserControllerTest         ran=4   passed=4   failed=0   skipped=0
  JsonAssertionsTest         ran=5   passed=5   failed=0   skipped=0

  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

JUnit XML (`build/test-results/test/*.xml`): tests=110, passed=94, failed=0, skipped=16.

## Done When

- [x] **All requirements met** — pass: `# API additions` documents base-path mapping (D002), all three read endpoints with parameters/errors/captured JSON, `articlesCount` total-match semantics (D007), stats counts with given-vs-received distinction (D006), public reads with invalid-token 401 (D003); `# Getting started` is Docker-only with no host `./gradlew run`; `# Development` adds `just build matrix` and `just docker spec`; `# CI` covers both jobs, failure surfaces, and `expected-failures.txt` rule.
- [x] **`README.md` has `# API additions` and `# CI` sections that cover every item in both requirements, with no host `./gradlew run` instructions left in Getting started, and every JSON example traceable to a captured response** — pass: `grep gradlew README.md` finds only the CI job description (`./gradlew build` in Actions); all three JSON examples match the container capture output above verbatim.

## Notes

- Branch/camp-fresh step skipped per orchestrator instruction (already on `docs/submission`).
- No file:line anchor drift; task cited `README.md:79-95` for Getting started and `:54` for Development — both matched.
- Task cited `.justfiles/docker.just:5` for port; actual `host_port` default is `"18080"` at line 5.
- JSON examples use captured responses from `just docker up` session on 2026-09-16; no real JWT tokens appear in README (`just security secrets` clean).
- Stubbed endpoints and implemented write routes documented per orchestrator guidance to avoid overstating completeness.
- Nothing unresolved.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `fd730ea8-89fb-4ec3-a2a8-0ef71d576b13`, 10:02:52Z to 10:05:28Z).

**Both required sections exist** (`# API additions` at `README.md:81`, `# CI` at `:206`), and the stale content the
task named is gone:

```text
$ grep -nE 'JVM installed|gradlew run|localhost:7000' README.md
(no stale host instructions or wrong-port links)
$ grep -n '18080' README.md
65:just docker up          # start the app on http://localhost:18080
199:just docker up      # app at http://localhost:18080
```

The old section claimed "You need just JVM installed" and linked a label reading 8080 to port 7000; both are replaced
by the Docker-only workflow, and the host `./gradlew run` instructions that contradicted C7 are removed.

**The D006 distinction is documented in the form that prevents the mistake**, not just mentioned:

```text
| `favoritesCount` | articles the user **favorited** (favorites **given**) |

`Article.favoritesCount` on an article counts favorites that article **received** — the opposite
direction from a profile's `favoritesCount`.

A user who authored one article but gave no favorites returns
{"stats":{"articlesCount":1,"commentsCount":0,"favoritesCount":0}} even when their article received
favorites from others.
```

That worked example is the exact scenario the orchestrator's container check used to distinguish the two readings
(`05_user_activity/results/02_profile_stats_service_and_route.md`). The review gate asked for this to be written down;
it is, with the counter-example rather than only the rule.

**Error messages match what the running app actually returns.** The README quotes `q is required.`,
`limit must be between 1 and 100.` and `Profile not found.` — all three verified verbatim against containers during
earlier slices.

**Completeness is not overstated.** A "Still stubbed" list names article list and filters, the personal feed, get,
update and delete by slug, comment list and delete, and profile get, follow and unfollow. A callout also warns that
`GET /articles/feed` without `/popular` is still stubbed and requires a token — the distinction most likely to confuse
a reader given the new popular endpoint's path.

**No secrets.** `just security secrets` passes, and `grep -nE 'eyJ[A-Za-z0-9_-]{10,}'` finds no JWT-shaped string, so
the captured examples carry no real token.

**Observation, not a defect.** The favorite and comment endpoints appear in a compact table under "Implemented write
routes (for exercising the reads)" with their statuses and error conditions (404 unknown slug, 422 blank body, 401
without a token) but without verbatim JSON error bodies, unlike the three new read endpoints. The task required exact
messages for the new endpoints, which is satisfied; documenting the supporting write routes more briefly is an
editorial choice and does not misstate behavior.
