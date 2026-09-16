# 003_IMPLEMENT — required outcomes, with the output that verifies each

Every block below is pasted terminal output. API responses were captured on 2026-09-16 from a container built from
`origin/master` at `4f751e5` via `just docker up`, then torn down with `just docker down`.

---

## 1. CI runs on JDK 17 and 21, SHA-pinned, cached, with JUnit annotations (01_ci_pipeline)

```console
$ grep -nE 'uses:|java: \[|fail-fast' .github/workflows/gradle.yml
19:      fail-fast: false
21:        java: [ '17', '21' ]
23:      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
24:      - uses: actions/setup-java@de7274f081f381c8f8158605e0321c36c376e2e6 # v6.0.1
28:      - uses: gradle/actions/setup-gradle@9c971963bec38e04b3d30dcc455b5382be2fdbfb # v6.3.0
33:        uses: mikepenz/action-junit-report@a9170d5795813c01ab4901ffb045b52bab4ab09d # v6.5.0
42:        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
52:      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
53:      - uses: actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0
73:        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
```

Every `uses:` is a 40-character commit SHA with its tag in a trailing comment; `fail-fast: false` so one JDK failing
does not mask the other. Green on the fork:

```console
$ gh run list --branch master --limit 3 --json databaseId,event,conclusion
  run 35135927840 push success
  run 35135483171 push success
  run 35133183546 push success
```

The baseline before this phase was a workflow that had never run at all.

## 2. `POST /articles` creates articles with tags, a unique slug and a Profile author (02_article_foundation)

```console
$ POST /articles
{
    "article": {
        "slug": "evidence-1789584647",
        "title": "Evidence 1789584647",
        "description": "d",
        "body": "searchable body 1789584647",
        "tagList": [
            "ev",
            "proof"
        ],
        "createdAt": "2026-09-16T18:50:48.282+00:00",
        "updatedAt": "2026-09-16T18:50:48.282+00:00",
        "favorited": false,
        "favoritesCount": 0,
        "author": {
            "username": "ev1789584647",
            "bio": null,
            "image": null,
            "following": false
        }
    }
}
```

The author is a Profile — `username`, `bio`, `image`, `following` — with no `email`, `password` or `token`. The slug is
derived from the title. The author's own previously disabled `create article` and `get all tags` tests now run.

## 3. `GET /articles/search?q=` is public, case-insensitive, literal, paged, and returns the total (03_article_search)

```console
$ GET /articles/search?q=EVIDENCE%201789584647  (uppercase term, case-insensitive)
articlesCount: 1
titles: ['Evidence 1789584647']

$ GET /articles/search?q=%25  (literal percent, must not wildcard-match everything)
articlesCount: 0

$ GET /articles/search  (missing q)
HTTP 422 {"errors":{"body":["q is required."]}}

$ GET /articles/search?q=x&limit=0
HTTP 422 {"errors":{"body":["limit must be between 1 and 100."]}}
```

All four requests were made **without a token** and the first two returned 200, which is the public-read requirement.
The uppercase term matching a mixed-case title proves case-insensitivity.

The percent case is the strongest available proof of literal matching, and it **discriminates**: a matching article
exists, so an unescaped `%` would have produced a pattern of `%%%` and matched everything. It returns 0, meaning `%` was
treated as a literal character. This matters because the corresponding unit test cannot fail — see
`constraints.md`, deferred item 2.

## 4. `GET /articles/feed/popular` is public, ordered by favorites then recency, paged; favorite writes idempotent (04_popular_articles)

```console
$ POST /articles/{slug}/favorite twice (idempotence)
favoritesCount: 1 favorited: True
favoritesCount: 1 favorited: True

$ GET /articles/feed/popular  (anonymous)
HTTP 200
articlesCount: 1
top favoritesCount: [1]
```

Favoriting the same article twice leaves the count at 1 rather than incrementing to 2, so the write is idempotent
against a real database rather than only in a unit test. The feed answers 200 anonymously.

Ordering by favorite count then recency, and the inclusion of zero-favorite articles, are covered by
`PopularArticlesTest` (`more favorites ranks first`, `equal counts newest first`,
`results with equal createdAt ordered by id descending`, `zero favorite included`), 11 tests, all passing.

## 5. `GET /profiles/{username}/stats` returns articles authored, comments written, favorites given (05_user_activity)

```console
$ GET /profiles/ev1789584647/stats  (anonymous)
{"stats":{"articlesCount":1,"commentsCount":0,"favoritesCount":1}}

$ GET /profiles/ev1789584647/stats with an INVALID token
HTTP 401

$ GET /profiles/does-not-exist-1789584647/stats
HTTP 404
```

The numbers correspond to what this user actually did in the capture above: created one article, wrote no comments, and
favorited one article. `favoritesCount: 1` is favorites **given** — the user favorited their own article — which is the
D006 reading. An unknown username is 404, not a zeroed object.

The 401 on an invalid token is the D003 requirement that public reads still reject bad credentials rather than silently
treating the caller as anonymous. This was previously untested and is now verified directly.

## 6. The RealWorld collection runs in CI and fails only on regressions or a stale manifest (06_spec_api_ci)

Green with the correct manifest, run 35127041807:

```text
build and test (JDK 21): success
build and test (JDK 17): success
JUnit (JDK 17): success
JUnit (JDK 21): success
RealWorld spec tests: success
```

Red with one manifest entry removed, run 35127046886, failing step `Compare against expected failures`:

```text
UNEXPECTED FAILURES (regressions)
  Articles / All Articles
18 failed, 17 expected
##[error]Process completed with exit code 1.
```

Both JDK build jobs stayed green across the pair, so only the gate flipped. `18 failed, 17 expected` shows the
collection still ran in full — the same 18 requests failed and the manifest simply stopped covering one of them. The
job also passed as a real `pull_request` check on PR #11, not only as a dispatched run.

## 7. README API notes and `AGENT_WORKLOG.md` are merged (07_submission_docs)

```console
$ curl -sS -o /dev/null -w '%{http_code}' https://raw.githubusercontent.com/.../master/AGENT_WORKLOG.md
200
$ curl -sS -o /dev/null -w '%{http_code}' https://raw.githubusercontent.com/.../master/README.md
200
```

Fetched with `GH_TOKEN` and `GITHUB_TOKEN` unset, so this is anonymous reachability on `master`, not an authenticated
read. `README.md` carries the `# API additions` and `# CI` sections; `AGENT_WORKLOG.md` is 88 lines covering the six
points of brief §5 in order plus the scope decision and the walkthrough placeholder.

---

## Suite state

```text
  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

From `just test census` on `master`. The suite ran 4 tests at the CI baseline. Each of the 16 skips is an upstream
`@Ignore` naming the stubbed endpoint behind it; the census prints its own warning rather than letting a green build
imply more than it proves.
