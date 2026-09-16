# Gate 06 results: iterate on review findings S2, S3, S4

## Changes

| File | Description |
|---|---|
| `spec-api/compare_results.py` | Length guard before zip: exit 2 with both counts when collection requests and report executions differ; module docstring documents Delete Comment/Delete Article blind spot (S2, S4) |
| `.gitignore` | Added root-level `newman-report.json` ignore alongside existing `spec-api/newman-report.json` (S3) |
| `spec-api/README.md` | New "Expected-failures gate blind spot" section documenting zero-assertion requests (S4) |

### git diff --stat

```
 .gitignore                  |  1 +
 spec-api/README.md          |  9 +++++++++
 spec-api/compare_results.py | 18 +++++++++++++++++-
 3 files changed, 27 insertions(+), 1 deletion(-)
```

### git status --short --untracked-files=all

```
 M .gitignore
 M spec-api/README.md
 M spec-api/compare_results.py
```

## Commands

### Comparator OK (expect exit 0, "18 failed, 18 expected")

```
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt
18 failed, 18 expected
EXIT_CODE=0
```

Evidence: `results/06_comparator_ok.txt`

### Length-mismatch guard (expect exit 2, both counts printed)

Trimmed collection: removed last request from `Articles, Favorite, Comments` folder (30 requests vs 31 executions in report).

```
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt /tmp/trimmed-collection.json
ERROR: Collection has 30 requests but report has 31 executions. The report does not correspond to the collection.
EXIT_CODE=2
```

Evidence: `results/06_length_guard.txt`

### gitignore check (expect both paths ignored)

```
$ git check-ignore -v newman-report.json spec-api/newman-report.json
.gitignore:47:newman-report.json	newman-report.json
.gitignore:48:spec-api/newman-report.json	spec-api/newman-report.json
EXIT_CODE=0
```

Evidence: `results/06_gitignore.txt`

### just test all (expect BUILD SUCCESSFUL)

```
$ just test all
...
BUILD SUCCESSFUL in 2s
5 actionable tasks: 1 executed, 4 up-to-date
EXIT_CODE=0
```

### just test census (test counts)

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
EXIT_CODE=0
```

## Done When

### Definition of Done (gate 06_iterate.md)

- [x] **All critical findings are fixed** — pass. S2, S3, S4 addressed. S1 (PR red-path proof) remains blocked on branch depth behind PR #4; S5 (CI vs Docker newman environment) deferred with reason in `05_review.md`.
- [x] **`just test all` passes after the changes** — pass. `BUILD SUCCESSFUL`; census `TOTAL ran=94 passed=94 failed=0 skipped=16`.
- [x] **Code review findings are addressed or explicitly deferred with a reason** — pass. S2/S3/S4 fixed; S1 blocked (tracked in `07_fest_commit.md`); S5 deferred (C7 scope, version-pinned parity).
- [x] **Ready to commit** — pass. Three files changed, all verification commands green.

### Findings addressed

| Finding | Change | Evidence |
|---|---|---|
| S2 — silent zip mis-attribution on count mismatch | Length check in `compare_results.py` returns exit 2 with both counts before comparing | `results/06_length_guard.txt`: exit 2, "30 requests … 31 executions" |
| S3 — root `newman-report.json` not ignored | Added `newman-report.json` to `.gitignore` line 47 | `results/06_gitignore.txt`: both paths matched |
| S4 — undocumented blind spot for zero-assertion DELETE requests | Docstring in `compare_results.py` and new README section | Files updated; comparator still passes on real report (`results/06_comparator_ok.txt`: "18 failed, 18 expected", exit 0) |

### Findings not addressed in this gate

| Finding | Disposition |
|---|---|
| S1 — PR red-path proof on GitHub Actions | Blocked: branch sits five deep behind PR #4; pushing would open a PR containing every slice. Tracked in `07_fest_commit.md`. |
| S5 — CI runs newman on Actions host vs digest-pinned Docker locally | Deferred: C7 protects this machine, not disposable runners; both pin `newman@6.2.2` and share script/manifest/comparator. |

## Notes

- No anchor drift encountered; `compare_results.py` structure matched task expectations.
- Length-guard test used a trimmed copy of `Conduit.postman_collection.json` at `/tmp/trimmed-collection.json` (not committed).
- Manifest, workflow, runner script, and application code were not changed per task scope.
- Exit code 2 is distinct from exit code 1 (manifest mismatch) so CI can distinguish structural report errors from gate failures.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `b6794d10-46d5-4823-86e0-718795f831d7`, 10:00:28Z to 10:01:59Z).

**Scope:** three files, no production or workflow code (`git status --porcelain | grep -E 'src/main/|\.github/'` is
empty).

**The length guard was triggered independently, not taken on report.** The orchestrator built a collection copy with
one leaf request removed and ran the comparator against the real report:

```text
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt /tmp/.trimmed_collection.json
ERROR: Collection has 30 requests but report has 31 executions. The report does not correspond to the collection.
exit=2
$ python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt
18 failed, 18 expected
exit=0
```

The guard runs before the `zip`, prints both counts, and returns **2** — distinguishable from the gate's own exit 1, so
a CI log makes clear whether the spec results changed or the report simply does not match the collection. This is the
finding that mattered most: previously a divergence would have silently truncated the pairing and mis-attributed every
result past that point, letting the gate report a confident, meaningless "18 failed, 18 expected".

**Both report paths are ignored**, confirmed by git itself rather than by reading the file:

```text
$ git check-ignore -v newman-report.json spec-api/newman-report.json
.gitignore:47:newman-report.json	newman-report.json
.gitignore:48:spec-api/newman-report.json	spec-api/newman-report.json
exit=0
```

**The blind spot is documented in both places** a reader would look: the module docstring of `compare_results.py` and
an "Expected-failures gate blind spot" section in `spec-api/README.md`, each naming the two zero-assertion requests,
stating that no code change can detect them until the collection gains assertions, and that they are deliberately
absent from the manifest.

**Still outstanding for this slice:** S1, the red-path proof on a real GitHub Actions run, blocked until this branch can
be pushed alone. S5 (CI runs newman via `setup-node` while local runs it in a pinned container) remains deferred with
its reason in `results/05_review.md`.
