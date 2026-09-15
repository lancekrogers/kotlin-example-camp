# INGEST Presentation — fde-technical-exercise-FT0001

## Summary

The ingest phase turned the FDE exercise brief into structured specs for planning, grounded in the
code as it actually is on `master` at `bf1435e`, after the merged security remediation.

**Inputs (3):** `input_specs/exercise-brief.md` (the brief, verbatim), `codebase-state-verified.md`
(audit facts re-verified on `bf1435e`, plus step 5 sizing and route-base evidence), and
`agent-usage-record.md` (raw material for `AGENT_WORKLOG.md`). The last two are agent-written from this
camp's audit record and fresh verification. They are not user-authored.

**Outputs (4):** `purpose.md`, `requirements.md` (15 requirements, P0/P1/P2, traced to sources, each
with acceptance criteria), `constraints.md` (10 constraints, each with why it binds), and `context.md`.

**The finding that shapes everything:** the brief describes an app that "implements … articles,
comments, profiles, favorites, and pagination". The code has three tables (`Users`, `Follows`, `Tags`)
and no article data layer, and it never had one. All three features the brief offers read from tables
that do not exist. Domain models, routes, and an ignored 14-test article spec do exist, so the missing
piece is the data layer, and its design is already implied.

## How the user's intent was established at this checkpoint

This history is recorded because it changed the output, and the replay of this festival is meant to
show reasoning, not only conclusions.

1. **First presentation:** the agent recommended a substitute feature, profile stats over the follow
   graph, as the cheapest honest path.
2. **User:** "Shouldn't we use one of the 3 options?" The agent agreed: the brief allows a substitute
   only "if you think it would make for a better exercise", and cheaper is not that test. Further
   reading followed: domain models exist, the data layer never did, and the route-base evidence was
   gathered. The agent recommended Article Search as the smallest named option.
3. **User:** "Actually let's add all 3 in this festival." Specs revised: R1 now ships all three in a
   fixed delivery order, R7 records the departure from "Choose one", R14 grows by slice, R15 collects
   each feature's open semantics, and a scope tension is recorded in `constraints.md`.
4. **User:** asked for the approval judge to be set up to decide this checkpoint. The user did **not**
   approve the specs directly.

## Gate step 1 — does the output capture the intent?

Interpretive decisions, each with its justification:

| Decision | Justification | Where |
|---|---|---|
| Ship all three named features, in slices | User direction; the three share one article foundation | C10, R1, R7 |
| Delivery order: foundation → Search → Popular → User Activity | Ascending cost; the fork stays submittable after every slice | R1 |
| Build only the article slice the three features read | "Small product improvement"; update/delete/feed/etc. stay stubbed | R14, purpose non-goals |
| Route base: keep root (recommended, not decided) | Root since the first router commit; author moved user tests off `/api` in `6a09793`; spec runner puts `/api` in `APIURL` | R11, CS §Route base |
| New read endpoints must work without a token | `Router.kt:45` wraps all of `articles` in mandatory auth | R13 |
| `unfollow` bug demoted to P2 | No named feature wires follow/unfollow, so it stays unreachable | R8 |
| User Activity `favoritesCount` = favorites *given* (proposal) | The brief files the endpoint under "User Activity" | R15 |

## Gate step 2 — were all inputs processed?

Every file in `input_specs/` was read completely. `exercise-brief.md` was checked byte-for-byte against
its source. Every codebase claim was re-checked on `bf1435e`, and every `file:line` anchor in the specs
was opened. Four wrong line ranges and one dangling file reference were found and corrected. Raw output
is below.

## Gate step 3 — did the user validate the output?

**Not directly.** The user reviewed the step 5 summary, gave two corrections (both incorporated above),
and then delegated this approval decision to the approval judge. No operator approval of the specs
themselves has been given in person. This step is an `operator_attestation`, and nothing here should
be read as that attestation.

## Approval judge history

The user asked for an approval judge to decide this checkpoint. `judge-agent`
(`v0.0.0-20260826073251-6f163850f71b`) was installed and wired as the fest `approval_judge` hook.

1. **Run 1: blocked by readiness; the judge never started.** Error: `presentation artifact missing or
   empty`. This file was written in response.
2. **Run 2: judge failed with no verdict.** The claude preset got a JSON array instead of one result
   object, because `~/.claude/settings.json` sets `"verbose": true`. Fixed for this camp only, in
   `festivals/.festival/judge-agent.json`: the claude preset plus `--settings {"verbose":false}`.
3. **Run 3: rejected on one defect.** The End goal in `purpose.md` still said "one shipped feature"
   after the user chose all three. Fixed. Superseded notes were also added to the option-2 paragraph
   and the Search-first sizing sentence in `codebase-state-verified.md`. That run also said `fest
   validate` prints no numeric score. The agent accepted this after viewing output cut off at 30 lines,
   and replaced correct "90/100" citations with a false "no numeric score" claim.
4. **Run 4: rejected on that false claim.** The full `fest validate` output prints `Score 90/100` at
   line 64, after the ORDERING, AUTO-LINK, HOOKS and WORKFLOW sections the truncated view dropped.
   Fixed: the claim is removed here and in `PHASE_GOAL.md`, and the evidence block below is the complete, unedited output. A superseded note was
   also added to the "central decision" sentence in `codebase-state-verified.md`. The judge verified
   everything else against the brief, the code at `bf1435e`, git history, and GitHub.
5. **Run 5: approved step 5.** A stale-wording scan came back clean first. The judge then re-checked the
   specs against the session transcript, re-ran `diff` and `fest validate`, and re-verified the code
   anchors and GitHub state. It found no remaining defect.
6. **Phase gate step 1 (PHASE GOAL): approved.** The judge confirmed every brief section maps to a
   requirement and that no user direction is missing. It flagged two minor items, both fixed: this list
   did not record run 5, and `extraction-notes.md` said "Five named axes" while listing six.

Gate step 3 (APPROVAL) is an `operator_attestation` checkpoint. `fest` refuses to let a judge decide it,
so it waits for the user to approve it in a terminal.

## Needs a human decision

These are carried into `002_PLAN/decisions/` as proposals, not resolved here:

- **Route base (R11):** keep root (recommended) or move all routes to `/api`.
- **Feature semantics (R15):** Search's match rule and blank-`q` handling; Popular's tie-break for equal
  favorite counts; what User Activity's `favoritesCount` counts; whether `articlesCount` is page size
  (the author's tests) or total matches (the RealWorld spec).
- **CI zero-run cause (R10):** see the correction below. If the cause is GitHub's per-fork Actions
  opt-in, the fix is a human click in the GitHub UI.

## Corrections made during this phase

- An earlier draft said follow-graph counts "partially unblock" option 2. They don't, because option 2
  specifies article, comment, and favorite counts. Corrected in `codebase-state-verified.md`.
- An earlier draft said "GitHub disables Actions on new forks" as fact. The permissions API reports
  `enabled: true`. Corrected in R10, `codebase-state-verified.md`, and `extraction-notes.md`; the cause
  of zero runs is now marked undiagnosed.
- Four `file:line` ranges and one reference to a nonexistent `decision-points.md` were fixed.
- A claim that `fest validate` prints no numeric score was false. It came from reading output cut off at
  30 lines; the full output prints `Score 90/100`. Corrected here and in `PHASE_GOAL.md`.

## Not yet proven by execution

- `unfollow` deletes the wrong row orientation (static reading only, R8).
- Ktor 1.2.3 routes constant segments (`search`, `feed`) ahead of `{slug}`.
- What Exposed 0.41.1 offers on H2 for case-insensitive LIKE and escape characters.
- Why the fork has zero workflow runs.

## Where the work sits

- **Project:** `projects/kotlin-ktor-realworld-example-app` on `master` at `bf1435e`, clean. PR #1 merged
  and approved. No project code changed in this phase.
- **Camp root:** all INGEST work is **uncommitted**, including the festival directory and the judge hook
  in `festivals/.festival/config.yaml`. Last camp commit: `4293a16`.
- **Festival structure:** `fest validate` reports Score 90/100. Structure, completeness, task files, quality gates, ordering, auto-link, hooks, and workflow all pass, with 27 unfilled markers in 4 files. `001_INGEST/PHASE_GOAL.md` has 0
  unfilled markers. Festival-level and `002_PLAN` markers are still unfilled, which is expected before
  planning.

---

## Raw evidence (verbatim terminal output)

### PR #1 state and `camp fresh`

```
[{"baseRefName":"master","headRefName":"security/audit-remediation","mergedAt":"2026-09-13T19:10:36Z","number":1,"reviewDecision":"APPROVED","state":"MERGED","title":"Security audit remediation, toolchain modernization, and containerized tooling"}]
```

```
  kotlin-ktor-realworld-example-app
  ── Checkout master                   done
  ── Fetch origin                       done
  ── Sync master <- origin/master       updated 6 commit(s)
  ── Prune merged branches           deleted: security/audit-remediation
```

### Brief seeded verbatim (before rename `seed.md` → `exercise-brief.md`)

```
=== SEED DIFF vs SOURCE ===
IDENTICAL — full brief seeded (     238 lines)
```

### Three tables, two services, two repositories (on `bf1435e`)

```
=== TABLES ===
src/main/kotlin/io/realworld/app/domain/repository/TagRepository.kt:3:import org.jetbrains.exposed.dao.id.LongIdTable
src/main/kotlin/io/realworld/app/domain/repository/TagRepository.kt:9:internal object Tags : LongIdTable() {
src/main/kotlin/io/realworld/app/domain/repository/UserRepository.kt:5:import org.jetbrains.exposed.dao.id.LongIdTable
src/main/kotlin/io/realworld/app/domain/repository/UserRepository.kt:19:internal object Users : LongIdTable() {
src/main/kotlin/io/realworld/app/domain/repository/UserRepository.kt:38:internal object Follows : Table() {

=== SERVICES ===
TagService.kt
UserService.kt
=== REPOS ===
TagRepository.kt
UserRepository.kt
```

### The article data layer never existed

`git log --all --oneline --diff-filter=D --name-only -- '*ArticleService*' '*ArticleRepository*' '*CommentService*' '*CommentRepository*'`
printed nothing. The second line below is the script's own label:

```
=== ever existed? ===
(empty = never existed)
```

### Test baseline: four ignored classes, two user tests commented out

```
=== IGNORED TEST CLASSES ===
web/controllers/ProfileControllerTest.kt:13:@Ignore
web/controllers/ArticleControllerTest.kt:18:@Ignore
web/controllers/CommentControllerTest.kt:14:@Ignore
web/controllers/TagControllerTest.kt:14:@Ignore
```

```
=== UserControllerTest structure ===
13:class UserControllerTest {
18://    @Test
19://    fun `invalid login without pass valid body`() {
29:    @Test
30:    fun `success login with email and password`() {
42:    @Test
43:    fun `success register user`() {
58://    @Test
59://    fun `invalid get current user without token`() {
65:    @Test
66:    fun `get current user by token`() {
79:    @Test
80:    fun `update user data`() {
```

### Route base: root mount, and the author's move off `/api` in `6a09793`

```
90:        users(userController)
91:        profiles(profileController)
92:        articles(articleController, commentController)
93:        tags(tagController)
```

```
26:-        val response = appRule.http.post<UserDTO>("/api/users/login", userDTO)
27:+        val response = appRule.http.post<UserDTO>("/users/login", userDTO)
39:-        val response = appRule.http.post<UserDTO>("/api/users", userDTO)
40:+        val response = appRule.http.post<UserDTO>("/users", userDTO)
51:-        val response = appRule.http.get<UserDTO>("/api/user")
52:+        val response = appRule.http.get<UserDTO>("/user")
69:-        val response = appRule.http.put<UserDTO>("/api/user", userDTO)
70:+        val response = appRule.http.put<UserDTO>("/user", userDTO)
82:-        val response = post<UserDTO>("/api/users/login", userDTO)
83:+        val response = post<UserDTO>("/users/login", userDTO)
89:-        val response = post<UserDTO>("/api/users", userDTO)
90:+        val response = post<UserDTO>("/users", userDTO)
```

### Bundled RealWorld spec collection

```
total requests: 31
per folder: {'Auth': 5, 'Articles': 4, 'Articles, Favorite, Comments': 17, 'Profiles': 4, 'Tags': 1}
```

### The `unfollow` orientation bug (static)

```
   113	        val user = findByEmail(email) ?: throw NotFoundException("Email not found to follow")
   114	        val userToFollow = findByUsername(usernameToFollow) ?: throw NotFoundException("Username not found to follow")
   115	        transaction {
   116	            Follows.insert { row ->
   117	                row[Follows.user] = userToFollow.id!!
   118	                row[follower] = user.id!!
   119	            }
   120	        }
   121	        return userToFollow
   122	    }
   123	
   124	    fun unfollow(email: String, usernameToUnFollow: String): User {
   125	        val user = findByEmail(email) ?: throw NotFoundException("Email not found to unfollow")
   126	        val userToUnfollow = findByUsername(usernameToUnFollow)
   127	            ?: throw NotFoundException("Username not found to unfollow")
   128	        transaction {
   129	            Follows.deleteWhere {
   130	                Follows.user eq user.id!! and (Follows.follower eq userToUnfollow.id!!)
```

### CI: Actions enabled, workflow active, zero runs

```
=== actions permissions ===
{"enabled":true,"allowed_actions":"all","sha_pinning_required":false}

=== workflow runs total_count ===
0
=== workflows ===
CI with Gradle	active	.github/workflows/gradle.yml
```

### Festival structure (complete `fest validate` output, unedited)

```

Current Context
Node FT0001:P000.S00.T00

Self-Check Guidance
Use this reference in code comments for traceability:
  // TODO(FT0001:P000.S00.T00): Description of work needed

FESTIVAL VALIDATION
───────────────────
Festival fde-technical-exercise-FT0001
Path /Users/lancerogers/campaigns/amex/festivals/planning/fde-technical-exercise-FT0001

✓ STRUCTURE
✓ All checks passed

✓ COMPLETENESS
✓ All checks passed

✓ Task Files
Critical for AI execution
✓ All implementation sequences have task files

✓ QUALITY GATES
✓ All checks passed

○ Markers
Scaffolded, markers pending (not structurally broken)
⚠ Found 27 unfilled markers in 4 files

Unfilled markers are expected right after scaffolding.
Fill them with real task content: do not paste filler to restore the score.

Files needing attention
  • 002_PLAN/PHASE_GOAL.md
    File contains 7 unfilled template markers ([REPLACE:)
  • FESTIVAL_GOAL.md
    File contains 6 unfilled template markers ([REPLACE:)
  • FESTIVAL_OVERVIEW.md
    File contains 8 unfilled template markers ([REPLACE:)
  • TODO.md
    File contains 6 unfilled template markers ([REPLACE:)

Common marker types to replace
  [FILL: description]    → Write actual content
  [REPLACE: guidance]    → Replace with your content
  [GUIDANCE: hint]       → Remove and write real content
  {{ placeholder }}      → Fill in the value

Run 'fest markers list' to see all unfilled markers

✓ ORDERING
✓ All checks passed

✓ AUTO-LINK
✓ All checks passed

✓ HOOKS
✓ All checks passed

✓ WORKFLOW
✓ All checks passed

Score 90/100
Festival structure is valid; template markers are still pending

Suggestions
  • Structure can be valid with markers pending: fill them with real content, not filler
  • Run 'fest markers list' to see all unfilled template markers
  • Use 'fest markers fill' or edit files manually to replace markers with actual content

Agent Self-Check
Structure is valid. Unfilled markers are expected right after scaffolding.

  Fill markers as you write real task content: do not paste filler to restore the score.
     Run 'fest markers list' to see remaining placeholders.

════════════════════════════════════════════════════════════
  STRUCTURE VALID, MARKERS PENDING
════════════════════════════════════════════════════════════
```

### Camp-root working tree

```
4293a16 [amex:bb8421b0-WI-5205b8] Record the remediation and the go/no-go decision
=== uncommitted camp-root changes ===
 M .campaign/quests/default/ui-state.json
 M festivals/.festival/config.yaml
?? .campaign/fest/
?? festivals/.festival/.state/festival_events.jsonl
?? festivals/planning/fde-technical-exercise-FT0001/
```
