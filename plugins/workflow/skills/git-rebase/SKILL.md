---
name: git-rebase
description: Use when committing changes to code already committed in the current branch, or when preparing commits for PR review. Creates clean, reviewable commits using --fixup and interactive rebase.
---

# Git Rebase for Clean Commits

## Commit Strategy

| Scenario | Action |
|----------|--------|
| New feature/logical change | `git commit -m "feat: add X"` |
| Editing code from previous commit in this branch | `git commit --fixup=<commit-hash>` |

**Finding the right commit to fixup:**

```bash
git log --oneline                    # List commits with hashes
git show -s --format=%s <commit>     # View commit message to confirm
```

## Squash Workflow

**1. Sync with remote:**
```bash
git fetch
git status
# If behind: git pull
```

**2. Squash fixups into clean commits:**
```bash
git rebase -i --autosquash <first-commit-in-branch>~
```

**3. Rebase onto main:**
```bash
git fetch
git rebase -i origin/main
```

Resolve conflicts: `git add <files>` then `git rebase --continue`.
If stuck: `git rebase --abort`.

## Verify Before Push

```bash
git diff @{u}..HEAD
```

Should show only your intended changes. If wrong:
```bash
git reset --hard @{u}  # Reset to remote, start over
```

If correct:
```bash
git push -f
```

## Non-Interactive Rebase

When automating rebase (e.g., in scripts or Claude Code), use `GIT_SEQUENCE_EDITOR` to provide the rebase plan non-interactively.

**1. Create a rebase script:**

```bash
#!/bin/bash
cat > "$1" << 'EOF'
pick abc1234 First commit message
fixup def5678 Second commit (squash into first)
pick ghi9012 Third commit message
fixup jkl3456 Fourth commit (squash into third)
EOF
```

**2. Execute non-interactive rebase:**

```bash
chmod +x /tmp/rebase-script.sh
GIT_SEQUENCE_EDITOR=/tmp/rebase-script.sh git rebase -i --root
```

**Rebase commands:**
| Command | Effect |
|---------|--------|
| `pick` | Keep commit as-is |
| `fixup` | Squash into previous, discard message |
| `squash` | Squash into previous, combine messages |
| `reword` | Keep commit, edit message |
| `drop` | Remove commit |

## When Rebase Won't Work

Rebase fails with conflicts when **reordering** commits that have dependencies:

```
# This will conflict if commit B modifies files created by commit C
pick A
fixup C  # Was originally after B
pick B   # Creates files that C modifies
```

**Solution:** Use the soft reset approach in `/cleanup-commits` command, which avoids conflicts by removing all commits and recommitting in logical groups.

## Merge to Main

After PR approval, merge with a merge commit to preserve branch history:

```bash
git checkout main
git pull
git merge --no-ff <your-branch-name>
git push
```

The PR should appear as merged. History in main must be linear (no branched paths) with all distinct commits plus the merge commit visible.
