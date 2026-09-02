# Diary: Add the `Security` starter workflow

The org's reusable `Security` workflow lives in `maragudk/workflows` (its PR #2). Callers need a small workflow file of their own carrying the triggers, concurrency, and permissions a reusable workflow does not inherit. This repository is the org `.github` repository, and `workflow-templates/` is where GitHub reads starter workflows for the Actions → New workflow picker, so the canonical caller file lives here.

## Step 1: Create the starter workflow

**Author:** main

### Prompt Context

**Verbatim prompt:** On the `maragudk/workflows` PR #2 README, Markus commented "Wouldn't it be better to have example files in .github repo?" and then "I'd much rather just have a single link to .github/.github/workflows, so nothing drifts." and, when asked whether to keep an inline copy as well, "a, and make the PR"
**Interpretation:** The caller example should exist exactly once, as a starter workflow in this repository, with the `workflows` README linking to it rather than duplicating it.
**Inferred intent:** One canonical caller file that every repository can adopt from the GitHub UI, with nothing to keep in sync by hand.

### What I did

Cloned `maragudk/.github` (previously `CONTRIBUTING.md`, `README.md`, `profile/`; no `workflow-templates/`). Added `/workflow-templates/security.yml`: the caller file from the `workflows` README with `$default-branch` placeholders in the `push` and `pull_request` branch filters, the `concurrency` block, `contents: read` plus `issues: write`, and a single job `uses: maragudk/workflows/.github/workflows/security.yml@main`. Added `/workflow-templates/security.properties.json` with name, description, `iconName: "octicon shield"`, category `Go`, and `filePatterns: ["go.mod$"]` so the template is offered to repositories with a Go module. Validated both files parse. Opened PR #1 with a note to merge after `workflows` PR #2 so the `uses:` target exists.

### Why

Starter workflows are GitHub's mechanism for a copy-once caller file; a README snippet in the `workflows` repository would be a second copy that drifts. Putting the template here means the `workflows` README can be a single link.

### What worked

The research done for the `workflows` repository had already established the `workflow-templates/` location, the `.properties.json` shape, and that the `.github` repository must be public for templates to appear in all repositories (it already was).

### What didn't work

Nothing failed.

### What I learned

The `$default-branch` placeholder is a textual substitution at instantiation, so it is usable anywhere in the template, not only under `branches:`.

### What was tricky

Only the path: the review comment mentioned `.github/.github/workflows`, but starter workflows must live at `workflow-templates/` in the repository root; corrected in the PR discussion.

### What warrants review

`/workflow-templates/security.yml` matches the caller shape the reusable workflow needs (triggers, concurrency, both permissions, `@main` pin); `/workflow-templates/security.properties.json` is valid JSON with the documented keys.

### Future work

Convert `gai`, `glue`, and `app` to callers using this template; `gomponents` as a new adopter. Further starter workflows as more reusable workflows are added to `maragudk/workflows`.

## Step 2: Address review

**Author:** main

### Prompt Context

**Verbatim prompt:** `/fabrik:address-code-review`, then "yes", then "Also diary?"
**Interpretation:** One inline comment on PR #1 — "Also default-branch?" on the `cancel-in-progress` expression — plus a request to add the implementation diary this repository was missing.
**Inferred intent:** Keep the template consistent with itself, and keep the diary convention across all repositories where this work happened.

### What I did

Changed `cancel-in-progress: ${{ github.ref_name != 'main' }}` to use `'$default-branch'`, so the whole template speaks in the placeholder. Replied to and resolved the thread. Started this diary.

### Why

The triggers already used the placeholder; a literal `main` two lines below them was an inconsistency waiting to bite a repository with another default branch.

### What worked

The single-comment review round was quick because the fix was one token.

### What didn't work

Nothing failed.

### What I learned

Nothing new.

### What was tricky

Nothing.

### What warrants review

The reusable workflow in `maragudk/workflows` still gates its issue steps on a literal `main` (decided there: all org repositories use `main`, and `github.event.repository.default_branch` is not reliably present on `schedule` events); this template is now fully placeholder-based, so a non-`main` repository would get the scan and concurrency right but never an issue.

### Future work

Unchanged from Step 1.
