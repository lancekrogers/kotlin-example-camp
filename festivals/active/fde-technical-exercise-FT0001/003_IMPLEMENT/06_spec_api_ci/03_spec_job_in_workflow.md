---
fest_type: task
fest_id: 03_spec_job_in_workflow.md
fest_name: spec_job_in_workflow
fest_parent: 06_spec_api_ci
fest_order: 3
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-15T12:05:06.148737-06:00
fest_tracking: true
---

# Task: Add the spec-test job to CI

## Objective

Add a `spec` job to the CI workflow that runs the RealWorld collection against a freshly built container and gates on the comparator.

## Requirements

- [ ] The job `needs: test`, builds the image from the repo's `Dockerfile`, starts it with a generated `JWT_SECRET`, and waits until the app answers. It then runs `spec-api/run-api-tests.sh` with `APIURL=http://localhost:8080`, followed by `python3 spec-api/compare_results.py`, whose exit code decides the job (D011).
- [ ] Every new action is pinned by commit SHA with its tag in a comment (`actions/setup-node` v7.0.0), matching the `test` job
- [ ] The newman report is uploaded as an artifact on every run (`if: always()`)
- [ ] On a real PR run, a deliberately broken manifest turns the job red; evidence is recorded in `results/03_spec_job.md`, then the manifest is restored

## Implementation

**Steps**

1. **Add the `spec` job** to `.github/workflows/gradle.yml` (the file `01_ci_pipeline` replaced), after the `test` job.
   - `<<EXPR x>>` stands for a GitHub Actions expression, as in `01_ci_pipeline` task 02.
   - Resolve `<SHA:setup-node>` with that task's `sha` helper, and reuse the checkout and upload-artifact SHAs already in the file.
   ```yaml
     spec:
       name: RealWorld spec tests
       needs: test
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@<SHA:checkout> # v7.0.1
         - uses: actions/setup-node@<SHA:setup-node> # v7.0.0
           with:
             node-version: '22'
         - name: Build image
           run: docker build -t ktor-realworld:ci .
         - name: Start app
           run: |
             docker run -d --name app -p 8080:8080 -e JWT_SECRET="$(openssl rand -hex 32)" ktor-realworld:ci
             for i in $(seq 1 60); do
               if curl -fsS http://localhost:8080/tags >/dev/null; then echo "app ready"; exit 0; fi
               sleep 1
             done
             docker logs app
             exit 1
         - name: Run collection
           run: APIURL=http://localhost:8080 REPORT=newman-report.json ./spec-api/run-api-tests.sh || true
         - name: Compare against expected failures
           run: python3 spec-api/compare_results.py newman-report.json spec-api/expected-failures.txt
         - name: Upload newman report
           if: always()
           uses: actions/upload-artifact@<SHA:upload-artifact> # v7.0.1
           with:
             name: newman-report
             path: newman-report.json
   ```
   Why the job is shaped this way:
   - **`|| true` on the collection step.** newman exits non-zero whenever any request fails, and some failures are expected. The comparator is the gate.
   - **Readiness probe on `/tags`.** It is the endpoint the compose healthcheck already uses (`compose.yaml:13`).
   - **A generated `JWT_SECRET`.** A fresh value per run keeps any secret out of the repository, the policy `compose.yaml:11` documents.
2. **Check local parity first.** The job mirrors `just docker up` plus `just docker spec`. Before pushing, confirm the comparator exits 0 locally.
3. **Prove the job fails on a real run.** This needs the user's authorization.
   1. Push `ci/spec-tests` and open the PR with `--repo lancekrogers/kotlin-ktor-realworld-example-app --base master`.
   2. Push a throwaway commit that deletes one line from `spec-api/expected-failures.txt`.
   3. Confirm the `RealWorld spec tests` job fails, with that request listed under UNEXPECTED FAILURES (`gh run view <id> --repo lancekrogers/kotlin-ktor-realworld-example-app --log`).
   4. Revert the commit, push again, and confirm the job goes green.
   5. Record both run URLs and the log excerpt in `results/03_spec_job.md`.

**Error paths**

- **The Start app step fails after 60 attempts:** read the `docker logs app` output in that step's log.
- **npx cannot download newman:** retry once. If it keeps failing, record it; never remove the version pin.
- **The comparator passes locally but fails in CI with a different set of failures:** inspect the uploaded report, and fix the cause, not the manifest.

## Done When

- [ ] All requirements met
- [ ] On the PR, the `RealWorld spec tests` job is green with the real manifest and red with one manifest line removed, and `results/03_spec_job.md` records both run URLs and the failing log excerpt
