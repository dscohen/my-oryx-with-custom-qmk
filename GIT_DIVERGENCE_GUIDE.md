# Git Divergence Guide

This guide explains what to do when your local branch and remote branch have diverged, and how to resolve it.

## What Is Branch Divergence?

Divergence occurs when:
- **Remote** has commits you don't have locally
- **Local** has commits the remote doesn't have yet

This commonly happens when:
1. Someone else pushed changes to the repository
2. You made local commits before pulling the latest changes
3. You're working on multiple machines

### How to Detect Divergence

Run:
```bash
git status
```

You'll see a message like:
```
Your branch and 'origin/main' have diverged,
and have 1 and 1 different commits each, respectively.
```

## Resolution Methods

Choose **one** of the following approaches based on your preference:

### Method 1: Merge (Recommended for Most Cases)

**Use this if:** You want to keep all commits and create a merge commit combining both branches.

```bash
git pull --no-rebase origin main
git push origin main
```

**Pros:**
- Preserves full commit history
- Non-destructive
- Easy to see what was merged

**Cons:**
- Creates a merge commit
- Slightly more complex history

---

### Method 2: Rebase (Cleaner History)

**Use this if:** You prefer a linear commit history without merge commits.

```bash
git pull --rebase origin main
git push origin main
```

**Pros:**
- Linear, cleaner commit history
- No merge commit clutter

**Cons:**
- Rewrites local commit history
- Can be tricky if there are conflicts
- Not recommended if others are working with your branch

---

### Method 3: Fast-Forward Only (Strictest)

**Use this if:** You want to fail rather than create a merge commit, forcing you to rebase manually if needed.

```bash
git pull --ff-only origin main
```

If this fails, you must rebase or merge manually.

---

## Step-by-Step: When You Can't Push

### 1. Check Your Status

```bash
git status
```

### 2. Commit Any Uncommitted Changes

If you have uncommitted changes, commit them first:

```bash
git add .
git commit -m "your commit message"
```

### 3. Pull Remote Changes

Choose your preferred method above. Most common:

```bash
git pull --no-rebase origin main
```

### 4. Resolve Any Conflicts (if they occur)

If there are merge conflicts:

```bash
# See which files have conflicts
git status

# Edit conflicted files manually
# Look for markers like:
# <<<<<<<< HEAD
# your changes
# ========
# their changes
# >>>>>>> origin/main

# After editing, stage the resolved files
git add <resolved-file>

# Complete the merge
git commit
```

### 5. Push to Remote

```bash
git push origin main
```

---

## Common Scenarios

### Scenario 1: "I made commits but the remote has new commits too"

```bash
# Pull and merge (or rebase)
git pull --no-rebase origin main

# Push your merged result
git push origin main
```

### Scenario 2: "I have uncommitted changes and divergence"

```bash
# Commit your changes
git add .
git commit -m "WIP: my changes"

# Now pull and merge
git pull --no-rebase origin main

# Push
git push origin main
```

### Scenario 3: "I want to throw away my local commits and use remote version"

**Warning: This is destructive!**

```bash
# Reset to remote state (loses all local commits!)
git reset --hard origin/main

# Verify
git status
```

### Scenario 4: "I want to undo the merge I just did"

```bash
# If you haven't pushed yet
git merge --abort

# If you already committed the merge
git reset --hard HEAD~1
```

---

## Prevention Tips

To avoid divergence in the future:

1. **Pull before starting work**
   ```bash
   git pull origin main
   ```

2. **Pull before pushing**
   ```bash
   git pull origin main
   git push origin main
   ```

3. **Set a default merge strategy** (optional)
   ```bash
   # Use merge (default)
   git config pull.rebase false

   # Or use rebase
   git config pull.rebase true

   # Or only fast-forward
   git config pull.ff only
   ```

4. **Check status frequently**
   ```bash
   git status
   ```

---

## When Things Go Wrong

### "I messed up the merge, how do I undo it?"

If you haven't pushed yet:
```bash
git merge --abort
```

If you already committed:
```bash
# This undoes the last commit (the merge)
git reset --hard HEAD~1
```

If you already pushed:
```bash
# Revert the merge commit
git revert -m 1 <merge-commit-hash>

# Or reset (destructive, avoid if others are working on the branch)
git reset --hard <commit-before-merge>
git push --force-with-lease origin main  # Use with extreme caution
```

### "I have merge conflicts and I'm confused"

```bash
# See all conflicted files
git status

# See the conflicts in a specific file
git diff <filename>

# Abort the merge and start over
git merge --abort
```

---

## Quick Reference

| Command | What It Does |
|---------|-------------|
| `git status` | See if branches have diverged |
| `git pull origin main` | Pull remote changes (uses configured strategy) |
| `git pull --no-rebase origin main` | Pull with merge |
| `git pull --rebase origin main` | Pull with rebase |
| `git push origin main` | Push local commits to remote |
| `git log --oneline -5` | See recent commits |
| `git diff origin/main..main` | See differences between remote and local |

---

## For This Repository

**Current setup:** Merge strategy (use `git pull --no-rebase origin main`)

**Typical workflow:**
```bash
# 1. Make your changes
# (edit files...)

# 2. Commit
git add .
git commit -m "description of changes"

# 3. Before pushing, pull latest
git pull --no-rebase origin main

# 4. Resolve any conflicts (if needed)
# (edit conflicted files...)
# (stage and commit)

# 5. Push
git push origin main
```

---

## Additional Resources

- [Git Documentation - Merge vs Rebase](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
- [GitHub - Resolving Merge Conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts)
- [Git Rebase vs Merge](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

---

## Still Stuck?

If you're unsure, the safest approach is always:

```bash
git pull --no-rebase origin main
git push origin main
```

This preserves your history and is easy to understand.
