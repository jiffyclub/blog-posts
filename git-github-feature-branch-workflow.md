When I started using [Git][] a couple years ago the feature I really fell in
love with was the easy [branching][]. Branches are awesome and you should
use them no matter what VCS you use, but the fact that Git is really designed
around branching is a good reason to use Git. In this post I'll tell you about
how to use branches in Git and with [GitHub][], but first let's talk about why
branches are so great.

# Branches

Let's say you're already using version control, because you know you
[should really be using version control][final.doc], right? Branches are a way
to make isolated, VCS managed copies of your version controlled files so that
it's possible to make changes in one copy without disturbing the other.
Why is that valuable? There are many reasons:

* You can always have a copy of your project where it's in its most recent
    working state.
* You can work on multiple bug fixes or enhancements at the same time and have
    each set of changes only in its own branch.
* You can make experimental branches and simply delete them if the changes
    don't work out.
* If you collaborate with others branches allow you to work isolated from
    other peoples' changes while still giving you access to them.

For all those reasons (and more to come) I use branches all the time, even when
I use [Subversion][] at work. And it pays off: I've had some code in a branch
for months because we don't have time to merge that particular change and deal
with its effects, but I can just leave it there and get on with other things.

# Branch Development Workflow



[Git]: http://git-scm.com/
[GitHub]: http://github.com
[branching]: http://en.wikipedia.org/wiki/Branching_(revision_control)
[final.doc]: http://www.phdcomics.com/comics/archive.php?comicid=1531
[Subversion]: http://subversion.apache.org/
