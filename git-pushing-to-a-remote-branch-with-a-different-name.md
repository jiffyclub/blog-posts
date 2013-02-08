Normally when I do a push in git I do something like `git push origin master`,
which really means push from the local branch named `master` to the remote
branch named `master`. If you want to push to a remote branch with a different
name than your local branch, separate the local and remote names with a colon:
`git push origin local-name:remote-name`.
