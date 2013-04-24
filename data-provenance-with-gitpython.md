# Data Provenance

When running scientific software it is considered a
[best practice][best practices paper] to automatically record the versions
of all the software you use. This practice is sometimes referred to as
recording the [provenance][] of the results and helps make your analysis
more [reproducible][reproducibility]. Almost all software libraries will
have a version number that you can somehow access from your own software.
For example, [NumPy's][numpy] version number is recorded in the variable
`numpy.__version__` and most Python packages will having something similar.
Python's version is in the variable `sys.version` (and, alternatively,
`sys.version_info`).

However, a lot of personal or lab software doesn't have a version number.
The software might change so fast and be modified by so many people that
manually incrememented version numbers aren't very practical. There's still
hope in this situation, though, if the software is under [version control][].
In [Subversion][] the [keyword properties][] feature is often used to
[record provenance][swc svn provenance]. There isn't a compatible feature
in [Git][], but for Python software in Git repositories we can engineer a
provenance solution using the [GitPython][] package.

# Returning to Previous States with Git

When you [make a commit][git commit] in Git the state of the repository is
recorded and given a label based on a [hash][] of the commit data. We can use
the [commit hash][] to return to any recorded state of the repository using
the "[git checkout][]" command. This means that if you know the commit hash
of your software when you created a certain set of results, you can always set
your software back to that state to reproduce the same results. Very handy!

# Recording the Commit Hash

When you import a [Python module][], code at the global level of the module is
actually executed. This is often used to set global variables within the
module, which is what we'll do here. [GitPython][] lets us interact with
Git repos from Python and one thing we can do is query a repo to get the
[commit hash][] of the current "[HEAD][]". (`HEAD` is a label in Git pointing
to the latest commit of whatever state the repository is currently in.)

[best practices paper]: http://arxiv.org/abs/1210.0530
[provenance]: http://en.wikipedia.org/wiki/Provenance#Data_provenance
[reproducibility]: http://en.wikipedia.org/wiki/Reproducibility
[numpy]: http://www.numpy.org/
[version control]: http://en.wikipedia.org/wiki/Version_control
[Subversion]: http://subversion.apache.org/
[keyword properties]: http://svnbook.red-bean.com/en/1.4/svn.advanced.props.special.keywords.html
[swc svn provenance]: http://software-carpentry.org/4_0/essays/provenance.html
[Git]: http://git-scm.com/
[git commit]: http://git-scm.com/book/en/Git-Basics-Recording-Changes-to-the-Repository#Committing-Your-Changes
[hash]: http://en.wikipedia.org/wiki/Hash_function
[git checkout]: https://www.kernel.org/pub/software/scm/git/docs/git-checkout.html
[GitPython]: http://pythonhosted.org/GitPython/0.3.1/index.html
[Python module]: http://docs.python.org/2/tutorial/modules.html
[commit hash]: http://docs.python.org/2/tutorial/modules.html
[HEAD]: http://www.gitguys.com/topics/head-where-are-we-where-were-we/
