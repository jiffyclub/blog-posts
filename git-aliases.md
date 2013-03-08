These are a few `git` aliases I've made recently to make my life a little
easier. The first two are for displaying the log, and the rest for merging.
See the [Git wiki][aliases] for more on how to add aliases.

# Logs

I pretty much always want the one-line log, so this is the basic version of
that:

    ls = log --oneline --decorate

`--decorate` shows any labels on commits, such as branch names.
Sometimes I want the above but with the graph view turned on:

    lg = log --oneline --decorate --graph

# Merging

[Merges][merge] in git fall into two basic categories: fast-forward merges
in which the branch label is simply updated to a new commit (but no new commit
is made), and all other merges in which a new commit is made with multiple
parents.

By default the `merge` command will attempt to do a fast-forward merge. If that
won't work it will move to some other "real" merge strategy. I think the
difference between fast-forward and the other merges is sufficient that they
shouldn't happen with the same command so I've set up aliases to separate them.
First the alias for doing a fast-forward. This will fail if a fast-forward
is not possible:

    ff = merge --ff-only

And then an alias for forcing a non-fast-forward merge even in situations where
a fast-forward would be possible:

    mrg = merge --no-ff

(This is the sort of merge [GitHub][] does when you merge a pull request.)

## Pulling

The `pull` command is really `fetch` + `merge` so it takes most of the same
options. When I do a pull I generally only want it to succeed if it's possible
to fast-forward my local branch. If git must do a real merge something is
probably wrong, so I have a fast-forward-only `pull` alias:

    ffpull = pull --ff-only

[aliases]: https://git.wiki.kernel.org/index.php/Aliases
[merge]: http://marklodato.github.com/visual-git-guide/index-en.html#merge
[GitHub]: http://github.com
