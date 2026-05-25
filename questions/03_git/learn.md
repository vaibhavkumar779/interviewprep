# Git - LEARNING MATERIAL

---

## Git Three States

```mermaid
graph LR
    WD[Working Directory] -->|git add| SA[Staging Area / Index]
    SA -->|git commit| LR[Local Repository]
    LR -->|git push| RR[Remote Repository]
    RR -->|git fetch| LR
    RR -->|git pull| WD
    LR -->|git checkout| WD
    SA -->|git restore --staged| WD
```

## Merge vs Rebase

```mermaid
gitGraph
    commit id: "A"
    commit id: "B"
    branch feature
    commit id: "C"
    commit id: "D"
    checkout main
    commit id: "E"
    checkout feature
    merge main id: "Merge Commit" tag: "git merge main"
```

```mermaid
gitGraph
    commit id: "A"
    commit id: "B"
    commit id: "E"
    commit id: "C'" tag: "rebased"
    commit id: "D'" tag: "rebased"
```

### Merge: Creates merge commit, preserves history (non-linear)
### Rebase: Replays commits on top, creates linear history (rewrites hashes)

---

## Git Advanced Operations Cheat Sheet

### Cherry-Pick
```bash
# Apply specific commit to current branch
git cherry-pick abc123

# Cherry-pick without auto-committing
git cherry-pick --no-commit abc123

# Cherry-pick range
git cherry-pick A..B    # commits after A up to B
git cherry-pick A^..B   # commits from A to B inclusive
```

### Bisect
```bash
git bisect start
git bisect bad                  # current is broken
git bisect good v1.0            # v1.0 was working
# Git checks out middle commit → you test → mark good/bad
git bisect good    # or git bisect bad
# Repeat until culprit found
git bisect reset                # return to original state

# Automated bisect
git bisect start HEAD v1.0
git bisect run ./test.sh        # auto-runs script on each commit
```

### Stash
```bash
git stash                       # save all changes
git stash save "message"        # with description
git stash -u                    # include untracked files
git stash list                  # see all stashes
git stash show stash@{0}        # see diff
git stash pop                   # apply latest + remove
git stash apply stash@{2}       # apply specific, keep in list
git stash drop stash@{0}        # delete specific
git stash clear                 # delete all
git stash branch new-branch     # create branch from stash
```

### Reset vs Revert
```mermaid
graph TD
    subgraph Reset ["git reset (rewrites history)"]
        R1["--soft: move HEAD only<br/>(changes stay staged)"]
        R2["--mixed: move HEAD + unstage<br/>(changes in working dir)"]
        R3["--hard: move HEAD + discard all<br/>(DANGEROUS - data lost)"]
    end
    subgraph Revert ["git revert (safe)"]
        V1["Creates NEW commit that<br/>undoes the target commit"]
        V2["History is preserved<br/>Safe for shared branches"]
    end
```

### Reflog
```bash
git reflog                      # show all HEAD movements
git reflog show feature-branch  # specific branch history
git checkout HEAD@{5}           # go back 5 moves
git branch recovery HEAD@{5}    # save recovered commit
```

---

## Git Hooks

```mermaid
graph LR
    subgraph ClientSide [Client-Side Hooks]
        H1[pre-commit<br/>Lint, format, tests]
        H2[commit-msg<br/>Validate message format]
        H3[pre-push<br/>Run tests before push]
        H4[prepare-commit-msg<br/>Template commit msg]
        H5[post-commit<br/>Notifications]
    end
    subgraph ServerSide [Server-Side Hooks]
        H6[pre-receive<br/>Enforce policies]
        H7[update<br/>Per-branch checks]
        H8[post-receive<br/>Trigger CI, notifications]
    end
```

---

## Branching Strategies

```mermaid
gitGraph
    commit id: "init"
    branch develop
    commit id: "dev work"
    branch feature/auth
    commit id: "auth work"
    checkout develop
    merge feature/auth
    branch release/1.0
    commit id: "bugfix"
    checkout main
    merge release/1.0 tag: "v1.0"
    checkout develop
    merge release/1.0
```
**GitFlow**: main + develop + feature/* + release/* + hotfix/*. Best for scheduled releases.

**Trunk-Based**: Everyone works on main. Short-lived branches (<1 day). Feature flags for incomplete work. Best for continuous deployment.

**GitHub Flow**: main + feature branches + PRs. Simple. Good for web apps.

---

## Gerrit Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Gerrit as Gerrit Server
    participant CI as Jenkins CI
    participant Repo as Git Repository

    Dev->>Dev: Write code + commit
    Dev->>Gerrit: git push origin HEAD:refs/for/main
    Gerrit->>Gerrit: Create Change (Change-Id)
    Gerrit->>CI: Trigger verify build
    CI->>Gerrit: Verified +1 (or -1)
    Gerrit->>Gerrit: Reviewer gives Code-Review +2
    Gerrit->>Repo: Submit (merge to main)
```

### Gerrit vs GitHub PR

| Aspect | Gerrit | GitHub PR |
|---|---|---|
| Unit of review | Single commit | Branch (multiple commits) |
| Update mechanism | Amend commit (same Change-Id) | Push new commits |
| Scoring | +1/+2 for approve, -1/-2 for reject | Approve/Request Changes |
| Merge style | Rebase preferred | Merge/Squash/Rebase options |
| Push target | `refs/for/<branch>` | Regular branch + create PR |

---

## Google Repo

```mermaid
graph TD
    M[manifest.xml] -->|defines| R1[Repo: firmware]
    M -->|defines| R2[Repo: userspace]
    M -->|defines| R3[Repo: tools]
    M -->|defines| R4[Repo: kernel]

    CMD1[repo init -u manifest-url] --> SYNC[repo sync]
    SYNC --> R1
    SYNC --> R2
    SYNC --> R3
    SYNC --> R4

    CMD2[repo forall -c 'git status'] -->|runs on| R1
    CMD2 -->|runs on| R2
    CMD2 -->|runs on| R3
    CMD2 -->|runs on| R4
```
