# Task 03 results: CI red-path probe

## Local failure

**Command:** `just test only CiRedProbeTest`

**Exit code:** 1

**Output:**

```
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin FROM-CACHE
> Task :compileJava NO-SOURCE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :compileTestKotlin
> Task :compileTestJava NO-SOURCE
> Task :testClasses UP-TO-DATE

> Task :test FAILED

CiRedProbeTest > ci red-path probe - must fail FAILED
    java.lang.AssertionError: CI must report this failure as an annotation expected:<1> but was:<2>
        at org.junit.Assert.fail(Assert.java:89)
        at org.junit.Assert.failNotEquals(Assert.java:835)
        at org.junit.Assert.assertEquals(Assert.java:647)
        at io.realworld.app.CiRedProbeTest.ci red-path probe - must fail(CiRedProbeTest.kt:9)

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.
5 actionable tasks: 3 executed, 1 from cache, 1 up-to-date

1 test completed, 1 failed

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':test'.
> There were failing tests. See the report at: file:///app/build/reports/tests/test/index.html

* Try:
> Run with --scan to get full insights.

BUILD FAILED in 6s
error: recipe `gradle` failed on line 16 with exit code 1
error: recipe `only` failed on line 20 with exit code 1
```

### Orchestrator verification

A cursor-agent subagent ran steps 2 and 3 (`composer-2.5`, session `1140b1a4-b584-4b9d-9b52-47ae4eb87d5e`,
20:51:52Z to 20:52:23Z). The orchestrator then checked the result:

- **Branch state.** `git status --untracked-files=all` on `ci/prove-red` shows exactly one change,
  `?? src/test/kotlin/io/realworld/app/CiRedProbeTest.kt`. Its content matches the task document line for line.
- **Independent re-run.** `just test only CiRedProbeTest` exited 1 with the same failure:
  ```text
  > Task :test FAILED
  CiRedProbeTest > ci red-path probe - must fail FAILED
      java.lang.AssertionError: CI must report this failure as an annotation expected:<1> but was:<2>
          at io.realworld.app.CiRedProbeTest.ci red-path probe - must fail(CiRedProbeTest.kt:9)
  BUILD FAILED in 3s
  ```
- **Commit scope.** The probe commit uses `fest commit --no-root`, so the camp's submodule pointer never
  records a commit on a branch that will be deleted.

## GitHub run

Probe commit `0dfea6e` (`fest commit --no-root`) was pushed to `origin/ci/prove-red`. It opened
https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/pull/2 at 2026-09-15T20:53:18Z against the fork's
`master`. Run https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856
(`pull_request`) was created at 20:53:21Z.

**Result: the pipeline turns red, with a readable per-test annotation, on both JDKs.**

```text
$ gh pr checks 2 --repo $R
JUnit (JDK 17)	fail	0	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/runs/104561297538
JUnit (JDK 21)	fail	0	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/runs/104561319791
build and test (JDK 17)	fail	1m8s	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856/job/104560931611
build and test (JDK 21)	fail	1m12s	https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856/job/104560931839
$ gh run view 35022352856 --repo $R --json headSha,event,conclusion,url,jobs --jq '{headSha, event, conclusion, url, jobs: [.jobs[] | {name, conclusion}]}'
{"conclusion":"failure","event":"pull_request","headSha":"0dfea6e3fd5a221f31f3f7a9f949aa0f99df962f","jobs":[{"conclusion":"failure","name":"build and test (JDK 17)"},{"conclusion":"failure","name":"build and test (JDK 21)"},{"conclusion":"failure","name":"JUnit (JDK 17)"},{"conclusion":"failure","name":"JUnit (JDK 21)"}],"url":"https://github.com/lancekrogers/kotlin-ktor-realworld-example-app/actions/runs/35022352856"}
$ gh api repos/$R/commits/$head/check-runs --jq '.check_runs[] | "\(.id)\t\(.name)\t\(.conclusion)\t\(.output.title)"'
104561319791	JUnit (JDK 21)	failure	26 tests run, 4 passed, 21 skipped, 1 failed.
104561297538	JUnit (JDK 17)	failure	26 tests run, 4 passed, 21 skipped, 1 failed.
104560931839	build and test (JDK 21)	failure	null
104560931611	build and test (JDK 17)	failure	null
$ gh api repos/$R/check-runs/104561319791/annotations --jq '.[] | "\(.path):\(.start_line) \(.annotation_level) \(.title) | \(.message | split("\n")[0])"'
src/test/kotlin/io/realworld/app/CiRedProbeTest.kt:9 failure CiRedProbeTest.ci red-path probe - must fail | java.lang.AssertionError: CI must report this failure as an annotation expected:<1> but was:<2>
$ gh api repos/$R/check-runs/104561297538/annotations --jq (same filter)
src/test/kotlin/io/realworld/app/CiRedProbeTest.kt:9 failure CiRedProbeTest.ci red-path probe - must fail | java.lang.AssertionError: CI must report this failure as an annotation expected:<1> but was:<2>
```

What this proves, against the task's Done When and D011:

- **Both matrix jobs run and fail independently.** `fail-fast: false` works: JDK 17 did not cancel JDK 21.
- **The failure is readable without opening logs.** Each JUnit check run names the test, file and line, and quotes the
  assertion message.
- **The report counts skipped tests.** 21 of 26 are skipped (`include_skipped: true`), which matches the class-level
  `@Ignore` baseline.
- **Runs work on the fork.** This is also the confirmation for task 01.

The job summary (`output.summary`) came back empty through the check-runs API. The annotations and the title carry
the evidence.

## Cleanup

The PR was closed without merging, and the probe branch was deleted on the fork and locally. `gh pr close` ran
from outside the repository, so it could delete only the remote branch; it could not switch the checkout the
testing gate was using. The local branch was then deleted explicitly.

```text
$ gh pr close 2 --repo $R --delete-branch
✓ Closed pull request lancekrogers/kotlin-ktor-realworld-example-app#2 (CI red-path probe (do not merge))
✓ Deleted branch ci/prove-red
$ git branch -D ci/prove-red   (current branch: ci/jdk-matrix)
Deleted branch ci/prove-red (was 0dfea6e).
$ gh pr view 2 --repo $R --json state,closedAt,mergedAt --jq '{state, closedAt, mergedAt}'
{"closedAt":"2026-09-15T20:57:15Z","mergedAt":null,"state":"CLOSED"}
$ git ls-remote --heads origin
bf1435e0a20c2f0cc79836fabcd8dae1408e8222	refs/heads/master
588dbce14584193104b137e3c7b03f90faccb13e	refs/heads/security/audit-remediation
$ git branch --list
* ci/jdk-matrix
  master
$ git cat-file -e ci/jdk-matrix:src/test/kotlin/io/realworld/app/CiRedProbeTest.kt
CiRedProbeTest.kt absent on ci/jdk-matrix
```

The probe commit never entered the camp. It was committed with `--no-root`, so the camp's submodule pointer
still records the task 02 commit:

```text
$ git -C <camp> ls-tree HEAD projects/kotlin-ktor-realworld-example-app
160000 commit 9752feae0721c1193fcdf57eee3c79e19ad58026	projects/kotlin-ktor-realworld-example-app
```
