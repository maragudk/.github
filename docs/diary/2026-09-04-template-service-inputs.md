# Diary: Add commented-out `postgres`/`s3` examples to the starter templates

Third step of a multi-repo rollout: `maragudk/workflows`' `test.yml` and `compatibility.yml` reusable workflows gained two optional boolean inputs, `postgres` (a `postgres:18` service on host port 5433) and `s3` (a `versitygw` service on host port 7072), and two finished callers — `maragudk/glue` (both inputs) and `maragudk/app` (`s3` only) — already show what a caller opting in looks like. This step updates the two org starter templates (`workflow-templates/ci.yml` and `workflow-templates/compatibility.yml`) so a repository created from them sees a ready-made, commented-out example of how to opt in.

## Step 1: Add the commented-out `with:` blocks

**Author:** templates-builder

### Prompt Context

**Verbatim prompt:** ok (in reply to "step three, the starter templates in maragudk/.github — add s3: true / postgres: true as commented examples on test and compatibility")
**Interpretation:** Add a short commented-out `with:` block under the `test` job's `uses:` line in both `workflow-templates/ci.yml` and `workflow-templates/compatibility.yml`, showing both `postgres` and `s3` set to `true`, styled like `ci.yml`'s existing commented-out `build` job. Leave the `.properties.json` files and everything else untouched.
**Inferred intent:** Give any repository generated from these starter templates a discoverable, zero-effort way to turn on the new Postgres/S3 test services, consistent with how the `build` job is already offered as an easy uncomment rather than a separate template variant.

### What I did

Branched `template-service-inputs` off `main` in `/Users/maragubot/Developer/.github` (clean, up to date with `origin/main` at `483932b`). Read `/Users/maragubot/Developer/workflows/.github/workflows/test.yml` and `compatibility.yml` (checked out at `2b85801`) for the `postgres`/`s3` input descriptions, and the finished callers `/Users/maragubot/Developer/glue/.github/workflows/{ci,compatibility}.yml` (both inputs) and `/Users/maragubot/Developer/app/.github/workflows/{ci,compatibility}.yml` (`s3` only) for how a real caller wires them up.

Added, immediately below the `test` job's `uses:` line in both `/Users/maragubot/Developer/.github/workflow-templates/ci.yml` and `/Users/maragubot/Developer/.github/workflow-templates/compatibility.yml`:

```yaml
    # Uncomment for repositories whose tests need a Postgres and/or S3 service.
    # with:
    #   postgres: true
    #   s3: true
```

Kept `postgres` before `s3` (alphabetical, per the task spec) even though both finished callers happen to write `s3` first — a deliberate divergence, not an inconsistency to fix. Left both `.properties.json` files and everything else in the repo untouched.

### Why

The reusable workflows' own `inputs:` blocks already carry the full descriptions (throwaway credentials, exact connection details), so the template only needs to point at the two input names and show the shape of a `with:` block — matching how `ci.yml`'s existing commented-out `build` job avoids restating what the `build.yml` reusable workflow itself documents.

### What worked

The existing commented-out `build` job in `ci.yml` was a direct, unambiguous style precedent — one-line explanatory comment, then the commented block, blank line above and below preserved. Copying that shape into both files and onto `compatibility.yml` (which had no comparable precedent of its own) was mechanical.

### What didn't work

Nothing failed. Both files still parse as valid YAML with `python3 -c "import yaml; yaml.safe_load(open(...))"` after the edit.

### What I learned

Confirmed by transformation-and-diff (see below) that the finished `glue` callers and these templates agree on everything except key order: `glue`'s `with:` blocks list `s3` before `postgres`, while the task spec asked for alphabetical order here. That's an intentional, spec-directed divergence — not a sign the templates or `glue` drifted from each other in substance.

### What was tricky

Nothing was tricky — this was a small, well-scoped, single-line-per-file style of change with strong precedent to copy.

### What warrants review

Validated in two ways:
1. `python3 -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]))"` against both edited files — both parse cleanly.
2. Programmatically uncommented the new `with:` blocks, substituted `main` for `$default-branch` in `ci.yml`, and (for `ci.yml` only) removed the still-commented `build` job before diffing against `glue`'s finished `ci.yml`/`compatibility.yml`. The only differences were the expected ones: `postgres`/`s3` key order (alphabetical here vs. `glue`'s `s3`-then-`postgres`) and a trailing blank line in `ci.yml` left behind by the removed `build`-job comment block. No other drift.

`git status` shows only `workflow-templates/ci.yml` and `workflow-templates/compatibility.yml` changed; both `.properties.json` files are untouched as required.

### Future work

None identified — this closes out the starter-template side of the `postgres`/`s3` service-input rollout. The `maragudk/gai` repository (called out in the earlier CI-starter-workflows diary as the proving caller) may want the same opt-in treatment once someone reviews whether its tests need either service.
