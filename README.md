# Contributing Guide — Annotation Guideline Development Workflow

This document explains how to contribute to this annotation guideline repository, including everyday editing, review, and releasing a new version.

> This repo was created from the [annotation-guideline-template](https://github.com/VINCI-AppliedNLP/annotation-guideline-template).

---

## Repository Structure Overview

```
├── <Your_Guideline>.md    ← Source of truth (edit this)
├── archives/              ← Versioned Markdown backups (auto-generated on release)
├── pics/                  ← Images referenced in the guideline
├── style.css              ← PDF styling (used during conversion)
├── .github/workflows/     ← GitHub Actions automation
├── CONTRIBUTING.md        ← This file
└── README.md
```

> **Note:** You may add additional folders (e.g., `change_logs/`, `meeting_minutes/`) as your project needs. Only **root-level** `.md` files (excluding `README.md`, `CONTRIBUTING.md`) are processed by the workflow.

## Branching Model

| Branch | Purpose |
|--------|---------|
| `main` | **Stable released versions only.** Never commit directly here. |
| `dev`  | **Active development branch.** All day-to-day edits happen here. |

> After creating a repo from the template, create a `dev` branch if one does not already exist.

---

## 1. Day-to-Day Editing — Commit & Push to `dev`

**When to use:** Routine edits to the annotation guideline — wording tweaks, adding examples, fixing typos, updating tables, etc.

You can edit using **either** of these methods:

### Option A: Local Editor + Git Push

1. Make sure you are on the `dev` branch:
   ```bash
   git checkout dev
   git pull origin dev
   ```

2. Edit your **guideline Markdown file** (e.g., `XX_Guideline.md`).
   > ⚠️ **Do NOT edit `.docx` or `.pdf` files directly** — they are auto-generated and will be overwritten.

3. Stage, commit, and push your changes:
   ```bash
   git add XX_Guideline.md
   git commit -m "Brief description of your change"
   git push origin dev
   ```

### Option B: GitHub Web UI

1. Navigate to the repository on GitHub and switch to the `dev` branch.
2. Open your guideline `.md` file and click the **pencil icon** (✏️) to edit.
3. Make your changes, add a commit message, and commit directly to `dev`.

### What happens after pushing to `dev`

The GitHub Actions workflow automatically converts all root-level Markdown files (except `README.md` and `CONTRIBUTING.md`) to DOCX and PDF. The generated files are **not** committed to the repo — they are only available as **workflow artifacts** (downloadable from the Actions tab for 30 days) or attached to **releases**.

### What belongs in a commit & push

| ✅ Do commit & push | ❌ Don't commit & push |
|---|---|
| Typo fixes | Changes that need team review before merging to `main` |
| Adding/updating annotation examples | New version releases |
| Small wording or formatting improvements | Experimental large-scale restructuring |

---

## 2. Preparing a Release — Pull Request from `dev` → `main`

**When to use:** When the guideline on `dev` is ready to become the next official version — typically after team review and agreement.

### Option A: Using VS Code with GitHub Copilot (Recommended)

1. Ensure all your changes are committed and pushed to `dev`.
2. In VS Code, open the **GitHub Pull Request** extension or ask **GitHub Copilot** to create a pull request:
   > _"Create a pull request from dev to main titled 'Prepare release version X.Y'"_
3. Copilot can help **generate a summary of the updates** since the last release — ask it:
   > _"Summarize the changes between main and dev for the PR description"_
4. Review the PR, request reviewers if needed, and merge.

### Option B: Using GitHub Web UI

1. Ensure all your changes are committed and pushed to `dev`.

2. Go to your repository on GitHub.

3. Click **"New pull request"**:
   - **Base branch:** `main`
   - **Compare branch:** `dev`

4. Fill in the PR title and description:
   - Title: e.g., `Prepare release version 1.0`
   - Description: summarize what changed since the last release.

5. **Request reviewers** if needed.

6. After approval, **merge the pull request** (use "Merge commit" or "Squash and merge").

### When to use a Pull Request

| ✅ Use a PR | ❌ Don't need a PR |
|---|---|
| Merging `dev` → `main` for a new release | Small edits on `dev` (just push) |
| Major structural changes that need team review | Minor formatting or documentation updates |
| Content requiring expert sign-off | |

---

## 3. Releasing a New Version — Creating a Tagged Release

**When to use:** After the Pull Request is merged into `main` and you want to create an official tagged release with downloadable DOCX/PDF files.

There are **two ways** to create a release:

### Option A: Tag & Push (via Git CLI or Copilot)

Create and push a Git tag on `dev`. This triggers the workflow automatically and creates a full GitHub Release.

Using **Git CLI**:
```bash
git checkout dev
git pull origin dev
git tag 1.0
git push origin 1.0
```

Or ask **GitHub Copilot** in VS Code:
> _"Tag 1.0 and push"_

### Option B: Manual Workflow Dispatch (via GitHub Actions UI)

1. Go to the repository on GitHub.

2. Click the **"Actions"** tab.

3. Select the **"Convert Markdown to DOCX and PDF"** workflow from the left sidebar.

4. Click **"Run workflow"** and fill in the inputs:

   | Input | Value | Description |
   |-------|-------|-------------|
   | **Branch** | `dev` | Must be `dev` (workflow will not run on other branches) |
   | **Version to tag** | e.g., `1.0` | Version number for output files and Git tag |
   | **Create a GitHub Release** | ✅ checked | Creates a GitHub Release with DOCX/PDF attached |

5. Click **"Run workflow"**.

> ⚠️ The workflow dispatch is **restricted to the `dev` branch**. Selecting any other branch will cause the job to be skipped.

### What happens automatically (both options)

1. All root-level guideline Markdown files are converted to **DOCX** and **PDF** (with the version appended to filenames).
2. A Git **tag** (e.g., `1.0`) is created and pushed (if it doesn't already exist).
3. Guideline Markdown files (those with "Guideline" in the filename) are archived into `archives/` with a version suffix (e.g., `archives/XX_Guideline_v1.0.md`). The archive is committed and pushed to the `dev` branch.
4. A **GitHub Release** is created with DOCX, PDF, and versioned Markdown files attached for download.
5. Generated files are also available as **workflow artifacts** for 30 days.

> **Important:** The archiving step matches files whose names contain "guideline" (case-insensitive). Make sure your guideline Markdown file includes "Guideline" in its filename for automatic archiving to work.

### When to trigger a release

| ✅ Trigger release | ❌ Don't trigger release |
|---|---|
| After merging a PR into `main` for a new version | While still editing on `dev` |
| When the team agrees the guideline is ready for distribution | For intermediate draft conversions |

> **Tip:** If you just want to preview generated DOCX/PDF without creating a release, use **Actions → Run workflow** on `dev` with the version field empty and the release checkbox unchecked. The files will be available as workflow artifacts.

---

## Quick Reference — Complete Release Lifecycle

```
 ┌──────────────────────────────────────────────────────────┐
 │  1. EDIT on dev branch                                   │
 │     - GitHub Web UI  or  local editor + git push         │
 │     - Edit .md file only (DOCX/PDF are auto-generated)   │
 │     - Repeat as many times as needed                     │
 └──────────────────┬───────────────────────────────────────┘
                    │
                    ▼
 ┌──────────────────────────────────────────────────────────┐
 │  2. PULL REQUEST: dev → main                             │
 │     - VS Code + Copilot  or  GitHub Web UI               │
 │     - Team reviews → merge when approved                 │
 └──────────────────┬───────────────────────────────────────┘
                    │
                    ▼
 ┌──────────────────────────────────────────────────────────┐
 │  3. RELEASE (choose one)                                 │
 │     A. git tag {version} → git push origin {version}     │
 │     B. Actions → Run workflow on dev (set version +      │
 │        check release)                                    │
 │     → Creates tag, archives .md, publishes DOCX/PDF      │
 └──────────────────────────────────────────────────────────┘
```

---

## Tips

- **Always edit the `.md` file**, never the `.docx` or `.pdf`. Generated files are not committed to the repo.
- **Images** should be placed in the `pics/` folder and referenced in Markdown as `![alt](pics/filename.png)`.
- **Naming convention:** Include "Guideline" in your guideline Markdown filename (e.g., `XX_Guideline.md`) so the workflow can automatically archive versioned copies on release.
- **Multiple guideline files** are supported — the workflow converts all root-level `.md` files (except `README.md` and `CONTRIBUTING.md`).
- **CSS styling:** Edit `style.css` to customize the appearance of generated PDF files.
- If you need to re-generate DOCX/PDF without creating a release, use **Actions → Run workflow** on the `dev` branch (leave the version field empty and the release checkbox unchecked).
- **Copilot tips:** You can ask Copilot to help with commit messages, PR descriptions, change summaries, and tagging.