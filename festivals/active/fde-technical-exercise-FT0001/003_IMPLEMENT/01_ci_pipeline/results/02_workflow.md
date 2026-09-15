# Task 02: Replace workflow with JDK matrix — evidence

## Action versions and SHAs

Latest release check (`gh api repos/<owner>/<repo>/releases/latest --jq .tag_name`):

| Action | Latest release | Chosen tag | Resolved SHA |
|--------|---------------|------------|--------------|
| actions/checkout | v7.0.1 | v7.0.1 | `3d3c42e5aac5ba805825da76410c181273ba90b1` |
| actions/setup-java | v6.0.1 | v6.0.1 | `de7274f081f381c8f8158605e0321c36c376e2e6` |
| gradle/actions | v6.3.0 | v6.3.0 | `9c971963bec38e04b3d30dcc455b5382be2fdbfb` |
| mikepenz/action-junit-report | v6.5.0 | v6.5.0 | `a9170d5795813c01ab4901ffb045b52bab4ab09d` |
| actions/upload-artifact | v7.0.1 | v7.0.1 | `043fb46d1a93c77aae656e7c1c64a875d1fc6a0a` |

No newer patch releases existed at execution time; all tags match D011.

### Raw `gh api` output

**actions/checkout v7.0.1**

```
$ gh api repos/actions/checkout/releases/latest --jq .tag_name
v7.0.1

$ gh api repos/actions/checkout/git/ref/tags/v7.0.1 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"3d3c42e5aac5ba805825da76410c181273ba90b1","type":"commit"}
```

**actions/setup-java v6.0.1**

```
$ gh api repos/actions/setup-java/releases/latest --jq .tag_name
v6.0.1

$ gh api repos/actions/setup-java/git/ref/tags/v6.0.1 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"de7274f081f381c8f8158605e0321c36c376e2e6","type":"commit"}
```

**gradle/actions v6.3.0** (annotated tag — dereferenced)

```
$ gh api repos/gradle/actions/releases/latest --jq .tag_name
v6.3.0

$ gh api repos/gradle/actions/git/ref/tags/v6.3.0 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"67621b124fd2e251c5e8a0e6e3b91318f2287669","type":"tag"}

$ gh api repos/gradle/actions/git/tags/67621b124fd2e251c5e8a0e6e3b91318f2287669 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"9c971963bec38e04b3d30dcc455b5382be2fdbfb","type":"commit"}
```

**mikepenz/action-junit-report v6.5.0**

```
$ gh api repos/mikepenz/action-junit-report/releases/latest --jq .tag_name
v6.5.0

$ gh api repos/mikepenz/action-junit-report/git/ref/tags/v6.5.0 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"a9170d5795813c01ab4901ffb045b52bab4ab09d","type":"commit"}
```

**actions/upload-artifact v7.0.1**

```
$ gh api repos/actions/upload-artifact/releases/latest --jq .tag_name
v7.0.1

$ gh api repos/actions/upload-artifact/git/ref/tags/v7.0.1 --jq '{sha: .object.sha, type: .object.type}'
{"sha":"043fb46d1a93c77aae656e7c1c64a875d1fc6a0a","type":"commit"}
```

## gradlew mode

```
$ git ls-files -s gradlew
100755 23d15a9367071145e9c79bb4ddf879d1fbe78b5d 0	gradlew
```

Mode is `100755` (executable). No `chmod +x` step added.

## Pin check

```
$ grep -nE 'uses: [^@ ]+@[0-9a-f]{40} # v' .github/workflows/gradle.yml
23:      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
24:      - uses: actions/setup-java@de7274f081f381c8f8158605e0321c36c376e2e6 # v6.0.1
28:      - uses: gradle/actions/setup-gradle@9c971963bec38e04b3d30dcc455b5382be2fdbfb # v6.3.0
33:        uses: mikepenz/action-junit-report@a9170d5795813c01ab4901ffb045b52bab4ab09d # v6.5.0
42:        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1

$ grep -c 'uses:' .github/workflows/gradle.yml
5
```

5 `uses:` lines, 5 grep matches — all actions pinned by SHA with trailing tag comment.

## Local matrix build

Exit code: **0**

Last 40 lines of `just build matrix`:

```
You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 12s
10 actionable tasks: 7 executed, 3 from cache
--- building on JDK 21 ---
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :clean
> Task :checkKotlinGradlePluginConfigurationErrors
> Task :compileKotlin FROM-CACHE
> Task :compileJava NO-SOURCE
> Task :processResources
> Task :classes
> Task :jar
> Task :startScripts
> Task :distTar
> Task :distZip
> Task :assemble
> Task :compileTestKotlin FROM-CACHE
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test FROM-CACHE
> Task :check UP-TO-DATE
> Task :build

[Incubating] Problems report is available at: file:///app/build/reports/problems/problems-report.html

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.14.5/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 9s
10 actionable tasks: 7 executed, 3 from cache
matrix OK: JDK 17 and 21
```

## Orchestrator verification (2026-09-15)

This task was executed by a cursor-agent subagent (`composer-2.5`, headless, session
`34c0314b-a8a6-4fb4-8a18-d7d7d323f5ed`, 20:47:33Z to 20:49:12Z). The orchestrator checked its work instead of taking
the report on trust:

- **Diff.** `git diff .github/workflows/gradle.yml` matches the task's YAML line for line. The `<<EXPR x>>`
  stand-ins became real expressions, and `grep '<<\|<SHA'` finds no leftovers.
- **SHAs.** Each pinned SHA was re-resolved independently from its tag comment (dereferencing annotated tags).
  All five matched, and every tag equals the action's `releases/latest`:
  ```text
  actions/checkout v7.0.1 pinned=3d3c42e5aac5ba805825da76410c181273ba90b1 resolved=3d3c42e5aac5ba805825da76410c181273ba90b1 MATCH latest_release=v7.0.1
  actions/setup-java v6.0.1 pinned=de7274f081f381c8f8158605e0321c36c376e2e6 resolved=de7274f081f381c8f8158605e0321c36c376e2e6 MATCH latest_release=v6.0.1
  gradle/actions/setup-gradle v6.3.0 pinned=9c971963bec38e04b3d30dcc455b5382be2fdbfb resolved=9c971963bec38e04b3d30dcc455b5382be2fdbfb MATCH latest_release=v6.3.0
  mikepenz/action-junit-report v6.5.0 pinned=a9170d5795813c01ab4901ffb045b52bab4ab09d resolved=a9170d5795813c01ab4901ffb045b52bab4ab09d MATCH latest_release=v6.5.0
  actions/upload-artifact v7.0.1 pinned=043fb46d1a93c77aae656e7c1c64a875d1fc6a0a resolved=043fb46d1a93c77aae656e7c1c64a875d1fc6a0a MATCH latest_release=v7.0.1
  ```
- **Build caveat.** In the JDK 21 tail above, `:compileKotlin`, `:compileTestKotlin` and `:test` are `FROM-CACHE`.
  The build cache restored earlier outputs with identical inputs, so `matrix OK` here does not show the tests
  re-executing on JDK 21. The real proof is the CI run on both JDKs (task 03 and the slice PR). The sequence's
  testing gate re-runs the suite.
- **Scope.** The subagent ran no git, push, PR or fest commands; the orchestrator made the commit.

## Notes

Nothing went wrong. All five action tags matched their latest releases; `gradle/actions` v6.3.0 required dereferencing an annotated tag object. `gradlew` was already executable in git. Matrix build passed on both JDK 17 and 21.
