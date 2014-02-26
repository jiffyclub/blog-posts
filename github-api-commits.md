For fun I've been learning a bit about the [GitHub API][].
Using the API it's possible to do just about everything you can
do on GitHub itself, from commenting on PRs to adding commits to a repo.
Here I'm going to show how to do add commits to a repo on GitHub.
A notebook demonstrating things with code is available [here][notebook],
but you may want to read this post first for the high level view.

# Choosing a Client Library

The GitHub API is an [HTTP interface][http-verbs] so you can talk to
it via any tool that speaks HTTP, including things like [curl][].
To make programming with the API simpler there are
[a number of libraries][api-libs] that allow communication with GitHub
via means native to whatever language you're using.
I'm using Python and I went with the [github3.py][] library based
on its Python 3 compatibility, active development, and good documentation.

# Making Commits

The [repository api][] is the gateway for doing anything to a repo.
In github3.py this is corresponds to the [repository module][].

## Modifying a Single File

The special case of making a commit affecting a single file is much
simpler than affecting multiple files. [Creating][create-file],
[updating][update-file], and [deleting][delete-file] a file can be done
via a single API call once you have enough information to specify what
you want done.

## Modifying Multiple Files

Making a commit affecting multiple files requires making multiple API
calls and some understanding of [Git's internal data store][git-internals].
That's because to change multiple files you have to add all the changes
to the repo one at a time *before making a commit*. The process is
outlined in full in the [API docs about Git data][git-data-docs].

I should note that I think deleting multiple files in a single commit requires
a slightly different procedure, one I'll cover in another post.

- - -

That's the overview, look over the [notebook][] for the code!
http://nbviewer.ipython.org/gist/jiffyclub/9235955

[GitHub API]: http://developer.github.com/
[notebook]: http://nbviewer.ipython.org/gist/jiffyclub/9235955
[http-verbs]: http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[curl]: http://en.wikipedia.org/wiki/CURL
[api-libs]: http://developer.github.com/libraries/
[github3.py]: http://github3py.readthedocs.org/en/latest/
[repository api]: http://developer.github.com/v3/repos/
[repository module]: http://github3py.readthedocs.org/en/latest/repos.html
[create-file]: http://github3py.readthedocs.org/en/latest/repos.html#github3.repos.repo.Repository.create_file
[update-file]: http://github3py.readthedocs.org/en/latest/repos.html#github3.repos.contents.Contents.update
[delete-file]: http://github3py.readthedocs.org/en/latest/repos.html#github3.repos.contents.Contents.delete
[git-internals]: http://git-scm.com/book/en/Git-Internals
[git-data-docs]: http://developer.github.com/v3/git/
