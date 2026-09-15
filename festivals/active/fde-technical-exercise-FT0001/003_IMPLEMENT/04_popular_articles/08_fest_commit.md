---
fest_autonomy: high
fest_created: 2026-09-15T12:06:24.572773-06:00
fest_gate_id: fest-commit
fest_gate_type: commit
fest_id: 08_fest_commit.md
fest_managed: true
fest_name: Fest Commit Changes
fest_order: 8
fest_parent: 04_popular_articles
fest_status: pending
fest_tracking: true
fest_type: gate
fest_version: "1.0"
---

# Gate: Commit, Open the PR, and Merge When Green

Commit this sequence's changes with `fest commit`, then deliver them through a pull request on the fork (D012).

## Pre-Commit Checklist

- [ ] The testing, review and iterate gates for this sequence are complete
- [ ] `just gate` ends with `=== gate: PASSED ===`
- [ ] No debug code, temporary files, or secrets in the staged changes

## Commit

You **MUST** use `fest commit`, not `git commit`. `fest commit` tags the commit with task references for tracking.

```bash
fest commit -m "<type>: <summary>"
```

The message says what changed and why. Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`.

The following are **prohibited** in commit messages and PR descriptions:

- "Co-authored-by" tags for AI assistants
- AI tool attribution or advertisements
- Links to AI services or products

## Pull Request

Pushing, opening a PR and merging are outward-facing actions. Do them only when the user has authorized execution.

1. `git push -u origin <sequence branch>`
2. Open the PR against the fork explicitly. Without `--repo`, a fork's PR targets the upstream parent:
   ```bash
   gh pr create --repo lancekrogers/kotlin-ktor-realworld-example-app --base master --head <sequence branch> \
     --title "<slice title>" --body "<what changed and why; decisions implemented; evidence: census counts, results files>"
   ```
3. Wait for checks: `gh pr checks <n> --repo lancekrogers/kotlin-ktor-realworld-example-app --watch`. Both `build and test (JDK 17)` and `build and test (JDK 21)` must pass, and so must `RealWorld spec tests` once it exists.
4. Once the checks are green and the user authorizes it, merge in the same style as earlier slices, then run `camp fresh` in the project before the next sequence branches.
5. Record the PR URL and the green run URL in this sequence's `results/`.

## Definition of Done

- [ ] Commit created with `fest commit`, with no prohibited content
- [ ] PR opened against `lancekrogers/kotlin-ktor-realworld-example-app` `master`
- [ ] All required checks green on the PR
- [ ] Merged with the user's authorization, and local `master` synced with `camp fresh`