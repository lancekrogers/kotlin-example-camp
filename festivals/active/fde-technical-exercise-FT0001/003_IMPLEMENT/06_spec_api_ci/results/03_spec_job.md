# Evidence: the spec job's red path, proven on real runs

Requirement: "On a real PR run, a deliberately broken manifest turns the job red; evidence is recorded in
`results/03_spec_job.md`, then the manifest is restored."

Until 2026-09-16 this was the one claim in slice 6 proven only locally. It is now proven on GitHub Actions.

## Why the runs are `workflow_dispatch`, not `pull_request`

The workflow filters on the **base** branch:

```yaml
on:
  pull_request:
    branches: [ "master" ]
```

The slice branches are stacked (D012), so their PRs target the preceding feature branch, not `master`, and no
`pull_request` run is triggered at all. `gh pr checks` on #5-#10 lists zero checks for that reason. The workflow also
declares `workflow_dispatch`, so each branch was dispatched directly; the runs execute the same jobs against the same
commits.

## Green: the real manifest

Run <https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35127041807> (`feat/spec-tests`)

```text
build and test (JDK 21): success
build and test (JDK 17): success
JUnit (JDK 17): success
JUnit (JDK 21): success
RealWorld spec tests: success
```

## Red: one manifest entry removed

Branch `ci/prove-spec-red`, PR #10, one commit deleting exactly one line from `spec-api/expected-failures.txt`:

```text
dropped: Articles / All Articles
 spec-api/expected-failures.txt | 1 -
 1 file changed, 1 deletion(-)
```

Run <https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35127046886>

```text
build and test (JDK 17): success
build and test (JDK 21): success
JUnit (JDK 17): success
JUnit (JDK 21): success
RealWorld spec tests: failure
```

Failing step, `Compare against expected failures`:

```text
UNEXPECTED FAILURES (regressions)
  Articles / All Articles
18 failed, 17 expected
##[error]Process completed with exit code 1.
```

## What this establishes

- The job goes red on a stale manifest and green on the correct one, so it is a real gate rather than a step that
  always passes.
- The failure is **discriminating**, not incidental: both JDK build jobs stayed green across the two runs, and only
  `RealWorld spec tests` flipped. The failure surfaced at the comparator step, naming the exact request whose entry was
  removed.
- `18 failed, 17 expected` confirms the collection still ran in full; the same 18 requests failed, and the manifest
  simply stopped covering one of them. The red path is not the run breaking, it is the gate noticing.
- `|| true` is on the collection step only. newman exited non-zero in both runs; the comparator's exit code decided the
  job both times.

## Restore

The manifest is untouched on `feat/spec-tests`; the deletion exists only on the throwaway `ci/prove-spec-red`, which is
closed unmerged and its branch deleted. Nothing needed reverting on the slice branch.
