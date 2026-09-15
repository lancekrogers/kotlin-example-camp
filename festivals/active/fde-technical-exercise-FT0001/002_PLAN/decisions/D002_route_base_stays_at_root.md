# D002: Route base stays at root

**Status:** accepted (agent-decided under the user's delegation; see `../inputs/gaps.md`)
**Date:** 2026-09-15
**Traces:** R11, B§1 ("Follow existing conventions … preserve existing behavior", `docs/interview-exercise.md:96`)

## Context

The brief writes every endpoint as `/api/…`. This app has never served under `/api`:

- The routers are mounted straight onto `Routing` (`AppConfig.kt:89-93`). The earliest
  Ktor router in `5759ef1` was already mounted at root.
- In `6a09793`, while implementing users, the author changed six test paths from `/api/users…` and
  `/api/user` to `/users…` and `/user`.
- The bundled spec runner puts `/api` in its base URL. `APIURL` defaults to
  `https://conduit.productionready.io/api` (`spec-api/run-api-tests.sh:6`), and all 31 Postman
  requests are `{APIURL}/users`, `{APIURL}/articles`, and so on. The README runs it against this app with
  `APIURL=http://localhost:8080` (`README.md:95`).

## Options

### Option A: Keep root, and document how the brief's paths map onto it
- **Pros:** no existing URL changes. Consistent with every working route and the author's precedent.
- **Cons:** a grader who types `/api/articles/search` gets a 404 unless they have read the README.

### Option B: Move every route under `/api`
- **Pros:** the brief's paths work as written.
- **Cons:** changes the URL of every working endpoint, against "preserve existing behavior", and reverses
  `6a09793`. It touches `AppConfig.kt`, `UserControllerTest.kt` (4 paths), `HttpUtil.kt` (2),
  `compose.yaml:13`, `README.md:95`, `.justfiles/docker.just:83,93,99,102,107,112`, and
  `.justfiles/security.just:145`, all for no functional gain.

### Option C: Put only the new endpoints under `/api`
- **Pros:** the brief's paths work for the new features.
- **Cons:** two routing conventions in one app, with nothing to justify the split.

## Decision

**Option A.** The new endpoints are:

| Brief path | This app |
|---|---|
| `GET /api/articles/search?q=` | `GET /articles/search?q=` |
| `GET /api/articles/feed/popular` | `GET /articles/feed/popular` |
| `GET /api/profiles/:username/stats` | `GET /profiles/{username}/stats` |

## Consequences

- The author's ignored tests use `/api/articles`, `/api/profiles`, and `/api/tags`. When a slice enables
  them (D010), their paths lose the `/api` prefix, as does `HttpUtil.kt:70`. Only test code changes.
- `README.md` gets a short "API base path" note with the table above, and `AGENT_WORKLOG.md` explains
  the choice.
- The spec-test job (D011) sets `APIURL=http://localhost:8080` explicitly.
