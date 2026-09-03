# Diary: Add the `CI`, `Compatibility`, and `CD` starter workflows

Companion to `/Users/maragubot/Developer/.github/docs/diary/2026-09-02-security-starter-workflow.md`, which established the starter-workflow shape (`workflow-templates/<name>.yml` + `<name>.properties.json`) for the org's `Security` reusable workflow. A parallel builder is adding four more reusable workflows to `maragudk/workflows` (`lint.yml`, `test.yml`, `compatibility.yml`, `build.yml`, `cd.yml`, each with no inputs — see that repository's `docs/decisions.md`, 2026-09-02 entry). This step adds the three caller templates that consume them.

## Step 1: Create the CI, Compatibility, and CD starter workflows

**Author:** templates-builder

### Prompt Context

**Verbatim prompt:** Create and work on branch `ci-starter-workflows` from `main`. Read `workflow-templates/security.yml` and `security.properties.json` as the established shape, plus the security-starter-workflow diary and today's `workflows` repo decisions entry. Build three templates — `ci.yml` (lint + test jobs, plus a commented-out `build` job), `compatibility.yml` (scheduled test job), `cd.yml` (publish job) — each calling the corresponding reusable workflow in `maragudk/workflows@main`, each with a matching `.properties.json` following `security.properties.json`'s shape. Read `gai`'s `ci.yml`/`compatibility.yml` and `app`'s `cd.yml` for the triggers, concurrency, and permissions the callers currently carry. Nothing else in the repo changes.
**Interpretation:** Three new caller files plus three properties files, structurally identical in spirit to `security.yml`/`security.properties.json` but with per-workflow triggers and permissions as specified, and `uses:` targets pointing at the five reusable workflow names the parallel builder is adding.
**Inferred intent:** Give every Go repository in the org a one-click way to adopt the new job-sized reusable workflows from the Actions → New workflow picker, matching the pattern already proven for `Security`.

### What I did

Created branch `ci-starter-workflows` from `main` in `/Users/maragubot/Developer/.github`. Read `/Users/maragubot/Developer/.github/workflow-templates/security.yml` and `security.properties.json` for the shape, the security starter-workflow diary for context on `$default-branch` and starter-workflow mechanics, `/Users/maragubot/Developer/workflows/docs/decisions.md`'s 2026-09-02 entry for the five reusable workflow names and their zero-input, no-`permissions:`-declared contract, and `/Users/maragubot/Developer/gai/.github/workflows/{ci,compatibility}.yml` plus `/Users/maragubot/Developer/app/.github/workflows/cd.yml` as the source of triggers/concurrency/permissions to carry into the callers.

Added six files under `/Users/maragubot/Developer/.github/workflow-templates/`:
- `ci.yml` — `name: CI`; `push`/`pull_request` on `$default-branch`; the standard concurrency block; `permissions: contents: read`; jobs `lint` and `test` calling `lint.yml@main` and `test.yml@main`; a commented-out `build` job calling `build.yml@main` with a one-line comment above it ("Uncomment for repositories with a Dockerfile."). Deliberately omitted `workflow_dispatch` and the `evaluate`/other repo-specific jobs present in `gai`'s `ci.yml` — the spec enumerated exactly `push` + `pull_request` as triggers and exactly `lint`/`test`/`build` as jobs, and repo-specific jobs stay inline per the `workflows` repo's decision.
- `compatibility.yml` — `name: Compatibility`; `schedule` cron `"14 7 * * *"` + `workflow_dispatch`; `permissions: contents: read`; job `test` calling `compatibility.yml@main`.
- `cd.yml` — `name: CD`; `push` on `$default-branch` only (no concurrency block, not requested); `permissions: contents: read, packages: write`; job `publish` calling `cd.yml@main`.
- `ci.properties.json`, `compatibility.properties.json`, `cd.properties.json` — following `security.properties.json`'s shape: one-sentence `description`, an `octicon` `iconName` (`check`, `beaker`, `rocket` respectively), `categories: ["Go"]`, and `filePatterns` of `["go.mod$"]` for `ci`/`compatibility` and `["Dockerfile$"]` for `cd`.

### Why

The reusable workflows in `maragudk/workflows` declare no `permissions:` and take no inputs by design (per that repo's decision log), so callers must supply triggers, concurrency, and permissions themselves — exactly the gap `security.yml` already closes for the `Security` workflow. Splitting `ci.yml`'s jobs onto `lint.yml`/`test.yml`/`build.yml` (the last commented out) mirrors the "job-sized reusable workflow, composed by the caller" design explicitly chosen in the `workflows` repo rather than one monolithic CI reusable workflow.

### What worked

The `security.yml`/`security.properties.json` pair was a complete, unambiguous template for structure — copying its conventions (`$default-branch` placeholder, concurrency expression, properties.json key order) left little room for guessing. The `workflows` repo's decision entry independently confirmed every trigger/permission choice the task spec asked for, so there was no conflict to reconcile.

### What didn't work

Nothing failed. All three YAML files parsed with `python3 -c "import yaml; yaml.safe_load(...)"` and all three JSON files parsed with `python3 -c "import json; json.load(...)"`; `$default-branch` parses fine as a plain scalar wherever it appears, consistent with the security diary's note that it's a textual substitution usable anywhere in the template.

### What I learned

The `workflows` repo's decision log spells out that no reusable workflow there declares `permissions:` — the caller grants them, and the called workflow runs under the caller's `GITHUB_TOKEN`. That's what makes `cd.yml`'s `packages: write` meaningful to carry in the template rather than in the reusable workflow itself, and is worth keeping in mind if a future reusable workflow needs a permission no caller currently grants.

### What was tricky

Choosing `iconName` values: `security.properties.json` set a precedent (`octicon shield`) but nothing in the spec named icons for the other three. Picked plausible standard octicons (`check`, `beaker`, `rocket`) for CI/Compatibility/CD respectively — reasonable guesses, not verified against GitHub's live icon picker, so worth a glance in the Actions UI after this merges.

### What warrants review

- Grep-verified all five `uses:` lines (including the commented one) point at `maragudk/workflows/.github/workflows/<name>.yml@main` for exactly `lint`, `test`, `build`, `compatibility`, `cd` — no typos, no stray targets.
- `git status --short` shows only the six new files under `workflow-templates/`; nothing else in the repo changed.
- `ci.yml` intentionally has no `workflow_dispatch` trigger and no `evaluate`/lint-adjacent extra jobs beyond `lint`/`test`/commented-`build`, diverging from `gai`'s current `ci.yml` — confirm that's the intended scope (repo-specific jobs are meant to stay inline in each caller, not templated) rather than an omission.
- `iconName` choices (`octicon check`, `octicon beaker`, `octicon rocket`) are unverified guesses; check they render in the GitHub UI.

### Future work

Convert `gai`'s `ci.yml`/`compatibility.yml` and `app`'s `cd.yml` (and other org repositories) to use these templates once the corresponding reusable workflows land in `maragudk/workflows@main`. `gai` is called out in the `workflows` repo's decision log as the proving caller for lint/test/compatibility, left for a human to merge.
