# Decisions Index

Registry of architecture decisions made during planning.

| ID | Decision | Status | Date |
|----|----------|--------|------|
| [D001](D001_ship_all_three_features_as_slices.md) | Ship all three named features as ordered slices | accepted (agent-decided under delegation) | 2026-09-15 |
| [D002](D002_route_base_stays_at_root.md) | Route base stays at root | accepted (agent-decided under delegation) | 2026-09-15 |
| [D003](D003_public_reads_registered_before_auth_block.md) | Public reads registered before the mandatory auth block | accepted (agent-decided under delegation) | 2026-09-15 |
| [D004](D004_search_semantics.md) | Article Search semantics | accepted (agent-decided under delegation) | 2026-09-15 |
| [D005](D005_popular_ordering_and_favorites.md) | Popular ordering, pagination, favorite writes | accepted (agent-decided under delegation) | 2026-09-15 |
| [D006](D006_user_activity_counts.md) | User Activity counts | accepted (agent-decided under delegation) | 2026-09-15 |
| [D007](D007_articles_count_is_total_matches.md) | `articlesCount` is total matches | accepted (agent-decided under delegation) | 2026-09-15 |
| [D008](D008_authors_are_profiles.md) | Article and comment authors are Profiles | accepted (agent-decided under delegation) | 2026-09-15 |
| [D009](D009_slug_generation_and_test_data.md) | Slug generation, collisions, persistent test data | accepted (agent-decided under delegation) | 2026-09-15 |
| [D010](D010_enable_author_tests_per_slice.md) | Enable author tests per slice | accepted (agent-decided under delegation) | 2026-09-15 |
| [D011](D011_ci_pipeline_design.md) | CI pipeline, spec-test job, zero-run diagnosis | accepted (agent-decided under delegation) | 2026-09-15 |
| [D012](D012_delivery_branches_commits_prs.md) | Branches, fest commits, a PR per slice | accepted (agent-decided under delegation) | 2026-09-15 |

Every decision here was made by the planning agent under the user's standing delegation. The user said to keep
the `fest next` loop running until the festival reaches ready, and the approval judge decides checkpoints. Each record
names its evidence and the options it rejected, so the user can overrule any of them before execution starts.

## Status Values

- `proposed` — Under consideration
- `accepted` — Approved and final
- `superseded` — Replaced by a later decision

## Decision Template

Create `D###_title.md` files for each significant decision:

```markdown
# D001: [Decision Title]

**Status:** proposed | accepted | superseded
**Date:** YYYY-MM-DD

## Context

[Why is this decision needed?]

## Options

### Option A: [Name]
- **Pros:** [Benefits]
- **Cons:** [Drawbacks]

### Option B: [Name]
- **Pros:** [Benefits]
- **Cons:** [Drawbacks]

## Decision

[Which option was chosen and why]

## Consequences

[What changes or follow-up work results from this decision]
```
