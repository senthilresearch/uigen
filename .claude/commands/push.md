Commit all changes and push to origin master.

Do the following steps in order:

1. Run `git status` to see untracked and modified files
2. Run `git diff` to review all changes
3. Run `git log --oneline -10` to see recent commit history and follow the existing commit message style
4. Stage all relevant files (avoid secrets like `.env`)
5. Commit with a concise message describing the *why*, not the *what*
6. Run `git log origin/master..HEAD --oneline` to show commits that will be pushed
7. Run `git push origin master`
8. Report the result
