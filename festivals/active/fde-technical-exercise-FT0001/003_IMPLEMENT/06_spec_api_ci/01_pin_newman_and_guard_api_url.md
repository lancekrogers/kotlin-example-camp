---
fest_type: task
fest_id: 01_pin_newman_and_guard_api_url.md
fest_name: pin_newman_and_guard_api_url
fest_parent: 06_spec_api_ci
fest_order: 1
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:05:06.106005-06:00
fest_updated: 2026-09-16T03:43:20.954528-06:00
fest_tracking: true
---


# Task: Pin newman and require an explicit APIURL

## Objective

Pin newman, make the spec runner refuse to run without an explicit `APIURL`, and add a containerized local recipe that runs the collection against the local app.

## Requirements

- [ ] `spec-api/run-api-tests.sh` runs `npx --yes newman@6.2.2`. When `APIURL` is unset it exits with a clear message instead of defaulting to `https://conduit.productionready.io/api` (`:6`) (D011).
- [ ] The runner writes a newman JSON report to the path in `REPORT` (default `newman-report.json`) for task 02's comparator, and no longer uses `set -x`, which would echo the generated password into logs
- [ ] A `just docker spec` recipe runs the collection from a digest-pinned `node` container against the app started by `just docker up`, so newman never runs on the host (C7)
- [ ] Work happens on branch `ci/spec-tests`, created from the fork's `master` after `05_user_activity` merged (D012)

## Implementation

**Notation used below.** `<<JUST x>>` stands for a `just` interpolation of `x` (two opening braces, `x`, two closing braces). The festival's marker scanner treats literal double braces as unfilled placeholders, which is why it is written this way.

**Steps**

1. **Branch:**
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c ci/spec-tests
   ```
2. **Replace the runner.** The current `spec-api/run-api-tests.sh` has three problems: it enables `set -x` (`:2`), it defaults `APIURL` to a remote host (`:6`), and it runs `npx newman run` with no version (`:11`). Replace the whole file with:
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   SCRIPTDIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"

   # No default: a missing APIURL must never silently target a remote host.
   : "${APIURL:?set APIURL to the API base, e.g. http://localhost:8080}"
   USERNAME=${USERNAME:-u$(date +%s)}
   EMAIL=${EMAIL:-$USERNAME@mail.com}
   PASSWORD=${PASSWORD:-password}
   REPORT=${REPORT:-newman-report.json}
   NEWMAN_VERSION=6.2.2

   npx --yes "newman@${NEWMAN_VERSION}" run "$SCRIPTDIR/Conduit.postman_collection.json" \
     --delay-request 500 \
     --reporters cli,json \
     --reporter-json-export "$REPORT" \
     --global-var "APIURL=$APIURL" \
     --global-var "USERNAME=$USERNAME" \
     --global-var "EMAIL=$EMAIL" \
     --global-var "PASSWORD=$PASSWORD"
   ```
   Keep the file executable. git records it as mode `100755`, and the CI job in task 03 runs it directly.
3. **Update the runner's README.** In `spec-api/README.md:8`, change the example to `APIURL=http://localhost:8080 ./run-api-tests.sh`, and add a sentence saying `APIURL` is required.
4. **Pin the node image by digest.** Node 22 is an LTS line whose Debian image includes `bash`:
   ```bash
   docker pull node:22-bookworm-slim
   docker image inspect node:22-bookworm-slim --format json | python3 -c 'import json,sys; print(json.load(sys.stdin)[0]["RepoDigests"][0])'
   ```
   Record the printed `node@sha256:...` value in `results/01_runner.md`.
5. **Add a `spec` recipe** to `.justfiles/docker.just`, after `verify` (`:120-125`).
   - The app container is `ktor-realworld-dev` (`:4`), started by `up` (`:24`).
   - A container cannot reach the host's `localhost` on macOS or colima. So the node container joins the app container's network namespace, where the app answers on `localhost:8080`.
   - Replace `<DIGEST>` with the value from step 4.
   ```just
   # Run the RealWorld Postman collection against the running app container
   [no-cd]
   spec:
       #!/usr/bin/env bash
       set -euo pipefail
       if ! docker inspect <<JUST container_name>> >/dev/null 2>&1; then
           echo "start the app first: just docker up" >&2; exit 1
       fi
       docker run --rm --network container:<<JUST container_name>> \
           -v <<JUST root>>/spec-api:/spec -w /spec \
           -e APIURL=http://localhost:8080 -e REPORT=/spec/newman-report.json \
           node:22-bookworm-slim@<DIGEST> ./run-api-tests.sh
   ```
6. **Ignore the report.** Add `spec-api/newman-report.json` to `.gitignore` (currently 44 lines).
7. **Run it locally.**
   1. `just docker up`, then `just docker spec`. Requests against still-stubbed endpoints will fail, so the exit code is non-zero; task 02 turns that into a gate.
   2. Confirm `spec-api/newman-report.json` exists.
   3. `just docker down`.
8. **Check the guard.** `env -u APIURL bash spec-api/run-api-tests.sh` must exit non-zero with the `APIURL` message before newman starts. That run needs no network and no toolchain.

**Error paths**

- **"cannot join network of a non running container":** the app is not up. Run `just docker up` first.
- **The run hangs on an install prompt:** `--yes` is missing from the `npx` call.
- **No report appears on the host:** `REPORT` points outside the mounted `/spec` directory.

## Done When

- [ ] All requirements met
- [ ] `just docker spec` runs the collection from the digest-pinned node container and leaves `spec-api/newman-report.json` on the host; `env -u APIURL bash spec-api/run-api-tests.sh` exits non-zero with the APIURL message; `grep -nE 'set -x|productionready' spec-api/run-api-tests.sh` finds nothing