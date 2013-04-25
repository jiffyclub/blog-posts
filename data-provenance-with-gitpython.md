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
(Your software [is under version control][best practices paper], isn't it?)
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
module, which is what we'll do here.

[GitPython][] lets us interact with Git repos from Python and one thing we can
do is query a repo to get the [commit hash][] of the current "[HEAD][]".
(`HEAD` is a label in Git pointing to the latest commit of whatever state the
repository is currently in.)

What we can do with that is make it so that when our software modules are
imported they set a global variable containing the commit hash of their `HEAD`
at the time the software was run. That hash can then be inserted into data
products as a record of the software version used to create them. Here's
some code that gets and stores the hash of the `HEAD` of a repo:

    from git import Repo
    MODULE_HASH = Repo('/path/to/repo/').head.commit.hexsha

If the module we're importing is actually inside a Git repo we can use a bit
of Python magic to get the `HEAD` hash without manually listing the path
to the repo:

    import os.path
    from git import Repo
    repo_dir = os.path.abspath(os.path.dirname(__file__))
    MODULE_HASH = Repo(repo_dir).head.commit.hexsha

(`__file__` is a global variable Python automatically sets in
[imported][] modules.)

# Versioned Data

Some data formats, especially those that are text based, can be easily stored
in version control. If you can put your data in a Git repo then the same
strategy as above can be used to get and store the `HEAD` commit of the data
repo when you run your analysis, allowing you to reproduce both your software
and data states during later runs. If your data does not easily fit into Git
it's still a good idea to record a unique identifier for the dataset, but you
may need to develop that yourself (such as a simple list of all the data files
that were used as inputs).

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
[commit hash]: http://git-scm.com/book/en/Getting-Started-Git-Basics#Git-Has-Integrity
[HEAD]: http://www.gitguys.com/topics/head-where-are-we-where-were-we/
[imported]: http://docs.python.org/2/reference/simple_stmts.html#the-import-statement
