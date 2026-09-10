# Go / no-go decision record

## Recommendation (analyst)

**Go, with one precondition.**

The repository is safe to read, build, and run locally. Nothing malicious was found on any
surface examined — see [SECURITY_AUDIT.md](SECURITY_AUDIT.md) for what was checked and cleared.

**Precondition before the first build:** apply the three-line build-hygiene change that closes
the supply-chain vectors before any dependency is fetched.

```diff
 repositories {
-    mavenLocal()
     mavenCentral()
-    jcenter()
 }
```
(in **both** the `buildscript` block and the project block — `build.gradle:17-21` and `:31-35`)

```diff
-        kotlin_version = "1.3.+"
+        kotlin_version = "1.3.72"
```
(`build.gradle:6`)

Rationale: [A1](findings-supply-chain.md#a1), [A2](findings-supply-chain.md#a2), and
[A3](findings-supply-chain.md#a3) all concern code that executes during Gradle's **configuration
phase**, before any project code runs. Making that change first means the first `./gradlew build`
resolves only pinned artifacts from Maven Central.

Secondary precaution, only if the spec-api script is used: set `APIURL` explicitly. Running
`./spec-api/run-api-tests.sh` bare sends registration credentials to a third-party host
([A6](findings-supply-chain.md#a6)).

## What remains unverified

Stated plainly, because a no-execution audit has real limits:

- **No CVE / SCA scan of the resolved dependency tree.** That requires resolving dependencies,
  which means executing Gradle. The stack is 2019-era and largely EOL
  ([A10](findings-supply-chain.md#a10)); no vulnerability claims are made about it either way.
  Dependabot on the fork is the cheapest way to get that answer.
- **No runtime behavior was observed.** Every finding is derived from reading source, bytecode
  strings, and archive listings. Findings B5, B6, and B8 are read from the code as written and
  have not been demonstrated against a running server.
- **Transitive dependency contents were never inspected** — only the coordinates declared in
  `build.gradle`. Nothing was downloaded.
- **The Gradle 4.10 distribution itself was not examined.** It is not in the repository; the
  wrapper fetches it at build time, unverified ([A4](findings-supply-chain.md#a4)).

## Decision

- **Status:** pending — awaiting Lance
- **Decided by:**
- **Date:**
- **Notes:**

Steps 3 (document the ask) and 4 (document the repository) of this workitem are blocked until
this is recorded.
