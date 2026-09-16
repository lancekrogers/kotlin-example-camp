# Task 02: Expected-failures manifest and comparator

## Changes

| File | Description |
|------|-------------|
| `spec-api/compare_results.py` | Stdlib comparator: keys by `folder / request name`, detects assertion, testScript, and requestError failures |
| `spec-api/expected-failures.txt` | Manifest of 18 expected Newman failures grouped by stubbed endpoint (D001, D010) |

```
git diff --stat`:
(no tracked-file diffs; new files are untracked)
```

```
git status --short --untracked-files=all`:
?? spec-api/compare_results.py
?? spec-api/expected-failures.txt
```

## Commands

### Inspect newman report structure

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 -c '
import json
d = json.load(open("newman-report.json"))
ex = d["run"]["executions"][0]
print(sorted(ex.keys()))
print(sorted(ex["item"].keys()))
print(ex.get("assertions", [])[:1])
'
```

Exit code: 0

```
['assertions', 'cursor', 'id', 'item', 'request', 'response']
['event', 'id', 'name', 'request', 'response']
[{'assertion': 'Response contains "user" property', 'skipped': False}]
```

### just docker up

```
$ just docker up
```

Exit code: 0

```
up at http://localhost:18080
```

### just docker spec (generate fresh newman-report.json)

```
$ just docker spec
```

Exit code: 1 (expected; 18 failing requests)

Newman stats from `newman-report.json`:

```
requests: {'total': 31, 'pending': 0, 'failed': 0}
assertions: {'total': 112, 'pending': 0, 'failed': 16}
testScripts: {'total': 46, 'pending': 0, 'failed': 2}
```

### Empty manifest (discover failing set)

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 compare_results.py newman-report.json /dev/null
```

Exit code: 1

```
UNEXPECTED FAILURES (regressions)
  Articles / All Articles
  Articles / Articles Favorited by Username
  Articles / Articles by Author
  Articles / Articles by Tag
  Articles, Favorite, Comments / All Articles
  Articles, Favorite, Comments / All Articles with auth
  Articles, Favorite, Comments / All Comments for Article
  Articles, Favorite, Comments / Articles Favorited by Username
  Articles, Favorite, Comments / Articles Favorited by Username with auth
  Articles, Favorite, Comments / Articles by Author
  Articles, Favorite, Comments / Articles by Author with auth
  Articles, Favorite, Comments / Articles by Tag
  Articles, Favorite, Comments / Feed
  Articles, Favorite, Comments / Single Article by slug
  Articles, Favorite, Comments / Update Article
  Profiles / Follow Profile
  Profiles / Profile
  Profiles / Unfollow Profile
18 failed, 0 expected
```

### Comparator match

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 compare_results.py newman-report.json expected-failures.txt
```

Exit code: 0

```
18 failed, 18 expected
```

### Comparator negative tests

See `results/02_comparator.md` for removed-entry and bogus-entry outputs (both exit 1).

### just test all

```
$ just test all
```

Exit code: 0

```
BUILD SUCCESSFUL in 2s
5 actionable tasks: 1 executed, 4 up-to-date
```

### just test census

```
$ just test census
```

Exit code: 0

```
  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

## Done When

- [x] **All requirements met** — Results keyed by `folder / request name`; comparator uses stdlib only and exits 0 only on exact match; manifest comments name stubbed endpoints; negative comparator tests recorded in `results/02_comparator.md`.
- [x] **`python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt` exits 0** — Evidence: comparator match command above (exit 0, `18 failed, 18 expected`).
- [x] **`results/02_comparator.md` shows exit 1 for both removed-entry and bogus-entry manifests** — Evidence: `results/02_comparator.md`.

## Notes

- **Anchor drift:** `Conduit.postman_collection.json` request items lack `id` fields (only some script blocks have ids). The task template keyed by `item.id`; comparator instead walks the collection in execution order and pairs by index with `report["run"]["executions"]`, producing `folder / name` keys on both sides.
- **testScript errors:** `Single Article by slug` and `Update Article` fail with `JSONError` in `testScript`, not in `assertions` or `requestError`. Comparator extended beyond the task template to treat `testScript` errors as failures (18 requests, matching baseline).
- **Delete Comment / Delete Article:** Return 404 but have no test scripts; Newman does not record them as failures. They are correctly omitted from the manifest.
- **Python via Docker:** No `just` recipe for the comparator yet; all `python3` invocations used `docker run … python:3.12-slim-bookworm` per festival Docker-only rule.
- **Implemented endpoints passing:** Register, login, current user, update user, create article, favorite/unfavorite, add comment, and tags all pass Newman; none appear in the manifest.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `66bf71e7-04f0-4eff-88b0-50bc849498bf`, 09:43:20Z to 09:47:54Z).

An expected-failures manifest is the one artifact here whose purpose is to stop reporting failures, so it was checked
harder than a normal change: a single wrong entry converts a real bug into an accepted baseline and the CI job stays
green while an endpoint is broken.

**The comparator behaves correctly in all three directions** (run independently by the orchestrator):

```text
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt
18 failed, 18 expected                                              exit=0
$ # one entry removed
UNEXPECTED FAILURES (regressions)
  Articles / All Articles
18 failed, 17 expected                                              exit=1
$ # bogus entry added
UNEXPECTED PASSES (update the manifest)
  Nonexistent / Bogus Request
18 failed, 19 expected                                              exit=1
```

**No implemented endpoint is masked.** Every request was paired with its execution and inspected:

```text
request                                                    asserts failed   result  in-manifest
Auth / Register                                                  6      0     pass
Auth / Login                                                     6      0     pass
Auth / Login and Remember Token                                  7      0     pass
Auth / Current User                                              6      0     pass
Auth / Update User                                               6      0     pass
Articles, Favorite, Comments / Create Article                   15      0     pass
Articles, Favorite, Comments / Favorite Article                 17      0     pass
Articles, Favorite, Comments / Unfavorite Article               16      0     pass
Articles, Favorite, Comments / Create Comment for Article         8      0     pass
Profiles / Register Celeb                                        6      0     pass
Tags / All Tags                                                  3      0     pass
… 18 failing requests, all list/feed/get-by-slug/update/comments-list/profiles, all in the manifest
```

Every endpoint this festival implemented passes the official collection, with real assertion counts behind it
(Favorite Article alone asserts 17 times). All 18 manifest entries correspond to endpoints D001 leaves stubbed.

A keyword cross-check initially flagged 12 manifest entries as "naming an implemented feature". That was an artifact:
the Postman folder is named `Articles, Favorite, Comments`, so every request inside it matches "favorite" and
"comments". The per-request table above is the real answer.

### Two findings recorded rather than smoothed over

1. **Two requests can never fail this gate.** `Delete Comment for Article` and `Delete Article` carry **zero
   assertions**, so an assertion-based comparator counts them as passes even though both endpoints are stubbed and
   answer 404. They are correctly absent from the manifest — adding them would make the comparator demand a failure it
   cannot observe — but the gate is blind to those two requests. This is a limitation of the approach, not a defect in
   this task. If a later slice implements delete, nothing in the spec gate will notice either way.
2. **Two entries fail through `testScript` errors, not assertions.** `Single Article by slug` and `Update Article`
   report 0 assertions yet count as failures, because the stubbed response breaks the collection's own test script with
   a JSON parse error. The subagent extended `execution_failed` to treat `testScript` errors as failures after finding
   this; without that change the baseline would have been 16 failing requests and those two would have been invisible
   to the gate.

**Deviation from the task template, handled correctly.** The task's comparator keyed executions by `item.id`, but this
collection's items carry no ids. The subagent switched to ordered `folder / name` pairing and said so. The orchestrator's
independent pairing used the same approach and reproduced the identical 18-request failing set, so the two agree.

**Baseline for task 03:** 31 requests, 0 transport failures, 112 assertions with 16 failing, 46 test scripts with 2
failing, 18 failing requests, manifest of 18 entries, comparator exit 0.
