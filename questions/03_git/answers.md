# Git — COMPREHENSIVE ANSWERS

---

## Basics

**1. Git? Distributed VCS?**

```
Centralized (SVN):                  Distributed (Git):
┌──────────────────┐              ┌──────────────────┐
│  Central Server   │              │  Remote (GitHub)  │
│  (single copy)    │              │  (shared repo)    │
│                    │              └────────┬─────────┘
│  Every operation   │                       │ push/pull
│  needs server      │              ┌────────┼─────────┐
│                    │              ▼        ▼         ▼
└────────┬─────────┘        ┌────────┐ ┌────────┐ ┌────────┐
         │                    │ Dev A  │ │ Dev B  │ │ Dev C  │
    ┌────┼────┐              │ Full   │ │ Full   │ │ Full   │
    ▼    ▼    ▼              │ Clone  │ │ Clone  │ │ Clone  │
  Dev  Dev  Dev              │ (all   │ │ (all   │ │ (all   │
  (no local               │ history)│ │ history)│ │ history)│
   history)                  └────────┘ └────────┘ └────────┘

Distributed benefits:
  ✅ Work offline (commit, branch, log)
  ✅ Every clone is a full backup
  ✅ Faster operations (local)
  ✅ Better branching model
```

---

**2. Git three areas (working tree, staging, repository):**

```
Working Directory          Staging Area (Index)        Repository (.git)
(your files)               (next commit preview)       (committed history)
┌──────────────────┐      ┌──────────────────┐       ┌──────────────────┐
│                    │      │                    │       │                    │
│  Modified files    │ ──►  │  Staged changes    │  ──►  │  Committed        │
│                    │      │                    │       │  snapshots         │
│  edit file.py      │ git  │  selected changes  │ git   │                    │
│  create new.py     │ add  │  ready to commit   │ commit│  permanent history│
│  delete old.py     │      │                    │       │                    │
│                    │      │                    │       │                    │
└──────────────────┘      └──────────────────┘       └──────────────────┘

  git status    → shows what's modified/staged
  git diff      → working vs staging
  git diff --staged → staging vs last commit
```

---

**3-4. `.git/` directory structure:**

```
.git/
├── HEAD              ← Points to current branch (ref: refs/heads/main)
├── config            ← Repo-specific settings
├── description       ← GitWeb description
├── hooks/            ← Client/server-side hooks
│   ├── pre-commit
│   └── commit-msg
├── index             ← Staging area (binary file)
├── objects/          ← All data (commits, trees, blobs)
│   ├── ab/
│   │   └── c123...   ← Object files (content-addressed)
│   ├── info/
│   └── pack/         ← Packed objects (for efficiency)
├── refs/             ← Branch and tag pointers
│   ├── heads/        ← Local branches
│   │   ├── main      ← File containing commit SHA
│   │   └── feature
│   ├── tags/         ← Tags
│   └── remotes/      ← Remote tracking branches
│       └── origin/
│           └── main
└── logs/             ← Reflog (history of HEAD movements)
```

---

**5. What is a commit?**

```
A snapshot of your staged changes, NOT a diff.

Commit object contains:
  ┌──────────────────────────────────────┐
  │ commit abc123def456                   │
  │                                       │
  │ tree:    7890fed...  (file snapshot) │
  │ parent:  123abc...   (previous commit)│
  │ author:  Vaibhav <v@...> 1716...     │
  │ committer: Vaibhav <v@...> 1716...   │
  │                                       │
  │ message: "Fix login timeout bug"     │
  └──────────────────────────────────────┘

Commit chain:
  A ──► B ──► C ──► D (HEAD → main)
  Each commit points to its parent(s)
```

---

**6. SHA-1 hash?**

40-character hexadecimal hash identifying every Git object:
- Every commit, tree, blob, tag has a unique SHA
- Change one byte → completely different hash
- Ensures data integrity (tampering detection)
- Example: `a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0`
- Short form: first 7+ chars (`a1b2c3d`)

---

**7. HEAD? Detached HEAD?**

```
Normal HEAD:                         Detached HEAD:
HEAD → main → commit D              HEAD → commit B (directly)

  A ── B ── C ── D                   A ── B ── C ── D
                  ↑                        ↑
                main                     HEAD (detached!)
                  ↑                      main still at D
                HEAD

Happens when:
  git checkout abc123    (specific commit)
  git checkout v1.0      (tag)

Risk: New commits have no branch → lost when you switch!
Fix:  git checkout -b new-branch   (create branch from here)
```

---

**8. `fetch` vs `pull`?**

```
git fetch:                           git pull:
┌────────────────────────┐          ┌────────────────────────┐
│ Download remote changes │          │ fetch + merge           │
│ Update remote-tracking  │          │ (or fetch + rebase     │
│ branches                 │          │  with --rebase)         │
│                          │          │                          │
│ Does NOT modify your    │          │ MODIFIES your working  │
│ working directory        │          │ directory               │
│                          │          │                          │
│ Safe — inspect first    │          │ Can cause conflicts     │
│ git log origin/main     │          │                          │
└────────────────────────┘          └────────────────────────┘

Best practice: git fetch → review → git merge origin/main
```

---

**9-10. Shallow clone? `git rm` vs `rm`?**

```bash
git clone --depth 1 https://repo.git    # Only latest commit
# Use in CI: faster checkout, less bandwidth, history not needed

git rm file.txt     # Removes from filesystem AND stages the deletion
rm file.txt         # Only removes from filesystem (Git still tracks it)
# After rm: need git add file.txt to stage the deletion
```

---

## Branching & Merging

**11. What is a branch internally?**

```
A branch is just a 40-byte file containing a commit hash:

  .git/refs/heads/main     → contains: abc123...
  .git/refs/heads/feature  → contains: def456...

Creating a branch = creating one file (instant, O(1))
Switching branches = update HEAD + checkout files

  A ── B ── C (main)
            └── D ── E (feature)

  main    = pointer to C
  feature = pointer to E
  HEAD    = pointer to feature (current branch)
```

---

**12. Merge vs Rebase?**

```
MERGE (preserves history):

  Before:                    After merge:
  main:    A ── B ── C       A ── B ── C ── M (merge commit)
  feature:      └── D ── E            └── D ── E ──┘

  git checkout main
  git merge feature
  Creates merge commit M with TWO parents (C and E)
  History: non-linear but complete

REBASE (rewrites history):

  Before:                    After rebase:
  main:    A ── B ── C       A ── B ── C ── D' ── E'
  feature:      └── D ── E   (D' and E' are NEW commits)

  git checkout feature
  git rebase main
  Replays D, E on top of C with NEW hashes
  History: linear and clean

  ┌────────────────────────────────────────────────────────┐
  │ Golden Rule: NEVER rebase commits already pushed to    │
  │ a shared branch. It rewrites history → confuses team! │
  └────────────────────────────────────────────────────────┘
```

---

**13. When merge? When rebase?**

| Use Merge | Use Rebase |
|-----------|-----------|
| Shared branches (main, develop) | Local feature branches (before push) |
| Preserve complete history | Clean up messy local commits |
| Team collaboration | Personal branches only |
| PRs (squash merge) | Before opening PR |

---

**14-15. Fast-forward vs 3-way merge?**

```
Fast-Forward (no new commits on main):
  Before:  main: A ── B
           feature:    └── C ── D

  After:   main: A ── B ── C ── D
           (main pointer moved forward — no merge commit)

3-Way Merge (both branches have new commits):
  Before:  main:    A ── B ── C
           feature:      └── D ── E

  Git uses 3 snapshots:
    1. Common ancestor (B)
    2. Tip of main (C)
    3. Tip of feature (E)

  After:   main: A ── B ── C ── M
                       └── D ── E ──┘
  Creates merge commit M
```

---

**16-17. Merge conflicts — how to resolve:**

```
Conflict occurs when same lines changed in both branches:

<<<<<<< HEAD (your branch)
function login(user, password) {
=======
function login(username, pass) {
>>>>>>> feature-branch

Resolution:
  1. Open conflicted file
  2. Choose correct version (or combine both)
  3. Remove conflict markers (<<<<, ====, >>>>)
  4. git add resolved_file.py
  5. git commit   (or git merge --continue)

Abort: git merge --abort  (return to pre-merge state)

Tools: VS Code built-in merge editor, git mergetool
```

---

**18-19. `--no-ff` and `--squash`?**

```
--no-ff (no fast-forward):
  Forces merge commit even when FF is possible

  Without --no-ff:  A ── B ── C ── D     (linear, lost branch info)
  With --no-ff:     A ── B ── M          (merge commit preserves
                         └── C ── D ──┘    that feature branch existed)

  Best practice: use --no-ff for feature → main merges

--squash:
  Combines all branch commits into one staged change (no merge commit)

  feature: A ── B ── C ── D ── E  (5 messy commits)

  git merge --squash feature
  git commit -m "Add login feature"

  main: X ── Y ── Z ── [single clean commit]

  Use: Clean up messy feature branches in PRs
```

---

## Advanced Operations

**20. Cherry-pick:**

```bash
git cherry-pick abc123                    # Apply single commit to current branch
git cherry-pick abc123 def456            # Multiple commits
git cherry-pick abc123..def456           # Range (exclusive start)
git cherry-pick abc123^..def456          # Range (inclusive start)
git cherry-pick --no-commit abc123       # Stage but don't commit
git cherry-pick --abort                   # Cancel on conflict
```

```
Use cases:
  main:    A ── B ── C
  hotfix:       └── D ── E ── F

  Need only F on main (not D, E):
  git checkout main
  git cherry-pick F

  main:    A ── B ── C ── F'   (F' = copy of F)

  Common: backporting hotfixes, selective feature porting
```

---

**21. Git bisect (binary search for bugs):**

```
Scenario: Bug introduced somewhere in 1000 commits

git bisect start
git bisect bad                    # Current commit has bug
git bisect good v1.0              # v1.0 was bug-free

  Git checks out middle commit → you test:

  good ─────────────────────────── bad
  v1.0            ↑ test this      HEAD
                commit 500

  If good → search upper half
  If bad  → search lower half

  1000 commits → ~10 tests (log2(1000))

Automated:
  git bisect run ./test.sh
  # test.sh exits 0 = good, non-0 = bad
  # Git finds the exact bad commit automatically!

git bisect reset                  # Return to original state
```

---

**22. Stash:**

```bash
git stash                         # Save uncommitted changes
git stash -u                      # Include untracked files
git stash -m "WIP: login feature" # With message
git stash list                    # Show all stashes
git stash pop                     # Apply + remove from stash
git stash apply                   # Apply + keep in stash
git stash drop stash@{0}          # Delete specific stash
git stash clear                   # Delete ALL stashes
```

```
Use case: Working on feature, urgent bug fix needed

  1. git stash                    Save current work
  2. git checkout main            Switch to main
  3. git checkout -b hotfix       Create fix branch
  4. ... fix the bug ...
  5. git checkout feature         Go back
  6. git stash pop                Restore work
```

---

**23. Reset — soft, mixed, hard:**

```
                    Working Dir    Staging    Commit History
                    ───────────    ───────    ──────────────
git reset --soft    unchanged      unchanged  HEAD moves back
git reset --mixed   unchanged      CLEARED    HEAD moves back  (default)
git reset --hard    CLEARED        CLEARED    HEAD moves back

Example:
  A ── B ── C ── D (HEAD)

  git reset --soft HEAD~2   → HEAD at B, C+D changes staged
  git reset --mixed HEAD~2  → HEAD at B, C+D changes unstaged
  git reset --hard HEAD~2   → HEAD at B, C+D changes GONE ⚠️

  ┌──────────────────────────────────────────────────────┐
  │ NEVER use reset on pushed commits (rewrites history) │
  │ Use git revert instead (safe for shared branches)    │
  └──────────────────────────────────────────────────────┘
```

---

**24. Revert vs Reset:**

```
Reset (dangerous on shared branches):
  A ── B ── C ── D       →     A ── B
  Removes C and D from history (rewrite!)

Revert (safe):
  A ── B ── C ── D       →     A ── B ── C ── D ── D'
  Creates NEW commit D' that undoes D's changes
  History preserved, team not confused

  git revert HEAD              # Undo last commit
  git revert abc123            # Undo specific commit
  git revert HEAD~3..HEAD      # Undo last 3 commits
```

---

**25. Reflog — recover lost commits:**

```
git reflog    # Shows EVERY HEAD movement (even after reset --hard!)

  abc1234 HEAD@{0}: reset: moving to HEAD~2
  def5678 HEAD@{1}: commit: Add login feature
  789abcd HEAD@{2}: commit: Fix typo

Recovery:
  git checkout def5678             # Go to lost commit
  git branch recovered def5678    # Create branch to save it
  # Or:
  git reset --hard def5678        # Move HEAD back to it

Reflog is your safety net. Entries expire after 90 days (default).
```

---

## Git Hooks

**26. Client-side hooks:**

```
.git/hooks/
├── pre-commit        ← Before commit (lint, format, test)
├── commit-msg        ← Validate commit message format
├── pre-push          ← Before push (run tests, check branch)
├── prepare-commit-msg← Auto-generate commit message
└── post-commit       ← After commit (notify, stats)
```

```bash
# pre-commit: Run linter before allowing commit
#!/bin/bash
echo "Running linter..."
if ! flake8 .; then
    echo "❌ Lint failed. Fix issues before committing."
    exit 1
fi
echo "✅ Lint passed."

# commit-msg: Enforce commit message format
#!/bin/bash
if ! grep -qE '^(feat|fix|docs|chore|refactor|test): .+' "$1"; then
    echo "❌ Commit message must match: 'type: description'"
    echo "   Types: feat, fix, docs, chore, refactor, test"
    exit 1
fi
```

Share hooks across team:
```bash
# Option 1: .githooks/ directory + configure
git config core.hooksPath .githooks

# Option 2: pre-commit framework (Python)
pip install pre-commit
# .pre-commit-config.yaml in repo root

# Option 3: Husky (Node.js)
npx husky-init
```

---

## Submodules & Subtree

**27. Git submodules:**

```bash
git submodule add https://github.com/lib/repo.git libs/repo
git submodule update --init --recursive    # After clone, pull submodules
git submodule update --remote              # Update to latest upstream
```

```
Main repo:
├── src/
├── libs/
│   └── repo/     ← Submodule (separate Git repo, pinned to specific commit)
├── .gitmodules   ← Submodule configuration
└── README.md

Downsides:
  ❌ Complex workflow (easy to forget init/update)
  ❌ Detached HEAD in submodule
  ❌ Easy to get out of sync
  ❌ CI must explicitly init submodules
```

---

**28. Git subtree (alternative to submodules):**

```bash
git subtree add --prefix=libs/repo https://github.com/lib/repo.git main --squash
git subtree pull --prefix=libs/repo https://github.com/lib/repo.git main --squash
```

```
Subtree vs Submodule:
┌─── Submodule ──────────────────┬─── Subtree ────────────────────┐
│ Reference (pointer)             │ Full copy of code              │
│ Separate clone needed           │ Included in main clone         │
│ Complex workflow                 │ Simpler workflow               │
│ Easy to update upstream         │ Harder to push back upstream   │
│ .gitmodules config file         │ No extra config files          │
│ Better for: active upstream dev │ Better for: vendoring code     │
└─────────────────────────────────┴────────────────────────────────┘
```

---

## Advanced Commands

**29. Interactive rebase (`git rebase -i`):**

```bash
git rebase -i HEAD~5    # Edit last 5 commits

# Opens editor:
pick abc1234 Add login feature
pick def5678 Fix typo in login
pick 789abcd Add logout feature
pick 111aaa  WIP: debugging
pick 222bbb  Fix lint errors

# Change to:
pick abc1234 Add login feature
squash def5678 Fix typo in login      # Combine with above
pick 789abcd Add logout feature
drop 111aaa  WIP: debugging            # Remove this commit
fixup 222bbb  Fix lint errors          # Combine, discard message

# Commands:
# pick   = keep as-is
# squash = combine with previous (keep message)
# fixup  = combine with previous (discard message)
# reword = change commit message
# edit   = stop to amend
# drop   = remove commit
# reorder lines = reorder commits
```

---

**30. Useful Git commands:**

```bash
# Blame: who changed each line?
git blame src/app.py
# Shows: commit hash, author, date, line content per line

# Log: visual branch history
git log --oneline --graph --all --decorate

# Worktree: multiple working directories from one repo
git worktree add ../feature-worktree feature-branch
# Now work on feature without switching branches

# Show changes in a commit
git show abc1234

# Find commits that changed a file
git log --follow -- path/to/file.py

# Search commit messages
git log --grep="bug fix"

# Who made the most commits?
git shortlog -sn

# Clean untracked files
git clean -fd    # Force + directories
git clean -fdn   # Dry run first
```

---

## Workflows

**31. Git branching strategies:**

```
Trunk-Based (modern, recommended):
  main ──●──●──●──●──●──●──●──●──
          ↑  ↑  ↑  ↑  ↑  ↑
          └──┘  └──┘  └──┘
         Short-lived branches (< 1 day)
  + Feature flags for incomplete work
  + Best for continuous deployment

GitFlow (traditional):
  main    ──────────●──────────────●───
                     ↑              ↑
  release  ─────────●──────────────●───
                     ↑
  develop  ●──●──●──●──●──●──●──●──●──
            ↑  ↑  ↑
  feature  └──┘  └──
  + Clear release process
  - Complex, slow, merge conflicts

GitHub Flow (simple):
  main ──────●────────●────────●───
              ↑        ↑        ↑
  feature  ───┘     ───┘     ───┘
  Each feature branch → PR → merge to main
  + Simple, good for web apps
```

---

## Gerrit (Ciena context)

**32. Gerrit vs GitHub/Azure DevOps:**

```
GitHub/Azure DevOps:                Gerrit:
┌──────────────────────────┐      ┌──────────────────────────┐
│ Push multiple commits     │      │ Push ONE commit at a time │
│ Create Pull Request       │      │ Push to refs/for/main     │
│ Review all commits        │      │ Review individual commit  │
│ Merge PR → branch         │      │ +2 = approve, Submit     │
│                            │      │                          │
│ Review unit: PR (branch)  │      │ Review unit: single commit│
│ New commit → same PR      │      │ Amend → same Change-Id   │
│                            │      │                          │
└──────────────────────────┘      └──────────────────────────┘
```

---

**33. Gerrit workflow:**

```bash
# 1. Clone with commit-msg hook (adds Change-Id to commits)
git clone ssh://gerrit.ciena.com/project
scp -p gerrit.ciena.com:hooks/commit-msg .git/hooks/

# 2. Work on code
git add .
git commit -m "Fix network timeout in transponder module"
# commit-msg hook auto-adds:
# Change-Id: I1234567890abcdef...

# 3. Push for review (NOT to main directly!)
git push origin HEAD:refs/for/main

# 4. Reviewer scores:
# Code-Review: +2 (approve), +1 (looks good), -1 (needs work), -2 (block)
# Verified:    +1 (CI passed), -1 (CI failed)

# 5. Need changes? AMEND the same commit (same Change-Id!)
# Edit code...
git add .
git commit --amend    # Same Change-Id → updates same review
git push origin HEAD:refs/for/main

# 6. After +2 Code-Review AND +1 Verified → Submit (merge)
```

---

## Google Repo (Multi-repo management)

**34. What is Google Repo?**

```
Problem: Project spans 50+ Git repos (kernel, firmware, userspace, tools)
  How to clone them all? Keep them in sync? Branch across all?

Solution: Google Repo tool + manifest file

manifest.xml defines all repos:
┌────────────────────────────────────────────────────────┐
│ <manifest>                                              │
│   <remote name="origin" fetch="https://git.ciena.com"/>│
│   <default remote="origin" revision="main" sync-j="4"/>│
│                                                          │
│   <project name="on-firmware" path="src/firmware"/>     │
│   <project name="on-userspace" path="src/userspace"/>   │
│   <project name="on-kernel" path="src/kernel"/>         │
│   <project name="on-tools" path="tools" groups="tools"/>│
│ </manifest>                                              │
└────────────────────────────────────────────────────────┘
```

```bash
# Commands:
repo init -u https://git.ciena.com/manifest.git -b main
repo sync -j8                         # Download ALL repos
repo forall -c 'git status'           # Run command in ALL repos
repo start feature-xyz --all          # Create branch in ALL repos
repo upload                           # Push ALL changes to Gerrit
repo diff                             # Show diff across ALL repos

# Why Ciena uses it:
# Optical networking SW = kernel + firmware + drivers + management + tools
# Repo manages them as ONE project
# Yocto layer structure maps to repo projects
```

---

**35. Git commands cheat sheet:**

```bash
# ─── Basics ───
git init                              # New repo
git clone <url>                       # Clone
git status                            # What's changed
git add .                             # Stage all
git commit -m "message"               # Commit
git push origin main                  # Push

# ─── Branching ───
git branch feature                    # Create branch
git checkout -b feature               # Create + switch
git switch -c feature                 # Create + switch (modern)
git branch -d feature                 # Delete (safe)
git branch -D feature                 # Delete (force)

# ─── Merging ───
git merge feature                     # Merge into current
git merge --no-ff feature             # Force merge commit
git merge --squash feature            # Squash merge
git merge --abort                     # Cancel merge

# ─── History ───
git log --oneline --graph --all       # Visual history
git log -p                            # With diffs
git blame file.py                     # Who changed each line
git diff                              # Working vs staged
git diff --staged                     # Staged vs committed

# ─── Undo ───
git reset --soft HEAD~1               # Undo commit, keep staged
git reset --hard HEAD~1               # Undo everything ⚠️
git revert HEAD                       # Safe undo (new commit)
git stash                             # Save work temporarily
git reflog                            # Find lost commits

# ─── Remote ───
git remote -v                         # Show remotes
git fetch origin                      # Download changes
git pull --rebase origin main         # Pull + rebase
git push -u origin feature            # Push new branch
```
