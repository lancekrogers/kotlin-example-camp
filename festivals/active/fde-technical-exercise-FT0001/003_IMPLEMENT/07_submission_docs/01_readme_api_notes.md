---
fest_type: task
fest_id: 01_readme_api_notes.md
fest_name: readme_api_notes
fest_parent: 07_submission_docs
fest_order: 1
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:03:15.713604-06:00
fest_updated: 2026-09-16T04:07:44.955718-06:00
fest_tracking: true
---


# Task: Document the API additions and the checks in README

## Objective

Update `README.md` so a grader can use the three new endpoints and run every check without reading the code.

## Requirements

- [ ] A new `# API additions` section documents:
  - the base-path mapping from the brief's `/api/...` to this app's root routes (D002);
  - `GET /articles/search`, `GET /articles/feed/popular` and `GET /profiles/{username}/stats`, each with parameters, response shape and error statuses;
  - that `articlesCount` is the total number of matches (D007), and what each stats count means, including how it differs from `Article.favoritesCount` (D006);
  - that these reads are public, but a request with an invalid token still gets 401 (D003).
- [ ] The stale `# Getting started` section (`README.md:79-95`) is replaced with the Docker-only workflow and a CI section. Its problems: "You need just JVM installed" (`:81`), and a link labeled 8080 that points at port 7000 (`:83`)
- [ ] Every JSON example is captured from a running container, not hand-written
- [ ] Work happens on branch `docs/submission`, created from the fork's `master` after `06_spec_api_ci` merged (D012)

## Implementation

**Steps**

1. **Branch.**
   ```bash
   cd projects/kotlin-ktor-realworld-example-app
   camp fresh
   git switch -c docs/submission
   ```
2. **`# Development`** (`README.md:54`). It already explains the `just` modules. Add `just build matrix` and `just docker spec` to its code block, each with a one-line comment.
3. **Replace `# Getting started`** (`:79`, up to `# Help` at `:97`) with a short section:
   - Docker is the only prerequisite.
   - `just docker up` serves the app on http://localhost:18080 (`.justfiles/docker.just:5`).
   - `just test all` runs the test suite; `just build matrix` builds on JDK 17 and 21.
   - `just docker spec` runs the RealWorld collection against the running app.
   - Remove the host `./gradlew run` instructions, which contradict the containers-only rule.
4. **Add `# API additions`** after Development.
   - **Base path** (from D002):

     | Brief | This app |
     |---|---|
     | `GET /api/articles/search?q=` | `GET /articles/search?q=` |
     | `GET /api/articles/feed/popular` | `GET /articles/feed/popular` |
     | `GET /api/profiles/:username/stats` | `GET /profiles/{username}/stats` |

   - **Each endpoint:**
     - its query parameters: `q` is required; `limit` is 1..100 with default 20; `offset` is ≥ 0 with default 0;
     - a captured example response;
     - its errors: 422 with the actual messages, 404 for an unknown username, 401 for an invalid token.
   - **`articlesCount`:** the total number of matching articles, not the size of the returned page (D007).
   - **Stats counts** (D006), as a table: articles authored, comments written, favorites given. Add one sentence noting that `Article.favoritesCount` counts favorites an article *received*.
   - **Viewer fields:** with a valid token, `favorited` and `author.following` reflect the viewer.
   - **Ordering and slugs:**
     - Search is newest first.
     - Popular is sorted by favorite count, then newest.
     - Slugs come from titles and get a numeric suffix on collision; `search` and `feed` are reserved (D009).
5. **Add `# CI`.**
   - The two jobs: `build and test (JDK 17/21)` and `RealWorld spec tests`.
   - Where failures show up: the `JUnit (JDK …)` check annotations on a PR, and the `newman-report` artifact.
   - What `spec-api/expected-failures.txt` is for, and the rule for changing it: an entry is added only for an endpoint that is still stubbed.
6. **Capture the examples.** Run `just docker up`, then create a user, articles, a favorite and a comment with curl, as earlier tasks did. Call the three endpoints and paste trimmed real output. Finish with `just docker down`.

**Error paths**

- **An example does not match real output:** re-capture it. Never hand-edit an example.
- **`just security secrets` flags the README:** a real token was pasted into an example. Replace it with `<token>`.

## Done When

- [ ] All requirements met
- [ ] `README.md` has `# API additions` and `# CI` sections that cover every item in both requirements, with no host `./gradlew run` instructions left in Getting started, and every JSON example traceable to a captured response