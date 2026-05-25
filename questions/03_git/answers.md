# Git - ANSWERS

---

## Basics

**1.** Git = distributed VCS. Every developer has full repo copy. SVN/CVS = centralized (single server). Git is faster, works offline, better branching.

**2.** Distributed = every clone is a full repo with complete history. Can work offline, commit locally, then push.

**3.** Working Directory (modified files) → Staging Area/Index (`git add`) → Repository (`git commit`).

**4.** `.git/` contains: `HEAD` (current branch pointer), `objects/` (all content), `refs/` (branches/tags), `config` (repo settings), `hooks/`, `index` (staging area).

**5.** Commit = snapshot of staged changes. Stores: tree hash (file state), parent commit hash, author, committer, timestamp, message.

**6.** SHA-1 = 40-char hash identifying every object (commit, tree, blob). Ensures integrity — any change produces different hash.

**7.** HEAD = pointer to current branch/commit. Detached HEAD = HEAD points to a commit directly (not a branch). Happens with `git checkout <commit-hash>`.

**8.** `fetch` = download remote changes to local repo (doesn't modify working directory). `pull` = fetch + merge (or rebase with `--rebase`).

**9.** `git clone --depth 1` = shallow clone with only latest commit. Use in CI for faster checkout when history isn't needed.

**10.** `git rm` = remove from both working directory AND staging area. `rm` only removes from filesystem, Git still tracks it.

**11-20: Branching & Merging**

**11.** Branch = movable pointer to a commit. Internally just a file in `.git/refs/heads/` containing a commit hash. Lightweight (40 bytes).

**12.** Merge = combines branches with a merge commit (preserves history). Rebase = replays commits on top of target (linear history, rewrites commit hashes).

**13.** Merge for: shared branches, preserving history, team collaboration. Rebase for: cleaning up local commits before pushing, keeping linear history on feature branches.

**14.** Fast-forward = target branch has no new commits since branch point. Git just moves the pointer forward (no merge commit).

**15.** 3-way merge = when both branches have new commits. Uses 3 snapshots: common ancestor, tip of current, tip of target. Creates merge commit.

**16.** Conflict = same lines changed in both branches. Resolve: open conflicted files, choose changes, remove conflict markers, `git add`, `git commit`.

**17.** `<<<<<<<` = your changes, `=======` = separator, `>>>>>>>` = incoming changes.

**18.** `--no-ff` = force merge commit even when fast-forward is possible. Preserves the fact that a feature branch existed.

**19.** `--squash` = combine all branch commits into one, stage them, but don't commit. You create a single clean commit.

**20.** `git merge --abort` — returns to pre-merge state.

---

## Advanced Operations

**1-7: Cherry-Pick**
```bash
git cherry-pick abc123        # apply single commit
git cherry-pick A..B          # range (exclusive A)
git cherry-pick A^..B         # range (inclusive A)
git cherry-pick --no-commit   # stage but don't commit
git cherry-pick --abort       # cancel on conflict
```
Use cases: hotfix backport, wrong branch commit, selective feature porting.

**8-12: Bisect**
Binary search through commits. 1000 commits → ~10 steps (log2(1000)). Automated: `git bisect run ./test.sh` (script exits 0 = good, non-0 = bad).

**13-19: Stash**
Stashes stored in `.git/refs/stash`. `pop` = apply + delete. `apply` = apply + keep. `git stash -u` includes untracked. Stashes are per-repo, available on all branches.

**20-26: Reset/Revert/Reflog**

| Command | Effect | Safe for shared branches? |
|---|---|---|
| `reset --soft HEAD~1` | Move HEAD back, keep staged | NO |
| `reset --mixed HEAD~1` | Move HEAD back, unstage | NO |
| `reset --hard HEAD~1` | Move HEAD back, discard all | NO |
| `revert HEAD` | New commit undoing changes | YES |

Reflog recovery: `git reflog` → find lost commit hash → `git checkout <hash>` → `git branch recovery`

**27-34: Hooks**
Client-side hooks (`.git/hooks/`):
```bash
# pre-commit: lint check
#!/bin/bash
if ! flake8 .; then echo "Lint failed"; exit 1; fi

# commit-msg: enforce format
#!/bin/bash
if ! grep -qE '^[A-Z]+-[0-9]+: .+' "$1"; then
    echo "Commit msg must match 'JIRA-123: description'"
    exit 1
fi
```
Share hooks: Use Husky (npm), pre-commit framework (Python), or `.githooks/` directory with `core.hooksPath`.

**35-39: Submodules**
```bash
git submodule add https://github.com/lib/repo.git libs/repo
git submodule update --init --recursive
```
Downsides: complex workflow, detached HEAD in submodule, easy to get out of sync. Subtree: `git subtree add --prefix=libs/repo https://github.com/lib/repo.git main` — copies code, simpler but no upstream tracking.

**40-48: Advanced Commands**
- Interactive rebase: `git rebase -i HEAD~5` — reorder, squash, edit, drop commits.
- `git blame file.py` — show who last modified each line.
- `git log --oneline --graph --all` — visual branch history.
- `git worktree add ../feature-tree feature-branch` — multiple working directories from one repo.

---

## Workflows, Gerrit, Google Repo

**16-25: Gerrit**
- Push: `git push origin HEAD:refs/for/main` (NOT to main directly)
- Change-Id: Footer in commit message (generated by hook). Allows amending without creating new review.
- Score: Code-Review (+2 = approve, +1 = looks good to me, -1 = needs work, -2 = do not submit). Verified (+1 = CI passed, -1 = CI failed).
- Amend: `git commit --amend` (same Change-Id) updates the SAME review. In GitHub, you push new commits.
- Submit: After +2 Code-Review and +1 Verified, reviewer clicks Submit to merge.

**26-35: Google Repo**
```xml
<!-- manifest.xml -->
<manifest>
  <remote name="origin" fetch="https://git.ciena.com" />
  <default remote="origin" revision="main" sync-j="4" />
  <project name="on-firmware" path="src/firmware" />
  <project name="on-userspace" path="src/userspace" />
  <project name="on-tools" path="tools" groups="tools" />
</manifest>
```
Commands:
```bash
repo init -u https://git.ciena.com/manifest.git -b main
repo sync -j8                    # download all repos
repo forall -c 'git status'     # run on all repos
repo start feature-xyz --all    # create branch in all repos
repo upload                     # push to Gerrit for review
```
Ciena ON team uses it because: optical networking SW spans many repos (kernel, firmware, userspace, drivers, management, tools). Repo manages them as one project.
