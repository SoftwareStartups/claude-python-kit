---
name: cleanup-commits
description: Interactive command to cleanup commits by grouping and squashing them using rebase
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
  - TodoWrite
---

# Cleanup Commits Command

This command guides you through cleaning up commits in the current branch by grouping them logically and using interactive rebase to squash/fixup.

## Workflow

### Step 1: Fetch and Show Recent Commits

First, fetch the latest from origin and show recent commits for selection:

```bash
git fetch origin
git log --oneline -20
```

### Step 2: Present Commit Selection

Use `AskUserQuestion` to let the user select a starting commit. Present the commits as options with their hash and message. The selected commit will be the starting point for the rebase (commits from this point forward will be cleaned up).

Example question format:

- Header: "Start from"
- Question: "Select the starting commit for cleanup (all commits after this will be grouped and squashed):"
- Options: List 4-5 recent commits with format "abc1234 - commit message"

### Step 3: Analyze and Group Commits

After user selects the starting commit:

1. Get all commits from the starting point to HEAD:

   ```bash
   git log --oneline <starting-commit>..HEAD
   ```

2. Analyze the commits and group them logically by:

   - Related file changes (use `git show --stat <commit>` to see files)
   - Feature area (frontend, backend, tests, docs, config)
   - Semantic meaning (feat, fix, refactor, chore)

3. Propose logical groupings with suggested final commit messages.

### Step 4: Present Grouping for Review

Use `AskUserQuestion` to show the proposed groupings and let the user:

- Approve the groupings
- Request modifications
- See alternative grouping suggestions

Format the groupings clearly:

```text
Group 1: "feat: add user authentication"
  - abc1234 - add login form
  - def5678 - add auth service
  - ghi9012 - add session handling

Group 2: "fix: resolve navigation bugs"
  - jkl3456 - fix back button
  - mno7890 - fix menu collapse
```

### Step 5: Execute Cleanup

Once the user approves the grouping:

**Decision Tree:**

```
Need reordering? ──yes──> Soft Reset (Phase B)
       │
       no
       │
       v
Try Rebase (Phase A) ──conflicts──> Soft Reset (Phase B)
       │
       no conflicts
       │
       v
    Success!
```

**1. Create a backup branch:**

```bash
git branch backup-before-cleanup
```

**2. Determine approach:**

- **Same-order grouping** (commits stay in original order, just squash adjacent ones) → Phase A
- **Reordering needed** (commits need to be regrouped non-adjacently) → Phase B

#### Phase A: Non-Interactive Rebase (preferred)

Reference the `git-rebase` skill for details on non-interactive rebase.

1. Create a rebase script that writes the plan:

   ```bash
   #!/bin/bash
   cat > "$1" << 'EOF'
   pick abc1234 First group's first commit
   fixup def5678 First group's second commit
   pick ghi9012 Second group's first commit
   fixup jkl3456 Second group's second commit
   EOF
   ```

2. Execute non-interactive rebase:

   ```bash
   chmod +x /tmp/rebase-script.sh
   GIT_SEQUENCE_EDITOR=/tmp/rebase-script.sh git rebase -i --root
   ```

3. If conflicts occur:

   ```bash
   git rebase --abort
   ```

   Then proceed to Phase B.

#### Phase B: Soft Reset (fallback for reordering)

Use this when reordering is needed or rebase fails with conflicts.

**How it works:** Soft reset removes all commits but keeps files staged. New commits **replace** the old ones on the branch (old commits only exist on the backup branch).

1. Remove all commits (keeps files staged):

   ```bash
   git update-ref -d HEAD
   ```

2. Unstage all files:

   ```bash
   git reset
   ```

3. For each group, stage and commit the relevant files:

   ```bash
   git add <files-for-group-1>
   git commit -m "feat: group 1 message"

   git add <files-for-group-2>
   git commit -m "feat: group 2 message"
   # ... repeat for each group
   ```

**3. Rebase onto origin/main (if not already on main):**

```bash
git fetch
git rebase origin/main
```

**4. Show the result:**

```bash
git log --oneline origin/main..HEAD
```

### Step 6: Final Review

Present the cleaned up commit history and ask user to confirm before any push:

- Show `git log --oneline` of the new commits
- Show `git diff origin/main..HEAD --stat` for a summary of changes
- Remind user they can reset with: `git reset --hard backup-before-cleanup`

## Error Handling

- If rebase conflicts occur, guide the user through resolution
- Always create a backup branch before rebasing
- If anything goes wrong: `git rebase --abort` and `git checkout backup-before-cleanup`

## Safety Checks

Before starting:

- Verify working directory is clean: `git status --porcelain`
- Ensure we're not on main/master branch
- Check if there are unpushed commits to preserve
