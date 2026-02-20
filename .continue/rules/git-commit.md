---
name: Git Commit Workspace
description: Identify changed files and main changes, generate a commit message, commit, and push to the current branch remote automatically (no user confirmation).
allowUserExecution: true
allowedTools:
  - shell
  - bash
---

# Git Auto Commit Push (No Confirmation)

Perform an automatic Git workflow with no user confirmation:
1) identify changed files and main changes,
2) generate a commit message,
3) commit,
4) push.

## Workflow

### 1) Identify changed files and main changes

- Ensure we are inside a Git repository and there are changes:
```bash
git rev-parse --is-inside-work-tree
git status --porcelain=v1
````

* Abort if merge or rebase is in progress:

```bash
test -e .git/MERGE_HEAD && { echo "Merge in progress; aborting."; exit 1; }
test -d .git/rebase-merge && { echo "Rebase in progress; aborting."; exit 1; }
test -d .git/rebase-apply && { echo "Rebase in progress; aborting."; exit 1; }
```

* Collect changed files and summarize:

```bash
git status --short
git diff --stat
git diff --name-status
git ls-files --others --exclude-standard
```

* Read diffs for meaningful changes (enough to write an accurate message):

```bash
git diff
```

### 2) Generate commit message from identified changes

* Produce a single, accurate commit subject based on the diff intent.
* Prefer Conventional Commit format:

```text
<type>(<scope>): <summary>
```

* Choose type: `fix` | `feat` | `refactor` | `docs` | `test` | `chore` | `ci` | `build`
* Keep subject imperative and <= 72 characters.
* If needed, add 1–3 short body lines describing impact.

### 3) Commit (stage all changes)

```bash
git add -A
git diff --cached --stat
git commit -m "<subject>" $( [ -n "<body>" ] && printf '%s' "-m \"<body line 1>\" -m \"<body line 2>\"" )
```

### 4) Push (no confirmation)

```bash
branch="$(git branch --show-current)"
git rev-parse --abbrev-ref --symbolic-full-name '@{u}' >/dev/null 2>&1 \
  && git push \
  || git push -u origin "$branch"
```

## Output

Return:

```bash
git log -1 --oneline
git status --short
```

## Operational Rules

* Do not use `--no-verify` unless explicitly requested by the user.
* Do not discard/reset changes.
* If push is rejected, print the error and stop.
