# Festival Structure

## Festival Goal

Ship all three of the FDE exercise brief's named features, Article Search, Popular Articles and User
Activity, into the fork `lancekrogers/kotlin-ktor-realworld-example-app`. Each one ships as a finished,
reviewed slice, with tests that justify confidence, CI on JDK 17 and 21, an honest agent work log, and a
recorded walkthrough, so the fork is submittable after every slice.

## Hierarchy

- **Festival:** fde-technical-exercise-FT0001
  - **Phase 001:** INGEST (ingest). Done: requirements structured and approved.
  - **Phase 002:** PLAN (planning). This phase: gaps, decisions D001-D012, this structure, the plan.
  - **Phase 003:** IMPLEMENT (implementation). Working directory: `projects/kotlin-ktor-realworld-example-app`.
    - **Sequence 01_ci_pipeline** (R3, R10; D011, D012)
      - 01_diagnose_zero_ci_runs
      - 02_replace_workflow_with_jdk_matrix
      - 03_prove_ci_fails_red
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 02_article_foundation** (R14, R2, R12; D002, D008, D009, D010)
      - 01_authors_as_profiles
      - 02_articles_and_article_tags_schema
      - 03_slug_generation
      - 04_article_repository_and_service_create
      - 05_wire_create_article_endpoint
      - 06_enable_author_create_and_tag_tests
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 03_article_search** (R1, R2, R13, R15; D003, D004, D007)
      - 01_prove_lower_on_clob_body
      - 02_search_repository_query
      - 03_search_service_validation_and_paging
      - 04_public_search_route
      - 05_search_endpoint_tests
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 04_popular_articles** (R1, R2, R13, R15; D003, D005, D007, D010)
      - 01_favorites_schema_and_repository
      - 02_favorite_endpoints_and_real_counts
      - 03_popular_query_service_and_route
      - 04_popular_tests_and_enable_author_favorite_tests
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 05_user_activity** (R1, R2, R13, R15; D003, D006, D008, D010)
      - 01_comments_schema_and_add_comment
      - 02_profile_stats_service_and_route
      - 03_stats_tests_and_enable_author_comment_test
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 06_spec_api_ci** (R9; D011)
      - 01_pin_newman_and_guard_api_url
      - 02_expected_failures_manifest_and_compare_script
      - 03_spec_job_in_workflow
      - quality gates: testing, review, iterate, fest commit
    - **Sequence 07_submission_docs** (R4, R7, R11; D001, D002, D006, D007). The name matches fest.yaml's `*_docs` gate exclusion, so it carries its own commit and PR task.
      - 01_readme_api_notes
      - 02_agent_worklog
      - 03_commit_and_open_docs_pr
  - **Phase 004:** DELIVER (non_coding_action). No sequences, PHASE_GOAL.md action items only.
    - Record the 5-10 minute walkthrough (human)
    - Add the recording link to AGENT_WORKLOG.md, commit, and merge
    - Verify the fork and the recording from a logged-out session
    - Sync the camp submodule pointer and record the final state

## Dependencies

- 02 depends on 01: every later slice's PR needs visible CI on both JDKs.
- 03 depends on 02, which provides the articles, tags and create path it searches.
- 04 depends on 02. It runs after 03 by D001's order, and turns Search's `favorited`/`favoritesCount` into real values.
- 05 depends on 04, because `favoritesCount` counts favorites given.
- 06 depends on 05 (its expected-failures manifest reflects the final set of endpoints) and on 01 (it extends the workflow).
- 07 depends on 06, since the docs and work log describe the finished state.
- Phase 004 depends on phase 003, since the recording shows finished work.
