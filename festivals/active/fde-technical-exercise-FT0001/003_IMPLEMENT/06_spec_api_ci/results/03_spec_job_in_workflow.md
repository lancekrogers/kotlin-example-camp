# Task 03: Add spec-test job to CI — evidence

## Changes

| File | Description |
|------|-------------|
| `.github/workflows/gradle.yml` | Added `spec` job after `test`: builds Docker image, starts app with generated `JWT_SECRET`, runs collection with `\|\| true`, gates on comparator, uploads newman report with `if: always()`. |

### git diff --stat

```
 .github/workflows/gradle.yml | 31 +++++++++++++++++++++++++++++++
 1 file changed, 31 insertions(+)
```

### git status --short --untracked-files=all

```
 M .github/workflows/gradle.yml
```

## Commands

### Resolve `actions/setup-node` v7.0.0 SHA

```
$ gh api repos/actions/setup-node/releases/latest --jq .tag_name
v7.0.0

$ gh api repos/actions/setup-node/git/ref/tags/v7.0.0 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"820762786026740c76f36085b0efc47a31fe5020","type":"commit"}
```

Exit code: **0**

### YAML parse check

```
$ python3 -c 'import yaml; yaml.safe_load(open(".github/workflows/gradle.yml"))' && echo "YAML OK"
YAML OK
```

Exit code: **0**

### SHA pin check

```
$ grep -nE 'uses: [^@ ]+@[0-9a-f]{40} # v' .github/workflows/gradle.yml
23:      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
24:      - uses: actions/setup-java@de7274f081f381c8f8158605e0321c36c376e2e6 # v6.0.1
28:      - uses: gradle/actions/setup-gradle@9c971963bec38e04b3d30dcc455b5382be2fdbfb # v6.3.0
33:        uses: mikepenz/action-junit-report@a9170d5795813c01ab4901ffb045b52bab4ab09d # v6.5.0
42:        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
52:      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
53:      - uses: actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0
73:        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1

$ grep -c 'uses:' .github/workflows/gradle.yml
8
```

Exit code: **0** — 8 `uses:` lines, 8 SHA-pinned matches.

### Local parity: `just docker up`

```
$ just docker up
...
up at http://localhost:18080
```

Exit code: **0**

### Local parity: `just docker spec`

```
$ just docker spec
...
18 failed request(s) in newman summary (see newman output)
error: recipe `spec` failed with exit code 1
```

Exit code: **1** — newman exits non-zero when expected failures exist; CI uses `|| true` on the collection step so the comparator remains the gate.

Newman stats from `spec-api/newman-report.json`:

```
requests: {'total': 31, 'pending': 0, 'failed': 0}
assertions: {'total': 112, 'pending': 0, 'failed': 16}
testScripts: {'total': 46, 'pending': 0, 'failed': 2}
executions: 31
```

### Comparator (local gate, mirrors CI compare step)

```
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt
18 failed, 18 expected
```

Exit code: **0**

### `just test all`

```
$ just test all
...
BUILD SUCCESSFUL in 2s
5 actionable tasks: 1 executed, 4 up-to-date
```

Exit code: **0**

### `just test census`

```
$ just test census
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

Exit code: **0**

### JUnit XML totals (`build/test-results/test/*.xml`)

```
tests=110 failures=0 errors=0 skipped=16
```

## Done When

### Requirements

- [x] **pass** — Job `needs: test`, builds image, starts with generated `JWT_SECRET`, waits on `/tags`, runs `spec-api/run-api-tests.sh` with `APIURL=http://localhost:8080`, gates on `compare_results.py` (D011). Evidence: `.github/workflows/gradle.yml` lines 47–76.
- [x] **pass** — Every new action SHA-pinned with tag comment (`setup-node` v7.0.0 `820762786026740c76f36085b0efc47a31fe5020`; checkout and upload-artifact SHAs reused from `test` job). Evidence: grep output above (8/8).
- [x] **pass** — Newman report uploaded with `if: always()`. Evidence: `.github/workflows/gradle.yml` lines 71–76.
- [ ] **blocked** — On a real PR run, deliberately broken manifest turns job red; evidence in `results/03_spec_job.md`. Skipped per orchestrator scope: step 3 requires push/PR/`gh`, blocked until an earlier slice's PR merges.

### Done When (task checklist)

- [ ] **partial** — All requirements met locally; PR red/green probe not run (blocked).
- [ ] **blocked** — On the PR, `RealWorld spec tests` job green with real manifest and red with one manifest line removed, with run URLs in `results/03_spec_job.md`. Not executed; no push, PR, or `gh` commands per hard rules.

## Notes

- **Anchor drift:** None. Task YAML matched existing `gradle.yml` structure; checkout/upload-artifact SHAs unchanged from task 02.
- **`just docker spec` vs CI:** Local `just docker spec` exits 1 because newman returns non-zero on expected failures; CI collection step uses `|| true` so only the comparator decides pass/fail. Comparator exits 0 locally with 18 failed / 18 expected, matching baseline from tasks 01–02.
- **PR verification skipped:** Task step 3 (prove job red on real PR) deferred to orchestrator after earlier slice PR merges. No `results/03_spec_job.md` created.
- **setup-node SHA:** Lightweight tag (type `commit`); no annotated-tag dereference needed.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `ca49d150-2a3c-4eab-9ce2-716f6cbe7da2`, 09:49:37Z to 09:52:07Z).

**Every action pin re-resolved independently, including the new one:**

```text
actions/checkout v7.0.1              MATCH
actions/setup-java v6.0.1            MATCH
actions/setup-node v7.0.0            MATCH   <- added by this task
actions/upload-artifact v7.0.1       MATCH
gradle/actions/setup-gradle v6.3.0   MATCH
mikepenz/action-junit-report v6.5.0  MATCH
pinned=8 uses_total=8
```

Eight `uses:` lines, eight SHA pins with release tags in trailing comments. The spec job reuses the checkout and
upload-artifact SHAs already in the file rather than introducing second copies at different versions.

**The workflow parses, and the job is shaped as D011 specifies:**

```text
parsed OK; jobs = ['test', 'spec']
spec.needs = test
spec steps = ['actions/checkout', 'actions/setup-node', 'Build image', 'Start app',
              'Run collection', 'Compare against expected failures', 'Upload newman report']
step Upload newman report if: always()
```

- **`needs: test`** keeps the spec job from running when the unit suite is already red.
- **A generated `JWT_SECRET`** (`openssl rand -hex 32`) per run, so no signing key lives in the repository.
- **Readiness probe on `/tags`**, the same endpoint the compose healthcheck uses, polled up to 60 times with
  `docker logs app` dumped before failing — so a startup failure is diagnosable from the job log alone.
- **`|| true` on the collection step** is essential and correct: newman exits non-zero whenever any request fails, and
  18 failures are expected. Without it the job would fail before the comparator ever ran, and the gate would be
  "did anything fail" rather than "did the expected set change".
- **The comparator step is the gate**, and the report uploads with `if: always()` so a red run still yields the
  evidence needed to diagnose it.

**Local parity confirmed** against the same baseline as tasks 01 and 02: 31 requests, 0 transport failures, 112
assertions with 16 failing, 46 test scripts with 2 failing, and `18 failed, 18 expected` from the comparator.

### Deferred, and recorded rather than skipped quietly

The task's step 3 asks for proof on a real PR run: push a commit that deletes one manifest line, confirm the
`RealWorld spec tests` job turns red with that request listed under UNEXPECTED FAILURES, revert, confirm it goes green,
and record both run URLs. That requires a pushable branch, and `feat/spec-tests` is stacked five deep behind PR #4, so
pushing now would open a PR containing every slice's commits.

This is the same red-path evidence pattern slice 1 used successfully (`01_ci_pipeline/results/03_red_probe.md`), and it
is worth doing properly once this branch can be pushed alone. It is listed in this slice's commit gate as outstanding,
so it cannot be lost: **prove the spec job red with a broken manifest, then green with it restored, on the slice's own
PR.**
