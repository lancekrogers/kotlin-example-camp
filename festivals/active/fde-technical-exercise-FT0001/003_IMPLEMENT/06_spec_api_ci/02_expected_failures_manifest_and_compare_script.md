---
fest_type: task
fest_id: 02_expected_failures_manifest_and_compare_script.md
fest_name: expected_failures_manifest_and_compare_script
fest_parent: 06_spec_api_ci
fest_order: 2
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:05:06.128553-06:00
fest_updated: 2026-09-16T03:49:37.566411-06:00
fest_tracking: true
---


# Task: Add the expected-failures manifest and comparator

## Objective

Generate `spec-api/expected-failures.txt` from a real run, and add a comparator that fails on any unexpected failure or unexpected pass.

## Requirements

- [ ] Results are keyed by `folder / request name`, resolved from the collection file, because request names repeat across folders (for example, `All Articles` appears in both `Articles` and `Articles, Favorite, Comments`).
- [ ] `spec-api/compare_results.py` uses only the standard library, exits 0 only when the set of failed requests equals the manifest, and prints unexpected failures and unexpected passes separately
- [ ] Every group of manifest entries has a comment naming the stubbed endpoint that makes it fail (D001, D010)
- [ ] A deliberately wrong manifest (one entry removed, one bogus entry added) makes the comparator exit non-zero; the outputs are recorded in `results/02_comparator.md`

## Implementation

**Steps**

1. **Inspect the real report before writing any code.** Use the report task 01 produced:
   ```bash
   python3 - <<'EOF'
   import json
   d = json.load(open("spec-api/newman-report.json"))
   ex = d["run"]["executions"][0]
   print(sorted(ex.keys())); print(sorted(ex["item"].keys())); print(ex.get("assertions", [])[:1])
   EOF
   ```
   - Confirm each execution carries `item.id`, `item.name` and `assertions`, and that a failed assertion carries `error`.
   - A request can also fail with no assertion result at all. That appears as `requestError` and counts as a failure.
   - If the field names differ, use the names the file actually contains.
2. **Create `spec-api/compare_results.py`:**
   ```python
   #!/usr/bin/env python3
   """Compare a newman JSON report against the expected-failures manifest.

   Exit 0 only when the failing requests are exactly the ones listed. An unexpected
   failure is a regression; an unexpected pass means the manifest is stale.
   """
   import json
   import sys
   from pathlib import Path


   def request_names(collection):
       names = {}

       def walk(items, folder=None):
           for it in items:
               if "item" in it:
                   walk(it["item"], folder or it["name"])
               else:
                   names[it["id"]] = f"{folder} / {it['name']}" if folder else it["name"]

       walk(collection["item"])
       return names


   def failed_requests(report, names):
       failed = set()
       for ex in report["run"]["executions"]:
           broken = ex.get("requestError") or any(a.get("error") for a in ex.get("assertions", []))
           if broken:
               failed.add(names.get(ex["item"]["id"], ex["item"]["name"]))
       return failed


   def manifest(path):
       lines = Path(path).read_text().splitlines()
       return {l.strip() for l in lines if l.strip() and not l.lstrip().startswith("#")}


   def main(report_path, manifest_path, collection_path):
       names = request_names(json.loads(Path(collection_path).read_text()))
       failed = failed_requests(json.loads(Path(report_path).read_text()), names)
       expected = manifest(manifest_path)
       sections = (
           ("UNEXPECTED FAILURES (regressions)", sorted(failed - expected)),
           ("UNEXPECTED PASSES (update the manifest)", sorted(expected - failed)),
       )
       for title, items in sections:
           if items:
               print(title)
               print("\n".join(f"  {i}" for i in items))
       print(f"{len(failed)} failed, {len(expected)} expected")
       return 1 if any(items for _, items in sections) else 0


   if __name__ == "__main__":
       collection = sys.argv[3] if len(sys.argv) > 3 else str(Path(__file__).with_name("Conduit.postman_collection.json"))
       sys.exit(main(sys.argv[1], sys.argv[2], collection))
   ```
   Confirm the collection's items carry an `id` (Postman v2.1 collections normally do). If they do not, key both sides by folder and name instead.
3. **Generate the manifest from the real failing set.**
   1. Run `python3 spec-api/compare_results.py spec-api/newman-report.json /dev/null`. With an empty manifest, every failure prints under UNEXPECTED FAILURES.
   2. Write `spec-api/expected-failures.txt` from that list, grouped under comments such as `# GET /articles list and filters are still stubbed (D001)`.
   3. Check every entry against the stubbed list in D001 and D010. Register, login, current user, update user, create article, favorite/unfavorite, add comment and tags are all implemented. If one of those fails, it is a bug: record it and fix it on this branch. It does not go in the manifest.
   4. Later requests reuse variables set by earlier ones, such as the created article's slug. Where a stubbed endpoint makes a dependent request fail, say so in the comment.
4. **Prove the comparator works.**
   - Against the real report and manifest, it exits 0.
   - Delete one manifest line: it exits 1 and lists that line under UNEXPECTED FAILURES.
   - Add a bogus line: it exits 1 and lists it under UNEXPECTED PASSES.

   Paste all three outputs into `results/02_comparator.md`, then restore the manifest.

**Error paths**

- **`KeyError: 'id'`:** the report or the collection lacks item ids. Switch keys as described in step 2.
- **The failing set changes between runs:** a request depends on data or order (the collection builds usernames from `date +%s`). Record the flakiness and investigate it; do not paper over it with manifest entries.

## Done When

- [ ] All requirements met
- [ ] `python3 spec-api/compare_results.py spec-api/newman-report.json spec-api/expected-failures.txt` exits 0, and `results/02_comparator.md` shows exit 1 for both the removed-entry and bogus-entry manifests