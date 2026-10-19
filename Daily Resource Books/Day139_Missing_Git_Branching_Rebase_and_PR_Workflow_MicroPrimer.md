# Day 139 — Missing Primer: Git Beyond the Basics — Branching, Rebase, Recovery, and Pull-Request Workflow

**Do this first on Day 139 (≈ 45 min, before the CI/CD pipeline), then continue with Day 139's regular plan.**
**Needed by:** Day 139 (CI runs on pull requests and protected branches), every team job, and behavioral questions ("tell me about a merge conflict you resolved").
**Why it is missing:** Day 1 teaches commit/push/branch basics; `rebase`, `cherry-pick`, `revert` vs `reset`, `reflog`, `bisect` and branching strategies appear nowhere in the repo.
**Already in place:** Git and GitHub basics (Day 1), CI/CD micro-primer and the JDK note (Day 139).

---

## 1. Merge vs rebase
- **`git merge feature`** joins histories with a merge commit; history shows exactly what happened (`--no-ff` keeps the branch visible).
- **`git rebase main`** (while on `feature`) **replays your commits on top of `main`** → a straight-line history; the commits get *new hashes*.
- **Golden rule: never rebase commits others already have** (shared/pushed branches). Rebase your own local/feature branch freely; after rebasing a pushed branch of your own use `git push --force-with-lease` (refuses if someone else pushed).
- **Interactive rebase** `git rebase -i HEAD~4` — `reword`, `squash`/`fixup`, `drop`, reorder: tidy the history *before* opening the PR.

## 2. Undoing things — pick the safe tool
| Goal | Command | Notes |
|---|---|---|
| unstage / discard working changes | `git restore --staged f` / `git restore f` | |
| undo the last commit, keep the changes | `git reset --soft HEAD~1` | local only |
| undo a commit **on a shared branch** | `git revert <sha>` | adds a new inverse commit — safe |
| throw work away | `git reset --hard <sha>` | destructive |
| copy one commit onto this branch | `git cherry-pick <sha>` | hot-fix backports |
| park work | `git stash` / `git stash pop` | |
| **find "lost" commits** | `git reflog` | the safety net: every HEAD position for ~90 days |
| find the commit that introduced a bug | `git bisect start` / `bad` / `good` | binary search over history |

## 3. Conflicts, calmly
A conflict is Git asking *you* to combine two edits. Open the file, read both sides between `<<<<<<<`, `=======`, `>>>>>>>`, keep the right result, delete the markers, `git add`, then `git rebase --continue` (or commit the merge). Run the tests afterwards — a conflict-free merge can still be wrong. Small, frequent merges make conflicts small.

## 4. Branching strategies
| | Trunk-based | GitFlow |
|---|---|---|
| Branches | short-lived (hours–days) off `main`; releases from `main` | long-lived `develop`, `release/*`, `hotfix/*` |
| Fits | CI/CD, feature flags (Day 135's file), small teams moving fast | scheduled releases, multiple supported versions |
Most modern services use **trunk-based** with feature flags.

## 5. A healthy PR workflow
Small PRs with a clear description; **protected `main`** (no direct pushes, required reviews and required CI checks — Day 139); CI on every PR; squash-merge or rebase-merge for a clean history; descriptive commit messages (`feat:`, `fix:`, imperative mood); never commit secrets or `target/` (`.gitignore`); review for correctness, tests, naming, and blast radius.

## 6. Exercise (≈ 25 min, a scratch repo)
1. Create `main` with 3 commits; branch `feature`, add 3 commits; add a conflicting commit on `main`; **rebase** `feature` onto `main`, resolve the conflict, continue.
2. `git rebase -i` to squash your 3 feature commits into one with a good message.
3. `git reset --hard HEAD~2` by "accident", then recover with `git reflog` + `git reset --hard <sha>`.
4. Introduce a bug in commit 7 of a 10-commit history and find it with `git bisect`.
5. `git revert` a commit on `main` and explain why you did not use `reset`.

## Checklist
- [ ] Can contrast merge and rebase, and state the golden rule.
- [ ] Can undo a pushed commit safely and recover a "lost" one with `reflog`.
- [ ] Can resolve a conflict and explain trunk-based vs GitFlow.
- [ ] Exercises 1–5 done. Continue with Day 139's regular plan.
