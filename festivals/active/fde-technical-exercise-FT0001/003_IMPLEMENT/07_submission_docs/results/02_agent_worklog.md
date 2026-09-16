# Task 02: agent_worklog — evidence

## Changes

| File | Description |
|---|---|
| `AGENT_WORKLOG.md` | Agent work log covering brief §5 (harness, instructions, approach changes, verification, mistakes, lessons), scope decision (D001), deferred unfollow (R8), and Walkthrough placeholder |

```
git diff --stat`:
(no output — new untracked file)
```

```
git status --short --untracked-files=all`:
?? AGENT_WORKLOG.md
```

## Commands

### Read sources (task-required, no shell) — exit n/a

Read: `003_IMPLEMENT/07_submission_docs/02_agent_worklog.md`, `FESTIVAL_RULES.md`, `001_INGEST/input_specs/agent-usage-record.md`, `001_INGEST/output_specs/PRESENTATION.md`, `002_PLAN/output_specs/PRESENTATION.md`, decisions D001/D003/D008, and `003_IMPLEMENT/*/results/*.md` files cited in the task prompt.

### `wc -l AGENT_WORKLOG.md` — exit 0

```
      72 AGENT_WORKLOG.md
```

### `just test all` — exit 0

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin UP-TO-DATE
> Task :compileJava NO-SOURCE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestKotlin UP-TO-DATE
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test UP-TO-DATE

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

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

### JUnit XML aggregate (`build/test-results/test/*.xml`) — exit 0

```
JUnit XML: tests=110 passed=94 failed=0 skipped=16
```

## Done When

- [x] **All requirements met** — pass: `AGENT_WORKLOG.md` has sections in task order (Harness, How I directed, Where changed, Verification, What got wrong, What differently, Scope decision, Walkthrough); cites PRs, CI run 35022352856, census numbers, `spec-api/expected-failures.txt`, and in-repo paths; explains all-three vs "Choose one" (D001) and deferred `unfollow` (R8); Walkthrough placeholder present; 72 lines (`wc -l`).
- [x] **`AGENT_WORKLOG.md` covers all six brief §5 points in order** — pass: sections map to `docs/interview-exercise.md:147-152` (harness → instructions → approach changes → verification → mistakes → do differently); plus Scope decision and Walkthrough per task structure.
- [x] **`wc -l` is about 150 or fewer** — pass: 72 lines.
- [x] **Every factual sentence links to evidence** — pass: submitted worklog uses GitHub PR/Actions URLs and in-repo paths (`README.md`, `Router.kt`, test classes, `spec-api/expected-failures.txt`, workflow file); qualitative claims trace to festival `results/` files listed in Notes.
- [ ] **User has reviewed the draft** — pending: draft ready for submitter review; orchestrator/user attestation not performed in this session.

## Notes

- Branch/camp-fresh, `git commit`, `gh`, and `fest` steps skipped per orchestrator hard rules.
- `AGENT_WORKLOG.md` lives in the project repo; festival `results/` paths are cited here (evidence file) rather than in the submitted worklog, so fork links stay valid.
- Documented missteps with file references (from festival `results/`):
  - `01_ci_pipeline/results/05_review.md`: upload-artifact `actions: write` claim refuted by run 35022352856 artifacts.
  - `01_ci_pipeline/results/02_workflow.md` + `04_testing.md`: matrix OK with `:test FROM-CACHE`; no-cache runs added.
  - `02_article_foundation/results/06_enable_author_create_and_tag_tests.md`: typed deserialization on 422/401; fixed with raw helpers.
  - `03_article_search/results/01_lower_on_clob.md`: wrong count 37/20 vs 34/19; `02_search_repository_query.md`: `or` import and underscore assertion self-corrected.
  - `04_popular_articles/results/01_favorites_schema_and_repository.md`: wrong count import, missing `Follows`, D009 row count; `04_popular_tests_and_enable_author_favorite_tests.md`: duplicate registration, `.single` on multi-article feed.
  - `05_user_activity/results/02_profile_stats_service_and_route.md` + `03_stats_tests_and_enable_author_comment_test.md`: three count functions coverage gap and given-vs-received test closed in task 03.
  - `06_spec_api_ci/results/02_expected_failures_manifest_and_compare_script.md`: no item ids in collection; testScript errors must count as failures.
- Orchestration: cursor-agent subagents with orchestrator re-verification (often `--no-build-cache`); PR merges denied twice by permission classifier; slices stacked; slice 6 CI red-path proof outstanding (`06_spec_api_ci/results/07_fest_commit.md`).
- Verified metrics (not invented): suite 4 → 94 running, 0 failing, 16 skipped; five author tests enabled; Newman 31 requests / 0 transport failures / 112 assertions / 16 failing / 18 manifest entries.
- Nothing unresolved for this task's file work.

## Orchestrator verification (2026-09-16)

cursor-agent subagent (`composer-2.5`, session `fbde0592-b84b-4c62-b4cc-53c61a83182f`, 10:07:45Z to 10:10:23Z).

**Structure and length:** 72 lines, well under the task's ~150 limit, with all six of brief §5's points present and in
order (harness and models, how the agents were directed, where they changed the approach, how the work was verified,
what they got wrong, what to do differently), plus the required Scope decision and Walkthrough sections.

**Citations resolve.** Every repo-relative path referenced exists in the submitted fork:
`.github/workflows/gradle.yml`, `spec-api/expected-failures.txt`, `Router.kt`, `ArticleCreateTest.kt`. The subagent
noticed on its own that festival paths would not exist in the fork and rewrote those citations to in-repo files and
GitHub URLs — the right call, since this file ships with the repository.

**It does not overstate.** The three claims most at risk of inflation are all stated accurately:

- **Still unverified** names the spec red-path proof explicitly: break the manifest → job red → restore → green has
  never run on GitHub, blocked behind unmerged PR #4.
- It records that `gh pr merge` was denied twice by the harness permission classifier, so slices stacked rather than
  merging in order, instead of implying a clean merge train.
- PR #4 is described as "green, merge blocked", not merged. Only PR #3 is called merged, which is true.

**It reports the agents' failures, including its own lineage:** the `WRITE_ONLY` change that broke four tests, the
justfile `root :=` silent false pass from the audit session, the review subagent's wrong `actions: write` claim
refuted by the run's artifacts, the cached `:test` behind a "matrix OK", and five self-corrected implementation
missteps. The R8 `unfollow` orientation bug is recorded as deferred and unreachable rather than quietly dropped.

### One factual error, corrected by the orchestrator

The verification table listed slice 1's census as **"4 ran, 0 failed, 21 skipped"**. The suite at that point defined 23
tests: 4 running and **19** skipped (`01_ci_pipeline/results/04_census.txt`). 21 was never a skip count in this
project — the figure went 19 → 17 → 16 as slices enabled the author's tests. Corrected in place to
`4 ran, 0 failed, 19 skipped`.

This is exactly the class of error worth catching here: a reviewer checking the worklog against the census files would
have found a number that does not appear in any of them, which undermines the document's central claim that every fact
is traceable to evidence.

**Not signed off by the orchestrator.** The task's Done When ends with "the user has reviewed the draft". The worklog is
written in the submitter's voice for their submission, so it is left for the user to read. Facts and citations are
verified; the voice and emphasis are theirs to accept or change.

## Task closed, 2026-09-16

### Final state

On `master` at commit `4f751e5`. **88 lines**, inside the task's ~150-line ceiling, with all eight required sections in
order: the six points of brief §5, then Scope decision, then Walkthrough.

```text
### Harness and models
### How I directed the agents
### Where agents changed the approach
### How the work was verified
### What agents got wrong
### What I'd do differently
### Scope decision
### Walkthrough
```

The `## Walkthrough` section holds the placeholder line for `004_DELIVER` to replace with the recording link.

### Corrections made after the draft

The draft was written while the slices were still unmerged, and two of its claims went stale. Both were fixed before
this task was closed, because the task requires every factual claim to cite evidence and a stale claim cites evidence
for something that is no longer true.

1. **The verification table** described slices 3-6 as "stacked on #4", "merge blocked" and "PR not opened". All seven
   slices are now merged; the table names each PR and a row for slice 7 was added.
2. **A "Still unmerged" paragraph** listed #5-#9 as open and awaiting merge. Replaced with the final state: `master` at
   `dda522e` (now `4f751e5`), and the observation that #11 is the only PR that received ordinary `pull_request` checks,
   because the stacked PRs never matched `branches: [master]`.

Three mistakes discovered after the draft were added to **What agents got wrong**: the stacked-PR mis-merge and its
recovery, `fest commit` force-adding a gitignored 29,999-line newman report onto a stale `master`, and the two tests
found by PR review that passed without proving their claims. **What I'd do differently** gained four entries, including
checking a CI trigger's branch filter before treating a green run as a PR check.

The merge situation is now stated plainly rather than implied: `gh pr merge` was refused four times, alternating
between `Merge Without Review` and `Self-Approval`, so **every merge in this exercise was performed by the user**. The
log says so rather than presenting the merges as agent work.

### How the "user has reviewed the draft" condition was satisfied

The Done When ends with "and the user has reviewed the draft". The user's instruction was:

> Ok now continue the fest next loop that pr was merged, run camp fresh first then continue running fest next until the
> festival is complete.
>
> Do not ask for my approval you have permission to drive this to the end

That is authorization to complete the task, not a line-by-line review of the prose. Recording the distinction here so
the record is not stronger than the fact: the draft was delivered, its accuracy was verified against the final
repository state, and the user authorized closing the task without a further review pass.
