Manage the git repository for this project.

Do the following steps in order:

1. Run `git status` to see untracked and modified files
2. Run `git diff` to review all changes
3. Run `git log --oneline -10` to see recent commit history and follow the existing commit message style
4. Stage relevant files (avoid secrets like `.env`)
5. Commit with a concise message describing the *why*, not the *what*
6. Report what was committed and ask if you should push to remote

## Common GitHub Commands Reference

### Setup
```bash
git init
git remote add origin https://github.com/<user>/<repo>.git
git remote -v                          # verify remote
```

### Daily workflow
```bash
git status                             # check working tree
git diff                               # unstaged changes
git diff --staged                      # staged changes
git add <file>                         # stage specific file
git add -p                             # interactively stage hunks
git commit -m "message"
git push origin <branch>
git pull origin <branch>
```

### Branching
```bash
git checkout -b <branch>               # create and switch
git branch                             # list local branches
git branch -d <branch>                 # delete branch
git merge <branch>                     # merge into current
git rebase <branch>                    # rebase onto branch
```

### Inspection
```bash
git log --oneline -20                  # recent commits
git log --oneline --graph --all        # visual branch graph
git show <commit>                      # show commit details
git blame <file>                       # line-by-line authorship
git diff <branch1>..<branch2>          # diff between branches
```

### Stash
```bash
git stash                              # stash uncommitted changes
git stash pop                          # restore latest stash
git stash list                         # list all stashes
```

### Undo
```bash
git restore <file>                     # discard unstaged changes
git restore --staged <file>            # unstage a file
git revert <commit>                    # safe undo (creates new commit)
git reset --hard <commit>              # destructive: discard commits
```

### GitHub CLI (gh)
```bash
gh repo create                         # create new repo
gh pr create                           # open a pull request
gh pr list                             # list open PRs
gh pr checkout <number>                # check out a PR locally
gh issue list                          # list issues
gh issue create                        # create an issue
gh run list                            # list CI runs
gh run view <id>                       # view CI run details
```
