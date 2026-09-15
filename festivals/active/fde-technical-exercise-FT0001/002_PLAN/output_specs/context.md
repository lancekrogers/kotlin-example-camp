# PLAN Context: where the work sits and where facts came from

## Where the work landed

- **Nothing committed.** The festival directory, the judge hook in `festivals/.festival/config.yaml`, and
  `festivals/.festival/judge-agent.json` are uncommitted in the camp root. Git state is in `PRESENTATION.md`.
- **No project code changed.** The project is on `master` at `bf1435e`, where PR #1 (the security
  remediation) was merged.
- **No branches or PRs created.** Branches and per-slice PRs start at execution (D012).
- **Scaffold landed in the festival directory only.** Phases `003_IMPLEMENT` and `004_DELIVER`, 27 task files, 24
  gate files and the customized gate templates are all uncommitted in the camp root, like the rest of the
  festival.
- **Tooling behavior observed.** In fest v0.8.0, `fest create phase --dry-run` creates a real phase, and
  `fest remove phase --force` still prompts for confirmation. Both are recorded with evidence in
  `PRESENTATION.md`, Scaffolding.

## Provenance of upstream facts

| Fact | Source | Used by |
|---|---|---|
| Route selector qualities; tie resolution by registration order | `ktorio/ktor` at tag `1.2.3`: `RouteSelector.kt:28,33`, `RoutingResolve.kt:119-129`, `Authentication.kt:282,297,321-323` | D003 |
| `lowerCase`, `LikePattern.ofLiteral`, `like` with `ESCAPE`, `count`, `groupBy`, `orderBy`, `limit` | `JetBrains/Exposed` at tag `0.41.1`: `SQLExpressionBuilder.kt:20,58,135-178,409-410`, `Op.kt:479-486`, `Query.kt:164`, `AbstractQuery.kt:41,46-48` | D004, D005 |
| H2 LIKE special characters and text type | Exposed `0.41.1` `vendors/Default.kt:63,669`, `vendors/H2.kt:215` | D004 |
| Action versions: checkout v7.0.1, setup-java v6.0.1, gradle/actions v6.3.0, action-junit-report v6.5.0, dorny/test-reporter v3.0.0 | GitHub releases API, checked 2026-09-15 | D011 |
| newman 6.2.2 | npm registry, checked 2026-09-15 | D011 |
| Fork CI state: Actions enabled, 0 runs, public fork of `Rudge/...` (27 upstream runs) | GitHub REST API, checked 2026-09-14 and 2026-09-15 | D011, R10 |
