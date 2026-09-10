# Supply-chain and build-trust findings

These answer the question "is it safe to build and run this on my machine?" They concern what the
build **fetches and executes**, which is where the real exposure is — the committed code itself is
clean.

All line references are to `projects/kotlin-ktor-realworld-example-app`.

---

## A1 — `jcenter()` is a decommissioned repository in the resolution chain {#a1}

**Severity:** Medium · **Location:** `build.gradle:20` (buildscript), `build.gradle:34` (project)

JCenter stopped accepting new packages in 2021 and has been wound down. Two problems follow.

First, JCenter never enforced namespace ownership the way Maven Central does — anyone could
publish under any group ID. Any artifact resolved from it carries weaker provenance guarantees
than the same artifact from Central.

Second, and more seriously, a decommissioned service in a resolution chain is a **stale-domain
risk**. If `jcenter.bintray.com` ever lapses or is repointed, every build that lists it inherits
whatever the new owner serves. Because `jcenter()` appears in the **`buildscript` block** as well,
a poisoned artifact there would execute during Gradle's configuration phase — before any of your
code runs, and with your user's full privileges.

**Failure scenario:** the domain lapses and is re-registered; `./gradlew build` resolves the
Kotlin plugin (or any transitive dependency) from the attacker's host and executes it at configure
time. No warning, no prompt.

**Fix:** delete both `jcenter()` lines. Every dependency in this build is available on Maven
Central.

---

## A2 — `mavenLocal()` is consulted before Maven Central {#a2}

**Severity:** Medium · **Location:** `build.gradle:18` (buildscript), `build.gradle:32` (project)

`mavenLocal()` is listed **first** in both repository blocks, so `~/.m2/repository` takes
precedence over Central for every artifact. Any artifact sitting in the local Maven cache — put
there by an unrelated project, a stale install, or another tool — silently shadows the real one.

This is dependency confusion with a local origin. It is worse in the `buildscript` block for the
same reason as A1: those artifacts execute at configuration time.

It also makes builds non-reproducible between machines, which is directly relevant to a CI matrix
that is supposed to prove the code works.

**Fix:** remove both `mavenLocal()` lines. Nothing in this project publishes to or consumes from
the local Maven repository.

---

## A3 — `kotlin_version = "1.3.+"` floats a plugin that executes at build time {#a3}

**Severity:** Medium · **Location:** `build.gradle:6`, consumed at `build.gradle:23`

The wildcard resolves to whatever the newest 1.3.x is at resolution time. That version feeds
`classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"` — a **buildscript
classpath entry**, meaning arbitrary code that Gradle loads and runs during configuration.

A dynamic version on an executing plugin means the code your build runs is not pinned, not
recorded, and not reproducible. Combined with A1 and A2, the set of places that code can come from
is wider than it should be. It also means two builds of the same commit, days apart, can differ.

Every other dependency in the file is pinned to an exact version; this one is the outlier.

**Fix:** pin to an exact version (`1.3.72` is the last 1.3.x release). Note that any real
modernization will move this well past 1.3 anyway — see the Gradle upgrade forced by the JDK 17/21
CI requirement.

---

## A4 — The Gradle distribution download is not checksum-verified {#a4}

**Severity:** Low-Medium · **Location:** `gradle/wrapper/gradle-wrapper.properties:6`

`distributionUrl` uses HTTPS, which is good, but there is no `distributionSha256Sum` line. The
wrapper therefore accepts whatever the URL returns without verifying it.

The wrapper JAR is fully capable of checking — `Install.class` contains `distributionSha256Sum`,
`calculateSha256Sum`, and `verifyDownloadChecksum` — the property is simply not set. TLS is the
only integrity control in play, so a compromised mirror, a corporate TLS-intercepting proxy, or a
tampered cache would go undetected.

**Fix:** add the published `distributionSha256Sum` for the chosen distribution. Best done at the
same time as the Gradle upgrade that JDK 17/21 support requires.

---

## A5 — Wrapper JAR version does not match the requested distribution {#a5}

**Severity:** Low · **Location:** `gradle/wrapper/gradle-wrapper.jar` vs `gradle-wrapper.properties:6`

The JAR's own metadata reports Gradle **4.4** (`META-INF/MANIFEST.MF` →
`Implementation-Version: 4.4`; `build-receipt.properties` → `versionNumber=4.4`), while
`gradle-wrapper.properties` requests **gradle-4.10-all.zip**.

**This is drift, not tampering.** The JAR's contents were verified stock (see the cleared-surfaces
section of the main audit) and a 4.4 wrapper bootstrapping a 4.10 distribution works fine. It
happens when someone edits the properties file without re-running `gradle wrapper`. Worth
recording only because a version mismatch on a wrapper JAR is exactly the shape a real attack
takes, and confirming it was benign is part of the audit trail.

Reference: SHA-256 `88b5b31f390a268ab3773df580d83fd1e388f49c2b685f78a16600577bd72fe2`.

**Fix:** regenerate the wrapper during the Gradle upgrade.

---

## A6 — The API test script defaults to a third-party remote host {#a6}

**Severity:** Medium · **Location:** `spec-api/run-api-tests.sh:6`

```bash
APIURL=${APIURL:-https://conduit.productionready.io/api}
```

If `APIURL` is not set, the script runs the entire RealWorld suite against a **public third-party
server** rather than localhost. The suite registers a user, so a bare `./spec-api/run-api-tests.sh`
sends a generated username, an email address, and a password off your machine to a host you do not
control.

`conduit.productionready.io` was the old RealWorld demo API and is no longer maintained, which
makes this a stale-domain risk of the same shape as A1 — if the domain changes hands, the default
path sends credentials to its new owner.

The documented invocations do override it (`README.md:70` and `.travis.yml:8` both set
`APIURL=http://localhost:…`), so the remote default is only reachable by running the script bare.
That is an easy mistake to make.

**Fix:** default to localhost — `APIURL=${APIURL:-http://localhost:8080/api}`. There is no reason
for a local spec-test runner to reach the public internet.

---

## A7 — `npx newman` executes an unpinned remote package {#a7}

**Severity:** Medium · **Location:** `spec-api/run-api-tests.sh:11`

```bash
npx newman run $SCRIPTDIR/Conduit.postman_collection.json
```

With `newman` not installed locally, `npx` downloads and executes it from the npm registry at run
time, resolving to **whatever version is current** — no version pin, no lockfile, no integrity
check. This is remote code execution by design, and it runs with your user's privileges.

The risk is ordinary npm supply-chain risk (a compromised release of `newman` or any of its
transitive dependencies), but it is unbounded here because nothing constrains which version
arrives.

**Fix:** pin the version (`npx newman@6.x`), or add newman as a devDependency with a lockfile, or
run it from a container. If CI runs the spec tests as the brief's bonus item suggests, pin it
there too.

---

## A8 — `set -x` echoes the test password into logs {#a8}

**Severity:** Low · **Location:** `spec-api/run-api-tests.sh:2`, with `:9` and `:16`

`set -x` traces every command, so the `--global-var "PASSWORD=$PASSWORD"` argument is printed with
the password expanded. Impact is low while the value is the default literal `password`, but it
becomes a real leak the moment someone runs the script with a meaningful `PASSWORD` — and in CI
that trace lands in build logs that are often world-readable.

**Fix:** drop `set -x`, or `set +x` around the invocation.

---

## A9 — No dependency verification, and no SCA was performed {#a9}

**Severity:** Informational

The project has no lockfile, no `gradle/verification-metadata.xml`, and no dependency-verification
configuration. Nothing pins or checks the transitive graph, so A1–A3 have no backstop.

**Scope limit, stated plainly:** I did **not** run a software-composition-analysis scan. Doing so
requires resolving the dependency graph, which means executing Gradle — outside the no-execution
constraint for this audit. So this audit says nothing about known CVEs in the resolved tree. If
you want that answer, the honest way to get it is a resolution in a throwaway container, or
GitHub's Dependabot alerts on the fork (`dependabot.yml` is already present and configured for
Gradle).

---

## A10 — The dependency stack is 2019-era and largely end-of-life {#a10}

**Severity:** Informational

| Dependency | Version | Note |
|---|---|---|
| Kotlin | `1.3.+` | 1.3 is EOL; also unpinned (A3) |
| Ktor | `1.2.3` | Released 2019; 1.x is superseded by 2.x/3.x |
| Exposed | `0.14.1` | Released 2019 |
| Unirest | `1.4.9` | Old, test-only |
| Gradle | `4.10` | 2018; cannot run JDK 17+ |
| H2 | `2.2.224` | Recent (2023) — postdates the H2 console RCE CVEs |
| HikariCP | `5.1.0` | Recent |
| slf4j-simple | `2.0.9` | Recent |
| JUnit | `4.13.2` | Current for JUnit 4 |

Dependabot has kept the leaf libraries current, but the framework layer (Kotlin, Ktor, Exposed)
and the build tooling are frozen in 2019. No CVE claims are made here — see the scope limit in A9.

The practical consequence is in the appendix of the main audit: the brief's JDK 17/21 CI matrix
forces a Gradle upgrade from 4.10 to at least 8.5, which cascades into `testCompile` removal, the
`jcenter()` removal in A1, and very likely a Kotlin version bump.
