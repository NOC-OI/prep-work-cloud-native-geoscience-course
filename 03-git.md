---
title: Introduction to Git and GitHub
teaching: 45
exercises: 30
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Install Git locally and verify it is available in the terminal."
- "Create a GitHub account and understand why it is useful for collaboration."
- "Use core Git commands: add, status, diff, and commit."
- "Track file changes over time with Git history."
- "Create and switch branches for feature work."
- "Merge a feature branch into main."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "How do I install and configure Git on my machine?"
- "Why do I need a GitHub account for collaborative workflows?"
- "How do add, status, diff, and commit work together?"
- "How do I inspect what changed and when?"
- "How do branches help me work safely on new changes?"
- "How do I merge branch work back into main?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## Why version control?

Version control helps us keep a reliable history of changes to files.
It allows us to:

- Recover earlier versions.
- See who changed what and when.
- Work on new ideas in branches without breaking stable work.

This lesson is a simplified path through key ideas from the Software Carpentry Git novice material.[^git_novice]
It focuses on the minimum skills learners need before collaborative coding and data workflows in later episodes.

If Git is new to you, do not worry about memorising every command immediately.
The main goal is to understand the workflow: inspect changes, stage intentionally, and commit meaningful checkpoints.

## Core concepts before commands

Before using Git commands, it helps to understand four terms:

- **Working directory**: the files you are editing right now.
- **Staging area**: the "next commit" area, where you choose exactly what to record.
- **Repository**: the full Git history and metadata (stored in the `.git` directory).
- **Commit**: a named snapshot in history, including author, date, and message.

A beginner mental model:

1. Edit in working directory.
2. Move selected changes to staging area (`git add`).
3. Save staged snapshot (`git commit`).

This two-step save process is one of Git's strengths because it lets you build clean, logical history.

## Install Git locally

Install Git for your operating system using the Carpentries setup page:

- https://carpentries.github.io/workshop-template/install_instructions/#git

After installation, verify in a terminal:

```bash
git --version
```

If this prints a version number, Git is installed correctly and available on your terminal path.

If the command is not found, the shell cannot locate Git yet. In that case, complete installation first, then open a new terminal and run the check again.

Set your identity once:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Check the values:

```bash
git config --global --list
```

These identity settings are written into commit metadata so teammates can see authorship clearly.
Use your real name and an email address you want associated with your commits.

If you want to inspect one value only:

```bash
git config --global user.name
git config --global user.email
```

## Create a GitHub account

Create an account at:

- https://github.com

Recommended setup:

- Verify your email.
- Enable two-factor authentication.
- Add a profile name that collaborators can recognise.

For this episode, a GitHub account is mainly needed so learners are ready for remote collaboration in the "Cloud-Native Architectures and Modern Data Formats for Geoscience" course.
Git itself works locally without GitHub, but GitHub is the most common place to share repositories, open pull requests, and review changes.

In other words:

- **Git** tracks versions on your machine.
- **GitHub** is a hosting and collaboration platform for Git repositories.

## Start a repository and track changes

Create a project folder and initialise Git:

```bash
mkdir git-practice
cd git-practice
git init
```

`git init` creates a hidden `.git` directory where Git stores project history and metadata.

At this point, your repository has no commits yet. You can verify that with:

```bash
git log
```

which will report that no commits exist.

Create a file and check status:

```bash
echo "Sea surface temperature notes" > notes.txt
git status
```

`git status` is your main safety command. It tells you what has changed, what is staged, and what is still untracked.

For beginners, running `git status` frequently is the best way to avoid confusion.
If you are unsure what state the repository is in, run `git status` first.

Stage and commit:

```bash
git add notes.txt
git commit -m "Add initial notes file"
```

`git add` places content in the staging area, and `git commit` records that staged snapshot in history.

Important detail: `git commit` only records what is staged.
If you edited a file but did not `git add` it, that edit will not be included in the commit.

Check history:

```bash
git log --oneline
```

Each commit gets a unique identifier and message, forming a timeline of your project.

The timeline helps with reproducibility: you can always return to a specific commit and understand what changed.

## Understand add, status, diff, and commit

These four commands work together as a cycle:

- Edit files in your working directory.
- Inspect changes with `status` and `diff`.
- Select what to include with `add`.
- Save a logical checkpoint with `commit`.

This helps avoid accidental commits and makes history easier to read.

Think of staging as preparing slides for a presentation: you choose what belongs in this specific story.

Edit the file and inspect changes:

```bash
echo "Added a second line" >> notes.txt
git status
git diff
```

`git diff` shows unstaged edits line by line, so you can review before staging.

In `git diff` output:

- Lines with `-` were removed.
- Lines with `+` were added.
- Context lines are shown to help you locate the change.

Stage then inspect staged changes:

```bash
git add notes.txt
git diff --staged
```

`git diff --staged` confirms exactly what will be committed next.

This is especially useful before large commits, when it is easy to accidentally stage too much.

Commit staged work:

```bash
git commit -m "Update notes with second line"
```

Good commit messages are short, specific, and action-oriented.

Useful message pattern:

- Start with a verb: `Add`, `Update`, `Fix`, `Refactor`.
- Describe what changed, not what you did while changing it.
- Keep the first line concise.

Working pattern:

1. Edit files.
2. Check `git status`.
3. Review with `git diff`.
4. Stage with `git add`.
5. Commit with a clear message.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1: First tracked change

1. Create a file `workflow.txt`.
2. Add one line describing your workflow.
3. Stage and commit the file.
4. Confirm the commit appears in the log.

::::::::::::::: solution

```bash
echo "Use status, diff, add, commit" > workflow.txt
git add workflow.txt
git commit -m "Add workflow summary"
git log --oneline
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Track changes over time

Tracking changes is one of Git's biggest advantages for research and engineering work. You can always inspect how a file evolved and recover context for decisions.

This is valuable in scientific work, where you may need to explain how a figure, table, or derived dataset was produced weeks later.

Show what changed between working tree and last commit:

```bash
git diff
```

Show history compactly:

```bash
git log --oneline --decorate
```

`--decorate` shows branch and HEAD labels, helping you see where you are in history.

Show a commit and its patch:

```bash
git show
```

By default, `git show` displays the most recent commit and its exact patch.
You can also show a specific commit with:

```bash
git show <commit-id>
```

This is the core of change tracking: you can inspect both timeline and content deltas.

## Create and use branches

A branch is an independent line of development.
Using branches means you can experiment safely without destabilising `main`.

In collaborative projects, a common pattern is one branch per feature or fix.
This keeps `main` stable and reviewable.

Create a feature branch and switch to it:

```bash
git switch -c improve-notes
```

This creates the branch and moves you to it in one step.

You can list branches at any time with:

```bash
git branch
```

The current branch is marked with `*`.

Make a change and commit it:

```bash
echo "Branch-specific update" >> notes.txt
git add notes.txt
git commit -m "Add branch-specific update"
```

Switch back to main:

```bash
git switch main
```

At this point, `main` does not yet include branch-only commits until you merge.

This separation is intentional: it allows testing and review before integration.

## Merge a branch into main

Merging combines histories. In simple cases Git performs a fast-forward merge automatically; if the same lines changed in both branches, you may need to resolve a conflict manually.

If conflicts occur, Git will pause the merge and mark affected files.
Typical workflow is:

1. Open conflicted files and edit to the final desired version.
2. Stage resolved files with `git add`.
3. Complete merge with `git commit` (if required).

Merge your feature branch:

```bash
git merge improve-notes
```

Verify history now includes merged work:

```bash
git log --oneline --decorate --graph
```

If the project uses `master` instead of `main`, replace `main` in the commands accordingly.

Many modern repositories default to `main`, but older repositories may still use `master`.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2: Branch and merge

1. Create a new branch `add-reference`.
2. Add one new line to `notes.txt` and commit.
3. Switch back to `main`.
4. Merge `add-reference` into `main`.
5. Confirm the merged history with `git log --oneline --graph`.

::::::::::::::: solution

```bash
git switch -c add-reference
echo "Include data file reference" >> notes.txt
git add notes.txt
git commit -m "Add reference note"
git switch main
git merge add-reference
git log --oneline --graph
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Install Git locally and configure your identity before starting a project."
- "Create a GitHub account so you are ready for collaboration and remote repositories."
- "`git status`, `git diff`, `git add`, and `git commit` form the core local workflow."
- "Git history (`git log`, `git show`) lets you track what changed and when."
- "Branches isolate work; merging brings reviewed changes back into `main`."
- "The staging area is central to Git: it lets you build clean, intentional commits."
- "Frequent `git status` checks help you stay oriented and avoid mistakes."

::::::::::::::::::::::::::::::::::::::::::::::::::

[^git_novice]: Software Carpentry. Version Control with Git. https://swcarpentry.github.io/git-novice/
