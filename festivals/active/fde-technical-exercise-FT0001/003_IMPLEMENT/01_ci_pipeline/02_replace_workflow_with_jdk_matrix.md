---
fest_type: task
fest_id: 02_replace_workflow_with_jdk_matrix.md
fest_name: replace_workflow_with_jdk_matrix
fest_parent: 01_ci_pipeline
fest_order: 2
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:01:26.464587-06:00
fest_updated: 2026-09-15T14:51:21.630023-06:00
fest_tracking: true
---


# Task: Replace the workflow with a pinned JDK 17/21 matrix

## Objective

Replace `.github/workflows/gradle.yml` with a JDK 17/21 matrix that caches Gradle, pins every action by commit SHA, and reports test failures as annotations.

## Requirements

- [ ] Implements D011's `test` job exactly: triggers on push and pull_request to `master` plus `workflow_dispatch`; permissions `contents: read` and `checks: write`; matrix `java` 17 and 21 with `fail-fast: false`.
- [ ] Every third-party action is pinned by its full 40-character commit SHA, with the release tag in a trailing comment
- [ ] The local build is green on both JDKs before any push (`just build matrix`)
- [ ] Work happens on branch `ci/jdk-matrix`, and the workflow is committed there with `fest commit` before task 03 branches from it (D012)

## Implementation

**File:** `.github/workflows/gradle.yml`, replaced wholesale. The lines being retired are `:25` `actions/checkout@v3`, `:29` `java-version: '16'`, `:32` `gradle/gradle-build-action@v2` and `:35` `./gradlew build`. Keeping the same path keeps the workflow's identity on GitHub.

**Steps**

1. **Branch** from an up-to-date fork `master`:
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c ci/jdk-matrix
   ```
2. **Resolve a commit SHA** for each action tag. An annotated tag points at a tag object, so dereference it when `object.type` is `tag`:
   ```bash
   sha() {  # usage: sha owner/repo tag
     read -r obj typ < <(gh api "repos/$1/git/ref/tags/$2" --jq '.object.sha + " " + .object.type')
     if [ "$typ" = "tag" ]; then gh api "repos/$1/git/tags/$obj" --jq .object.sha; else echo "$obj"; fi
   }
   sha actions/checkout v7.0.1
   sha actions/setup-java v6.0.1
   sha gradle/actions v6.3.0
   sha mikepenz/action-junit-report v6.5.0
   sha actions/upload-artifact v7.0.1
   ```
   These versions were current on 2026-09-15. If a newer patch release exists when you run this, you may use it; record the version and SHA you chose in `results/02_workflow.md`.
3. **Write the workflow.** Two stand-ins need replacing:
   - `<<EXPR x>>` stands for a GitHub Actions expression wrapping `x`: a dollar sign, two opening braces, `x`, two closing braces. It is written this way because the festival's marker scanner treats literal double braces as unfilled template placeholders.
   - `<SHA:…>` stands for the SHA from step 2.

   ```yaml
   name: CI

   on:
     push:
       branches: [ "master" ]
     pull_request:
       branches: [ "master" ]
     workflow_dispatch:

   permissions:
     contents: read
     checks: write

   jobs:
     test:
       name: build and test (JDK <<EXPR matrix.java>>)
       runs-on: ubuntu-latest
       strategy:
         fail-fast: false
         matrix:
           java: [ '17', '21' ]
       steps:
         - uses: actions/checkout@<SHA:checkout> # v7.0.1
         - uses: actions/setup-java@<SHA:setup-java> # v6.0.1
           with:
             distribution: temurin
             java-version: <<EXPR matrix.java>>
         - uses: gradle/actions/setup-gradle@<SHA:gradle-actions> # v6.3.0
         - name: Build and test
           run: ./gradlew build
         - name: Publish test results
           if: always()
           uses: mikepenz/action-junit-report@<SHA:junit-report> # v6.5.0
           with:
             report_paths: build/test-results/test/*.xml
             check_name: JUnit (JDK <<EXPR matrix.java>>)
             include_skipped: true
             detailed_summary: true
             require_tests: true
         - name: Upload test reports
           if: failure()
           uses: actions/upload-artifact@<SHA:upload-artifact> # v7.0.1
           with:
             name: test-reports-jdk<<EXPR matrix.java>>
             path: build/reports/tests
   ```
   These settings were checked against each action's own `action.yml` at those tags on 2026-09-15:
   - setup-gradle's `validate-wrappers` defaults to `true` (`setup-gradle/action.yml:200-203`), so the Gradle wrapper jar is validated with no extra step.
   - setup-gradle's `cache-read-only` defaults to writing the cache only on the default branch (lines 25-28), so leave it unset.
   - action-junit-report v6.5.0 declares `report_paths`, `check_name`, `include_skipped`, `detailed_summary` and `require_tests` (`action.yml` lines 14, 41, 68, 101, 56).
4. **Check `./gradlew` is executable.** `git ls-files -s gradlew` must show mode `100755`. If it does not, add a `chmod +x ./gradlew` step before the build instead of changing the file mode in git.
5. **Build locally, in Docker.** Run `just build matrix` (`.justfiles/build.just:47-53`); it must end with `matrix OK: JDK 17 and 21`. Save the tail of the output to `results/02_workflow.md`.
6. **Commit** on `ci/jdk-matrix` with `fest commit -m "ci: JDK 17/21 matrix with pinned actions and JUnit annotations"`, so task 03 can branch from a committed state. GitHub validates workflow syntax when the first run is created. An invalid file shows up as a failed run marked as an invalid workflow file; fix it on this branch.

**Error paths**

- `sha` prints nothing, or the API returns 404: the tag name is wrong. List real tags with `gh api repos/<owner>/<repo>/tags --jq '.[].name' | head` and choose the intended release.
- `just build matrix` fails on JDK 21 only: reproduce it with `just build jdk 21` and fix the build before touching the workflow. The workflow must not paper over a real JDK 21 failure.
- The JUnit step later reports "No test results found": `require_tests: true` is doing its job. Confirm `./gradlew build` writes `build/test-results/test/*.xml` before changing the path.

## Done When

- [ ] All requirements met
- [ ] `just build matrix` ends with `matrix OK: JDK 17 and 21`
- [ ] `grep -nE 'uses: [^@ ]+@[0-9a-f]{40} # v' .github/workflows/gradle.yml` matches every `uses:` line in the file, so there are no tag-only pins
- [ ] The workflow is committed on `ci/jdk-matrix`