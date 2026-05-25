# Git — Deep-Dive Learning Guide

---

## 1. How Git Works Internally

Git is a **content-addressable filesystem** with a VCS on top. Everything is identified by SHA-1 hashes.

```
┌─── Git Object Model ──────────────────────────────────────────┐
│                                                                │
│  ┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐    │
│  │ Commit │────►│  Tree  │────►│  Blob  │     │  Tag   │    │
│  │        │     │        │     │        │     │        │    │
│  │ SHA:a1 │     │ SHA:b2 │     │ SHA:c3 │     │ SHA:d4 │    │
│  │        │     │        │     │        │     │        │    │
│  │ tree→b2│     │ file→c3│     │ actual │     │ commit │    │
│  │parent→ │     │ dir→e5 │     │ file   │     │ →a1    │    │
│  │author  │     │        │     │content │     │ name   │    │
│  │message │     │        │     │        │     │        │    │
│  └────────┘     └────────┘     └────────┘     └────────┘    │
│                                                                │
│  Commit → points to Tree (snapshot of all files)              │
│  Tree   → points to Blobs (file contents) and sub-Trees      │
│  Blob   → raw file content (deduplicated across commits)     │
│  Tag    → named pointer to a commit (annotated)              │
└────────────────────────────────────────────────────────────────┘
```

### The Three Areas

```
┌─────────────┐    git add    ┌──────────────┐   git commit   ┌────────────┐
│  Working    │──────────────►│   Staging     │───────────────►│ Repository │
│  Directory  │               │   (Index)     │                │ (.git/)    │
│             │◄──────────────│              │◄───────────────│            │
│  Your files │   git restore │  Next commit  │   git reset    │  History   │
│  (modified) │               │  preview      │                │  (commits) │
└─────────────┘               └──────────────┘                └────────────┘
```

```bash
# Check which area files are in
git status

# Working dir → Staging
git add file.txt          # Stage specific file
git add .                 # Stage all changes
git add -p                # Interactive: stage hunks (partial file)

# Staging → Repository
git commit -m "message"
git commit --amend        # Modify last commit (REWRITES HISTORY!)

# Undo staging
git restore --staged file.txt    # Unstage (keep changes in working dir)

# Undo working dir changes
git restore file.txt             # Discard changes (DESTRUCTIVE!)
```

---

## 2. Branching — Git's Killer Feature

A branch is just a **pointer to a commit** (40-byte file). Creating a branch is instant.

```
                 main
                  │
  A ── B ── C ── D
            │
            └── E ── F
                     │
                  feature
```

```bash
git branch feature          # Create branch (pointer to current commit)
git checkout feature        # Switch to branch
git checkout -b feature     # Create + switch (shortcut)
git switch -c feature       # Modern alternative to checkout -b

git branch -d feature       # Delete (only if merged)
git branch -D feature       # Force delete (even if not merged)
git branch -a               # List all branches (local + remote)
```

### HEAD — Where You Are

```
HEAD → main → commit D      (on branch main)
HEAD → feature → commit F   (on branch feature)
HEAD → commit C              (detached HEAD — not on any branch!)
```

---

## 3. Merging Strategies

### Fast-Forward Merge

```
Before:           After git merge feature:
  main                main
  │                   │
  A ── B ── C         A ── B ── C ── D ── E
            │                             │
            └── D ── E                    (feature, can delete)
                     │
                  feature

No merge commit created — just moves the pointer forward.
Only possible when main has NO new commits since branching.
```

### Three-Way Merge

```
Before:                     After git merge feature:
  main                        main
  │                           │
  A ── B ── C ── F            A ── B ── C ── F ── M (merge commit)
            │                           │         │
            └── D ── E                  └── D ── E┘
                     │                           │
                  feature                     feature

Merge commit M has TWO parents: F and E.
Happens when both branches have new commits.
```

### Squash Merge

```
After git merge --squash feature:
  main
  │
  A ── B ── C ── F ── S  (single commit with all feature changes)

Feature branch history (D, E) is collapsed into one commit.
Clean history but loses individual commit details.
```

---

## 4. Rebasing

Rebase **replays your commits** on top of another branch — creates new commits with new SHAs.

```
Before:                          After git rebase main (on feature):
  main                             main
  │                                │
  A ── B ── C ── F                 A ── B ── C ── F
            │                                     │
            └── D ── E                            └── D' ── E'
                     │                                       │
                  feature                                 feature

D' and E' are NEW commits (different SHAs) with same changes.
Now feature is based on latest main — fast-forward merge possible.
```

### Merge vs Rebase

| Aspect | Merge | Rebase |
|--------|-------|--------|
| History | Non-linear (merge commits) | Linear (clean line) |
| Safety | Safe (preserves history) | Rewrites history (new SHAs!) |
| Conflict resolution | Once (at merge) | Per commit (during replay) |
| Use for | Shared/public branches | Local/feature branches |
| Golden rule | — | **NEVER rebase shared/pushed branches** |

### Interactive Rebase (cleaning up before merge)

```bash
git rebase -i HEAD~4     # Edit last 4 commits

# Commands:
pick   abc1234 Add login page          # keep as-is
squash def5678 Fix typo in login       # merge into previous
reword ghi9012 WIP stuff               # change commit message
fixup  jkl3456 Another fix             # merge, discard message
drop   mno7890 Debug commit            # remove entirely
edit   pqr1234 Big commit              # pause to split
```

---

## 5. Git Reset vs Revert vs Restore

```
┌─── git reset ────────────────────────────────────────────────┐
│  Moves branch pointer backward — REWRITES HISTORY            │
│                                                               │
│  --soft   Move HEAD, keep staging + working dir               │
│  --mixed  Move HEAD, reset staging, keep working dir (default)│
│  --hard   Move HEAD, reset staging AND working dir (DANGER!)  │
│                                                               │
│  A ── B ── C ── D    →    A ── B ── C   (D is gone!)        │
│                │                    │                          │
│              HEAD                 HEAD                        │
│                                                               │
│  Use for: undoing LOCAL commits (not pushed)                 │
└───────────────────────────────────────────────────────────────┘

┌─── git revert ───────────────────────────────────────────────┐
│  Creates NEW commit that undoes a previous commit            │
│  SAFE for shared/pushed branches                             │
│                                                               │
│  A ── B ── C ── D ── D'   (D' undoes D's changes)           │
│                                                               │
│  Use for: undoing PUSHED commits                             │
└───────────────────────────────────────────────────────────────┘

┌─── git restore ──────────────────────────────────────────────┐
│  Restores files in working dir or staging area               │
│                                                               │
│  git restore file.txt           # discard working dir change │
│  git restore --staged file.txt  # unstage                    │
│  git restore --source=HEAD~2 file.txt  # from older commit  │
└───────────────────────────────────────────────────────────────┘
```

---

## 6. Cherry-Pick

Apply a specific commit from any branch onto current branch:

```
Before:                            After git cherry-pick E:
  main                               main
  │                                  │
  A ── B ── C                        A ── B ── C ── E'
            │                                 │
            └── D ── E ── F                   └── D ── E ── F
                          │                               │
                       feature                         feature

E' has same changes as E but different SHA (new commit).
```

```bash
git cherry-pick abc1234           # Apply single commit
git cherry-pick abc1234 def5678   # Apply multiple
git cherry-pick abc1234..def5678  # Apply range
git cherry-pick --no-commit abc   # Stage changes without committing
```

---

## 7. Stash

Save work-in-progress without committing:

```bash
git stash                    # Stash tracked changes
git stash -u                 # Include untracked files
git stash push -m "message"  # Named stash

git stash list               # List all stashes
git stash pop                # Apply latest + remove from stash
git stash apply stash@{2}    # Apply specific, keep in stash
git stash drop stash@{0}     # Delete specific stash
git stash clear              # Delete ALL stashes
```

---

## 8. Git Workflows

### GitFlow

```
main ────────────────────────────────────────────────── (releases)
  │                     │                    │
  └─ develop ───────────┴────────────────────┴──── (integration)
       │         │              │
       └─feature/login         └─feature/payment
       │                              │
       └─ release/v1.0 ──────────────►main (tag v1.0)
                                      │
                              hotfix/security ──► main + develop
```

### Trunk-Based Development

```
main ── A ── B ── C ── D ── E ── F ── G ── H ──── (always deployable)
             │              │
         feature/x      feature/y
         (short-lived,   (merge within 1-2 days)
          1-2 days max)
```

### GitHub Flow (simpler)

```
main ── A ── B ────── M ── C ──────── M ── (always deployable)
             │        ↑              ↑
             └─ feat ─┘   └─ fix ───┘
                PR + review    PR + review
```

| Workflow | Complexity | Best For |
|----------|-----------|----------|
| GitFlow | High | Versioned releases (mobile apps, libraries) |
| Trunk-Based | Low | CI/CD, microservices, frequent deploys |
| GitHub Flow | Low | Small teams, web apps |

---

## 9. Conflict Resolution

```
<<<<<<< HEAD (your changes)
function login(user) {
  return authenticateV2(user);
}
=======
function login(user) {
  return authenticate(user, { retry: true });
}
>>>>>>> feature/auth (incoming changes)
```

```bash
# During merge conflict:
git status                    # See conflicted files
# Edit files to resolve (remove markers)
git add resolved-file.txt     # Mark as resolved
git commit                    # Complete merge

# Abort merge
git merge --abort

# Use a merge tool
git mergetool
```

---

## 10. Git Hooks

```
.git/hooks/
├── pre-commit       # Before commit: lint, format, secret scan
├── commit-msg       # Validate commit message format
├── pre-push         # Before push: run tests
├── post-merge       # After merge: install deps
└── pre-receive      # Server-side: enforce policies

# Example: pre-commit (block commits with secrets)
#!/bin/bash
if git diff --cached | grep -iE '(password|secret|api.key|token)\s*=' ; then
    echo "ERROR: Potential secret detected!"
    exit 1
fi
```

Tools: **Husky** (Node), **pre-commit** (Python) for managing hooks.

---

## 11. Git Internals — Useful Commands

```bash
# Inspect objects
git cat-file -t abc1234       # Type: commit, tree, blob, tag
git cat-file -p abc1234       # Pretty-print contents
git log --graph --oneline     # Visual commit graph

# Reflog (safety net — logs ALL HEAD movements for 90 days)
git reflog                    # List all recent HEAD positions
git checkout HEAD@{5}         # Go back to where HEAD was 5 moves ago

# Bisect (find bug-introducing commit via binary search)
git bisect start
git bisect bad                # Current commit is bad
git bisect good v1.0          # v1.0 was good
# Git checks out middle commit — test, mark good/bad, repeat

# Blame (who changed each line)
git blame file.txt
git blame -L 10,20 file.txt  # Lines 10-20 only

# Clean (remove untracked files)
git clean -fd                 # Remove untracked files + dirs
git clean -fxd                # Also remove ignored files (full reset)
```

---

## 12. Remote Operations

```
┌─── Local ─────────────────────────────────────────────────────┐
│  Working Dir ──► Staging ──► Local Repo (.git/)               │
│                                    │                           │
│                              git push / git fetch              │
│                                    │                           │
└────────────────────────────────────┼───────────────────────────┘
                                     │
┌────────────────────────────────────▼───────────────────────────┐
│  Remote Repo (origin)                                          │
│  github.com/user/repo.git                                     │
└────────────────────────────────────────────────────────────────┘
```

```bash
git fetch origin              # Download remote changes (don't merge)
git pull origin main          # Fetch + merge (or rebase with --rebase)
git push origin main          # Upload local commits

git remote -v                 # List remotes
git remote add upstream URL   # Add another remote (fork workflow)

# Tracking branches
git branch -u origin/main     # Set upstream for current branch
git push -u origin feature    # Push + set upstream
```

### Fetch vs Pull

```
git fetch:  origin/main updated, your main unchanged
            (safe — inspect before merging)

git pull:   git fetch + git merge (or rebase)
            (convenient but can cause unexpected merges)

Best practice: git fetch → inspect → git merge/rebase
```

---

## 13. Tags

```bash
# Lightweight tag (just a pointer)
git tag v1.0

# Annotated tag (recommended — has metadata)
git tag -a v1.0 -m "Release 1.0"

# Push tags
git push origin v1.0          # Single tag
git push origin --tags         # All tags

# List & delete
git tag -l "v1.*"              # List matching
git tag -d v1.0                # Delete local
git push origin :refs/tags/v1.0  # Delete remote
```
