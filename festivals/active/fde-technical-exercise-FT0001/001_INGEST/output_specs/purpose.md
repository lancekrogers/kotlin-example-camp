# Purpose

## End goal

The fork at `github.com/lancekrogers/kotlin-ktor-realworld-example-app` is a complete, accessible
submission for the American Express FDE candidate technical exercise. It contains all three of the
brief's named features (Article Search, Popular Articles, and User Activity), each shipped as a slice
that fits the existing system. It also has tests that justify confidence in them, a CI pipeline that is
actually useful, an honest agent work log, and a recorded walkthrough.

## Why it matters

This is an interview submission, and the brief is explicit that it is **not** scored on box count:

> "We are evaluating the overall engineering approach, not whether you checked the largest number
> of boxes."

Six graded axes (`exercise-brief.md` §What We Care About): Engineering, Verification, Agentic
Engineering, Judgment, Communication, FDE Mindset. Two of them reward things that are not code:

- **Judgment** — "Where did you investigate further? What did you delegate? What did you verify
  yourself? What trade-offs did you make under the time constraint?"
- **FDE Mindset** — "Could you take the workflow you used here into a customer environment—where
  the codebase is unfamiliar, requirements are imperfect, and getting to a reliable result matters
  more than producing code quickly?"

The codebase makes this unusually literal. The brief describes an app that "implements …
articles, comments, profiles, favorites, and pagination"; the code implements users, a follow
graph, and tags. **All three offered features depend on tables that do not exist**
(`codebase-state-verified.md`). So the exercise as handed over is a requirements-imperfect
brownfield task — exactly the FDE scenario the last axis asks about. How that gap is handled is
the highest-signal part of the submission, above any individual endpoint.

## Success criteria

The festival is done when all six are true:

| # | Criterion | Verified by |
|---|---|---|
| 1 | All three named features shipped (Search, Popular, User Activity), each fitting existing conventions, with existing behavior preserved | Code review against `Router.kt`/service/repository patterns; existing 4 tests still green |
| 2 | Tests communicate intended behavior, edge cases, and that it works | Test names and assertions readable as a spec; suite green on JDK 17 and 21 |
| 3 | GitHub Actions builds and tests on JDK 17 **and** 21, surfaces failures clearly, caches Gradle | A **visible green run on the fork** — not just a committed YAML file |
| 4 | `AGENT_WORKLOG.md` covers all six points the brief lists, including a real mistake | Checklist against `exercise-brief.md` §5 |
| 5 | 5–10 minute walkthrough recorded, link in `AGENT_WORKLOG.md` | Link resolves for a logged-out viewer |
| 6 | Graders can access the repository and the recording | Fork visibility and recording permissions checked from outside the account |

## Explicit non-goals

- Finishing the RealWorld article subsystem. The three features need articles, tags, favorites, and
  comment creation (R14), and nothing more. Update, delete, get-by-slug, list, the personal feed, and
  comment listing and deletion stay stubbed and documented.
- Full RealWorld spec conformance.
- Any further security work. That shipped in PR #1 (`bf1435e`) and is out of scope here.

## Secondary purpose (from the user, not the brief)

The festival record is itself part of the submission story — `fest gif` will replay the run to show
the interviewer how the work was reasoned about. This raises the bar on decision documentation:
where a choice is made, the *why* and the rejected alternative need to be written down at the time,
not reconstructed afterwards.
