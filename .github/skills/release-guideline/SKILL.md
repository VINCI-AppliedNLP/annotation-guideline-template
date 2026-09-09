---
name: release-guideline
description: 'Release a new guideline version end to end: commit pending work on dev, open a dev -> main pull request, review it, merge it, verify main matches dev, tag main with an inferred or user-supplied version, and push the tag to trigger the GitHub Release workflow. Use when: cutting a release, publishing a new guideline version, tagging and releasing, shipping dev to main, creating a versioned DOCX/PDF release.'
argument-hint: 'Optionally give the version to release (e.g., 1.2) and/or a release summary. Leave empty to let the skill infer the next version from tag history.'
---

# Release a Guideline Version (commit → PR → merge → verify → tag → release)

## When to Use
- Work on `dev` is finished and the team is ready to publish an official versioned guideline.
- A tagged GitHub Release with DOCX / PDF / versioned Markdown assets is needed.
- The user asks to "release", "cut a version", "tag and push", or "ship dev to main".

**Do not use** for routine edits. Day-to-day changes are simply committed and pushed to `dev` (see `README.md` § 1).

## Prerequisites
- `git` and the GitHub CLI (`gh`) are available and authenticated (`gh auth status`).
- The repository follows the template branching model: `main` = released versions only, `dev` = active development.
- The release workflow is `.github/workflows/pandoc-convert.yml` ("Convert Markdown to DOCX and PDF"). It runs on tag pushes and produces the GitHub Release.

## Release Model (what actually happens)
1. Pending work is committed and pushed to `dev`.
2. A pull request `dev` → `main` is opened, reviewed, and merged (merge commit). **Only unambiguous, trivial problems are fixed autonomously during review** — anything judgment-dependent is brought back to the user (see Step 6).
3. `main` is **verified to be up to date with `dev`** — the tag must not be cut from a branch that is missing commits.
4. A **lightweight tag is created on the merge commit of `main`** and pushed. The local checkout stays on `dev` — the tag is created from `origin/main`, so **no branch switching is required**.
5. The tag push triggers the workflow, which converts Markdown → DOCX/PDF, archives versioned Markdown into `archives/` on `dev`, and publishes a GitHub Release with all assets attached.

> The workflow commits the `archives/` files back to `dev` as `github-actions[bot]`, so the local `dev` branch must be pulled again after the release completes (Step 10).

## Versioning Convention
- Tags in this repository are **`MAJOR.MINOR` with no `v` prefix** (e.g., `1.0`, `1.1`). Follow whatever pattern the existing tags use — do not introduce a new format.
- The workflow strips a leading `v`/`V` and appends `_v<version>` to output filenames, so a tag of `1.2` yields `XX_Guideline(INCEpTION)_v1.2.docx`.

---

## Procedure

### 1. Preflight checks
Run these before changing anything and stop on any failure:

```bash
gh auth status
git rev-parse --abbrev-ref HEAD      # must be dev
git fetch origin --tags --prune
git status --short --branch
git --no-pager log --oneline origin/dev..dev      # unpushed local commits
git --no-pager log --oneline dev..origin/dev      # unpulled remote commits
```

Handle each condition:
| Condition | Action |
|---|---|
| Not on `dev` | Stop and ask the user whether to switch to `dev`. Never release from another branch. |
| Behind `origin/dev` | `git pull --ff-only origin dev` before continuing. |
| Diverged from `origin/dev` | Stop and ask the user how to reconcile. Do not force-push. |
| Working tree clean **and** nothing unpushed | Skip Step 3; the release covers already-pushed commits. |

Also check what is about to be committed and **exclude**:
- Generated artifacts — `*.docx`, `*.pdf`, `*.html` produced by conversion. These are never committed; the workflow regenerates them.
- Raw transcripts or any file containing PHI / identifiable patient data. Only de-identified content belongs in this repository.

If such files are present in the working tree, list them and ask the user before staging anything.

### 2. Determine the version
1. List tag history newest-first: `git --no-pager tag --sort=-v:refname`.
2. If the user supplied a version, validate it: it must match the existing tag format and must **not** already exist locally or on the remote:
   ```bash
   git ls-remote --tags origin "refs/tags/<version>"
   ```
   If it exists, stop and ask for a different version.
3. If no version was supplied, infer the next one from the latest tag:
   | Change since last release | Bump |
   |---|---|
   | New/removed/renamed entity classes, attributes, or allowed values; restructured schema | **MAJOR** (`1.4` → `2.0`) |
   | New rules, clarifications, added examples, new Key Updates entry | **MINOR** (`1.1` → `1.2`) |
4. **Always confirm the version with the user before proceeding** — state the inferred version, the previous tag, and the reason for the bump.

### 3. Commit and push pending work on `dev`
Skip if the tree is clean and nothing is unpushed.

- Stage only intended files (never `git add -A` blindly if Step 1 flagged artifacts).
- Write a commit message summarizing the guideline changes, not the release mechanics — e.g. `Add MMRC severity rules and round 7 adjudication examples`.
- Include the co-author trailer if the user's conventions require it.

```bash
git add <paths>
git commit -m "<summary>"
git push origin dev
```

### 4. Pre-release content checks (report, do not silently fix)
Verify and report anything questionable, then ask whether to proceed:
- The guideline Markdown filename contains **"Guideline"** (case-insensitive) — required for automatic archiving into `archives/`.
- The **Key Updates** section has an entry covering the changes in this release (see the `update-guideline` skill if it is missing).
- No root-level `.md` file already carries a `_vX.Y` suffix (those are skipped by the version-suffix logic).
- Images referenced in the guideline exist under `pics/`.

### 5. Open the pull request `dev` → `main`
Check for an existing open PR first:

```bash
gh pr list --base main --head dev --state open
```

If one exists, reuse it and update its body. Otherwise create it:

```bash
gh pr create --base main --head dev \
  --title "Prepare release version <version>" \
  --body-file <path-to-generated-body>
```

Generate the body from the commit range since the last release:

```bash
git --no-pager log --oneline <last-tag>..origin/dev
```

Body template:

```markdown
## Release <version>

### Summary
[1-3 sentences on what this release changes for annotators.]

### Changes since <last-tag>
- [Grouped, human-readable change bullets — guideline rules, examples, schema, tooling]

### Release checklist
- [ ] Key Updates section reflects these changes
- [ ] Examples renumbered and non-duplicated
- [ ] No generated DOCX/PDF/HTML committed
- [ ] No PHI or raw transcript content included

Tag to be pushed after merge: `<version>`
```

### 6. Review the pull request
Fetch and actually read the diff — do not approve blind:

```bash
gh pr diff <number>
gh pr view <number> --json mergeable,mergeStateStatus,reviewDecision,statusCheckRollup
```

Review for:
- Unintended files (generated artifacts, editor config, PHI, secrets).
- Guideline consistency: example numbering, terminology matching the schema, appendix subsection placement.
- Broken relative links or missing images.
- Workflow/CI checks passing (`statusCheckRollup`).
- Merge conflicts or a `mergeStateStatus` that is not clean.

#### Fix autonomously vs. consult the user
Only fix things that are **objectively wrong with exactly one obvious correction**. Everything else must be brought to the user with the `ask_user` tool before proceeding — do not guess, and do not silently pick an interpretation.

| Fix it yourself (then re-review) | **Stop and consult the user** |
|---|---|
| Typos, broken Markdown syntax, malformed tables | Any **content decision** — conflicting annotation rules, a new rule that contradicts an earlier one, ambiguous schema wording |
| Out-of-sequence example numbering | Whether a questionable example is correct, should be reworded, or should be dropped |
| A stale relative link with one clear target | **Version issues** — the inferred bump looks wrong, the change set spans what looks like more than one release, or a tag/`_vX.Y` filename disagrees with the intended version |
| Removing an accidentally committed `.docx`/`.pdf`/`.html` | **Merge conflicts** between `dev` and `main`, or unexpected commits on `main` |
| Trailing whitespace, inconsistent blank lines | Suspected **PHI, secrets, or credentials** — never quietly scrub these; report and let the user decide |
| | **Failing CI checks** whose cause is not an obvious edit error |
| | Anything you are less than confident about, or where more than one reasonable resolution exists |

When consulting, present: what you found, where (file and line), why it is ambiguous, and the concrete options you see — then wait for the user's decision. **Never merge, and never continue to Step 7, with an unresolved consultation outstanding.**

Report the review as a short findings list, separating "fixed automatically" from "needs your decision". If blocking issues are found, stop, fix them on `dev` (or apply the user's chosen resolution), push, and re-review.

Optionally record the review on GitHub:
```bash
gh pr review <number> --approve --body "<summary>"
```
> GitHub rejects self-approval of your own PR. If this fails, note it and continue — use `--comment` instead to leave the review notes.

### 7. Merge the pull request
**Ask the user for explicit confirmation before merging.** Then:

```bash
gh pr merge <number> --merge --delete-branch=false
```

- Use `--merge` (merge commit) to match this repository's history. Never squash or rebase — it would rewrite `dev`'s history relative to `main`.
- **Never** delete the `dev` branch.
- If the merge is blocked by branch protection (required approvals), stop and tell the user which requirement is unmet and who needs to approve. Do not attempt to bypass protection.

### 8. Verify `main` is up to date with `dev` (gate — do not skip)
The tag is cut from `main`, so `main` must already contain everything being released. Re-fetch and prove it:

```bash
git fetch origin --tags --prune
git rev-parse origin/main origin/dev
git merge-base --is-ancestor origin/dev origin/main   # must exit 0
git --no-pager diff --stat origin/main origin/dev     # must print nothing
git --no-pager log --oneline origin/main..origin/dev  # must be empty
```

Interpretation and required action:
| Result | Meaning | Action |
|---|---|---|
| Ancestor check passes and the diff is empty | `main` contains all of `dev`; contents are identical | Proceed to Step 9 |
| `origin/main..origin/dev` lists commits | Commits landed on `dev` after the PR merged, or the merge did not include them | **Stop.** Show the commits and ask whether to include them (update the PR / open a follow-up PR and merge) or to release without them |
| Diff is non-empty but the ancestor check passes | `main` has extra commits not in `dev` (e.g., a hotfix committed on `main`) | Report it; the tag will include those commits. Offer to merge `main` back into `dev` after the release so the branches reconverge |
| Ancestor check fails | The merge did not land, or history was rewritten | **Stop.** Do not tag. Re-check the PR state and report |

Also confirm the merge commit itself:
```bash
git --no-pager log --oneline -1 origin/main
```
Record this SHA — it is what the tag must point at.

### 9. Tag `main` and push
Tag the verified merge commit on `main` **without leaving `dev`**:

```bash
git tag <version> origin/main
git rev-parse <version>        # must equal the SHA recorded in Step 8
git push origin <version>
```

If the user instead wants the tag on the tip of `dev` (the older convention used by tags `1.0`/`1.1`), use `git tag <version> origin/dev` — but ask first, since the two commits differ once `main` has a merge commit.

> The workflow's `paths: '**.md'` filter does **not** apply to tag pushes (GitHub ignores path filters for tags), so the run triggers even when the merge commit touches no Markdown file.

### 10. Verify the release and resync `dev`
Watch the triggered workflow run:

```bash
gh run list --workflow=pandoc-convert.yml --limit 3
gh run watch <run-id>
```

Then confirm the release and its assets:

```bash
gh release view <version>
```

Expected assets: one `.docx`, one `.pdf`, and one `_v<version>.md` per converted root-level guideline file.

Finally, pull the archive commit the workflow pushed to `dev`:

```bash
git pull --ff-only origin dev
```

Confirm `archives/<Guideline>_v<version>.md` now exists locally.

### 11. Report
Summarize:
- Version released and the previous version.
- PR number and merge commit SHA.
- Result of the `main` ↔ `dev` verification gate.
- Tag name and the commit it points to.
- Workflow run conclusion and the release URL.
- Assets attached to the release.
- Anything skipped or flagged for follow-up.

---

## Failure Recovery

| Problem | Fix |
|---|---|
| Tag pushed with the wrong name/commit, release not yet published | `git push origin :refs/tags/<version>` then `git tag -d <version>`, and re-tag correctly. |
| Release published from a bad tag | `gh release delete <version> --yes`, delete the tag as above, fix the content on `dev`, then redo Steps 5-10 with a **new** version number rather than reusing the old one. |
| Tag push did not trigger the workflow | Fall back to manual dispatch, which is restricted to `dev`: `gh workflow run pandoc-convert.yml --ref dev -f create_tag=<version> -f create_release=true`. |
| Workflow failed mid-run | `gh run view <run-id> --log-failed`, fix on `dev`, then `gh run rerun <run-id>` or re-dispatch. |
| Workflow's archive commit conflicts with local `dev` | `git pull --rebase origin dev` and resolve; never force-push `dev`. |
| `main` has drifted ahead of `dev` after a hotfix | Open a `main` → `dev` sync PR (or `git merge origin/main` on `dev`) so the next release starts from converged branches. |

## Quality Checks
- Never tag before the Step 8 verification gate passes — a tag cut from a `main` that is missing commits produces a release with the wrong content.
- Never resolve a content decision, version ambiguity, or merge conflict on your own — consult the user (Step 6) and wait for a decision before merging.
- Never commit generated `.docx`, `.pdf`, or `.html` files — the workflow produces them as release assets.
- Never push a tag that already exists on the remote; released versions are immutable.
- Never merge without an explicit user confirmation, and never bypass branch protection.
- Never force-push `dev` or `main`, and never delete the `dev` branch.
- Never commit raw transcripts, PHI, secrets, or credentials.
- Confirm the inferred version with the user before tagging — an incorrect tag is public the moment it is pushed.
- The release is only complete when the workflow run has concluded successfully **and** `gh release view <version>` lists the expected assets.
