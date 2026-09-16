# Task 01: Pin newman and require an explicit APIURL

## Changes

| File | Description |
|------|-------------|
| `spec-api/run-api-tests.sh` | Replaced runner: pin newman@6.2.2, require APIURL, JSON report, remove set -x |
| `spec-api/README.md` | Updated example to `APIURL=http://localhost:8080`; note APIURL is required |
| `.justfiles/docker.just` | Added `spec` recipe using digest-pinned node container on app network |
| `.gitignore` | Ignore `spec-api/newman-report.json` |

```
git diff --stat`:
 .gitignore                |  3 +++
 .justfiles/docker.just    | 13 +++++++++++++
 spec-api/README.md        |  4 +++-
 spec-api/run-api-tests.sh | 13 +++++++++----
 4 files changed, 28 insertions(+), 5 deletions(-)
```

```
git status --short --untracked-files=all`:
 M .gitignore
 M .justfiles/docker.just
 M spec-api/README.md
 M spec-api/run-api-tests.sh
```

## Commands

### Pull node image and get digest

```
$ docker pull node:22-bookworm-slim && docker image inspect node:22-bookworm-slim --format json | python3 -c 'import json,sys; print(json.load(sys.stdin)[0]["RepoDigests"][0])'
```

Exit code: 0

```
22-bookworm-slim: Pulling from library/node
...
Digest: sha256:83f487e0a63425e5b4d146fb5e5be574bcbe1b7b843d3ebafdd95eaf7767a7e5
Status: Downloaded newer image for node:22-bookworm-slim
docker.io/library/node:22-bookworm-slim
node@sha256:83f487e0a63425e5b4d146fb5e5be574bcbe1b7b843d3ebafdd95eaf7767a7e5
```

### APIURL guard check

```
$ env -u APIURL bash spec-api/run-api-tests.sh
```

Exit code: 1

```
spec-api/run-api-tests.sh: line 7: APIURL: set APIURL to the API base, e.g. http://localhost:8080
```

### grep for removed patterns

```
$ grep -nE 'set -x|productionready' spec-api/run-api-tests.sh
```

Exit code: 1 (no matches)

### just docker up

```
$ just docker up
```

Exit code: 0

```
up at http://localhost:18080
```

### just docker spec

```
$ just docker spec
```

Exit code: 1 (expected: stubbed endpoints fail assertions)

Newman ran from digest-pinned `node:22-bookworm-slim@sha256:83f487e0a63425e5b4d146fb5e5be574bcbe1b7b843d3ebafdd95eaf7767a7e5` inside `--network container:ktor-realworld-dev`. Collection executed against `http://localhost:8080`. Exit summary:

```
 18 failed assertions (stubbed list/feed/update/delete/comment list/delete/profiles endpoints)
error: recipe `spec` failed with exit code 1
```

Full CLI output was truncated in the agent capture; key lines included newman 6.2.2 run, requests to `http://localhost:8080/...`, and 18 assertion failures.

### Confirm report on host

```
$ ls -la spec-api/newman-report.json
```

Exit code: 0

```
-rw-r--r--  1 lancerogers  staff  1266228 Sep 16 03:40 spec-api/newman-report.json
```

### just docker down

```
$ just docker down
```

Exit code: 0

```
stopped
```

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

## Done When

- [x] **All requirements met** — pass
  - `run-api-tests.sh` pins `npx --yes newman@6.2.2`, requires APIURL, writes JSON report, no `set -x`
  - `just docker spec` runs from digest-pinned node container
  - `spec-api/newman-report.json` created on host after spec run
  - Branch step skipped per orchestrator (already on `feat/spec-tests`)

- [x] **`just docker spec` runs collection and leaves report; APIURL guard works; grep clean** — pass
  - `just docker spec` exit 1 with newman report at `spec-api/newman-report.json` (1266228 bytes)
  - `env -u APIURL bash spec-api/run-api-tests.sh` exit 1 with message `set APIURL to the API base, e.g. http://localhost:8080`
  - `grep -nE 'set -x|productionready' spec-api/run-api-tests.sh` exit 1 (no matches)

## Notes

- **Branch step skipped:** orchestrator placed work on `feat/spec-tests`; task listed `ci/spec-tests` (D012).
- **Node digest:** `node@sha256:83f487e0a63425e5b4d146fb5e5be574bcbe1b7b843d3ebafdd95eaf7767a7e5` recorded in `results/01_runner.md`.
- **Anchor drift:** none; `docker.just` `verify` recipe still at lines 118–125, `container_name` at line 4.
- **`just docker spec` non-zero exit:** expected per task (stubbed endpoints); task 02 adds the expected-failures gate.
- **Test counts:** TOTAL ran=94 passed=94 failed=0 skipped=16 (`just test census`).

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `a5fab1a4-c773-42d0-bbda-f2950f83bb76`, 09:37:55Z to 09:40:17Z).

**The guard is real, and it is a security fix rather than ergonomics.** Before this task, an unset `APIURL` defaulted
the collection to a remote production-looking host, and `set -x` echoed the generated password into the logs. Verified
independently:

```text
$ env -u APIURL bash spec-api/run-api-tests.sh
exit=1
spec-api/run-api-tests.sh: line 7: APIURL: set APIURL to the API base, e.g. http://localhost:8080
newman never started (correct)
$ grep -nE 'set -x|productionready' spec-api/run-api-tests.sh
(clean)
$ git ls-files -s spec-api/run-api-tests.sh
100755 …   # still executable, so task 03's CI job can run it directly
```

The failure happens at the parameter expansion on line 7, before `npx` is reached — the output contains no mention of
newman at all.

**The image is digest-pinned, matching the CI slice's convention:**

```text
.justfiles/docker.just:138  node:22-bookworm-slim@sha256:83f487e0a63425e5b4d146fb5e5be574bcbe1b7b843d3ebafdd95eaf7767a7e5
```

A `node:22` tag would have reintroduced exactly the floating-dependency risk that slice 1 removed by pinning every
GitHub action to a commit SHA. The recipe also refuses to run when the app container is absent, and joins that
container's network namespace so the app answers on `localhost:8080` — necessary because a container cannot reach the
host's localhost on macOS or colima.

**Independent live run against the real app container:**

```text
$ just docker up && just docker spec
spec_exit=1
report bytes = 1266232
requests:   {'total': 31, 'pending': 0, 'failed': 0}
assertions: {'total': 112, 'pending': 0, 'failed': 16}
testScripts:{'total': 46, 'pending': 0, 'failed': 2}
failures recorded: 18
```

What that establishes:

- **The collection genuinely reaches the app.** 31 requests with **0 transport failures**: every request got an HTTP
  response, so the container networking approach works. A misconfigured `APIURL` would have shown failed requests
  instead.
- **16 of 112 assertions fail**, which is the expected shape while list, feed, get-by-slug, update, delete,
  comment list/delete and the profile endpoints remain stubbed (D001). The non-zero exit is by design until task 02
  compares against an expected-failures manifest.
- **The JSON report lands on the host** at `spec-api/newman-report.json`, which is what task 02's comparator consumes,
  and `.gitignore:47` keeps it out of git.

`spec-api/README.md` now documents `APIURL=http://localhost:8080 ./run-api-tests.sh` and states that `APIURL` is
required.

**Baseline for task 02:** 112 assertions, 16 failing, 18 failure entries, across 31 requests. The manifest task 02
writes should match that set exactly; any difference means either a real regression or a stale manifest.
