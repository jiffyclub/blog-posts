
I wrote a bit ago about
[making commits via the GitHub API](http://penandpants.com/2014/02/26/making-
commits-via-the-github-api/).
That post outlined making changes in two simplified situations:
making changes to a single file and making updates to two existing files at
the root of the repository.
Here I show a more general solution that allows arbitrary changes
anywhere in the repo.

I want to be able to specify a repo and branch and say
"here are the contents of files that have changed or been created and
here are the names of files that have been deleted,
please take all that and this message and make a new commit for me."
Because the [GitHub API](http://developer.github.com/)
is so rudimentary when it comes to making commits that will
end up being a many-stepped process, but it's mostly the same steps
repeated many times so it's not a nightmare to code up.
At a high level the process goes like this:

- Get the current repo state from GitHub
    - This is the names and hashes of all the files and directories,
        but not the actual file contents.
- Construct a local, malleable representation of the repo
- Modify the local representation according to the given
    updates, creations, and deletions
- Walk though the modified local "repo" and upload new/changed
    files and directories to GitHub
    - This must be done from the bottom up because a change at
        the low level means every directory above that level will need
        to be changed.
- Make a new commit pointed at the new root tree
    (I'll explain trees soon.)
- Update the working branch to point to the new commit

This blob post is readable as an IPython Notebook at
http://nbviewer.ipython.org/gist/jiffyclub/10809459.
I've also reproduced the notebook below. <!--more-->

I'll start off with the preliminaries that allow me to pull
down the current repo state.
I use the [github3.py](http://github3py.readthedocs.org/en/latest/index.html)
library for abstracting the GitHub API requests.


    import os.path
    from github3 import login

Basic information required for connecting to GitHub
and [which repo](https://github.com/jiffyclub/demodemo)
and branch to work on:


    username = 'jiffyclub'
    token = 'zzz'
    repo_name = 'demodemo'
    branch_name = 'master'

A [Repository](http://github3py.readthedocs.org/en/latest/repos.html#repository-
objects)
instance will be the main interface to the repo.


    gh = login(username=username, token=token)
    repo = gh.repository(username, repo_name)

To actually see repo contents we have to pick a specific branch.
"[Recursing](https://developer.github.com/v3/git/trees/#get-a-tree-recursively)"
on the tree is how I get one long list of all the things in the repo.


    # get the current repo layout
    branch = repo.branch(branch_name)
    tree = branch.commit.commit.tree.recurse()

The repository file structure is represented by a
[Tree](http://github3py.readthedocs.org/en/latest/git.html#github3.git.Tree)
object and individual things within the repo are represented by
[Hash](http://github3py.readthedocs.org/en/latest/git.html#github3.git.Hash)
objects.


    h = tree.tree[0]
    h.path, h.mode, h.sha, h.type

    ('README.md', '100644', 'c385d5f2330a39aca84f2f7999346244bbf0a997', 'blob')



By looping over the tree I can print out the whole repo structure:


    for h in tree.tree:
        print(h.path)

    README.md
    dir1
    dir1/dir1-1.txt
    dir1/dir1-2.txt
    dir1/dir2
    dir1/dir2/dir2-1.txt
    dir1/dir2/dir2-2.txt
    dir1/dir2/dir3
    dir1/dir2/dir3/dir3-1.txt
    dir1/dir2/dir3/dir3-2.txt
    dir4
    dir4/dir4-1.txt
    dir5
    dir5/dir5-1.txt
    dir5/dir5-2.txt
    dir8
    dir8/dir8-1.txt
    dir8/dir8-2.txt
    root1.txt
    root2.txt
    setup.fish


# Malleable Local Repo

Git tracks repository state using two kinds of objects:
[blobs](http://git-scm.com/book/en/Git-Internals-Git-Objects),
which contain file contents,
and [trees](http://git-scm.com/book/en/Git-Internals-Git-Objects#Tree-Objects),
which contain file and directory names pointing to blobs and other trees.

My plan is to represent the current repository state locally,
modify that local state, and finally add the changes to GitHub
via the API.


    def split_one(path):
        """
        Utility function for splitting off the very first part of a path.
        
        Parameters
        ----------
        path : str
        
        Returns
        -------
        head, tail : str
        
        Examples
        --------
        >>> split_one('a/b/c')
        ('a', 'b/c')
        >>> split_one('d')
        ('', 'd')
        
        """
        s = path.split('/', 1)
        if len(s) == 1:
            return '', s[0]
        else:
            return tuple(s)


    split_one('dir1/dir2/dir3')

    ('dir1', 'dir2/dir3')



To match Git's blobs and trees the core of my code will be two classes:
`File` and `Directory`. `File` will be quite simple; it will know
how to post a new blob to GitHub and not much else:


    class File(object):
        """
        Represents a file/blob in the repo.
        
        Parameters
        ----------
        name : str
            Name of this file. Should contain no path components.
        mode : str
            '100644' for regular files, 
            '100755' for executable files.
        sha : str
            Git sha for an existing file, 
            omitted or None for a new/changed file.
        content : str
            File's contents as text. 
            Omitted or None for an existing file,
            must be given for a changed or new file.
        
        """
        def __init__(self, name, mode, sha=None, content=None):
            self.name = name
            self.mode = mode
            self.sha = sha
            self.content = content
        
        def create_blob(self, repo):
            """
            Post this file to GitHub as a new blob.
            
            If this file is unchanged nothing will be done.
            
            Parameters
            ----------
            repo : github3.repos.repo.Repository
                Authorized github3.py repository instance.
            
            Returns
            -------
            dict
                Dictionary of info about the blob:
                
                path: blob's name
                type: 'blob'
                mode: blob's mode
                sha: blob's up-to-date sha
                changed: True if a new blob was created
            
            """
            if self.sha:
                # already up to date
                print('Blob unchanged for {}'.format(self.name))
                changed = False
            else:
                assert self.content is not None
                print('Making blob for {}'.format(self.name))
                self.sha = repo.create_blob(self.content, encoding='utf-8')
                changed = True
            
            return {'path': self.name,
                    'type': 'blob',
                    'mode': self.mode,
                    'sha': self.sha,
                    'changed': changed}

The `Directory`, with its listing of files and other directories,
ties everything together.
With the root directory we can find anything else in the repo.
In fact, the hash of the root tree of a repo is what Git keeps
a record of when you make a commit.
Everything else is referenced off that tree and any trees it contains.


    class Directory(object):
        """
        Represents a directory/tree in the repo.
        
        Parameters
        ----------
        name : str
            Name of directory. Should not contain any path components.
        sha : str
            Hash for an existing tree, omitted or None for a new tree.
        
        """
        def __init__(self, name, sha=None):
            self.name = name
            self.sha = sha
            self.files = {}
            self.directories = {}
            self.changed = False
        
        def add_directory(self, name, sha=None):
            """
            Add a new subdirectory or return an existing one.
            
            Parameters
            ----------
            name : str
                If this contains any path components new directories
                will be made to a depth necessary to construct the full path.
            sha : str
                Hash for an existing directory, omitted or None for a new directory.
            
            Returns
            -------
            `Directory`
                Reference to the named directory.
                If `name` contained multiple path components only the
                reference to the last directory referenced is returned.
            
            """
            head, tail = split_one(name)
            if head and head not in self.directories:
                self.directories[head] = Directory(head)
            
            elif not head:
                # the input directory is a child of the current directory
                if name not in self.directories:
                    self.directories[name] = Directory(name, sha)
                return self.directories[name]
            
            return self.directories[head].add_directory(tail, sha)
        
        def add_file(self, name, mode, sha=None, content=None):
            """
            Add a new file. An existing file with the same name
            will be replaced.
            
            Parameters
            ----------
            name : str
                Name of file. If it contains path components new
                directories will be made as necessary until the
                file can be made in the appropriate location.
            mode : str
                '100644' for regular files, 
                '100755' for executable files.
            sha : str
                Git hash for file. Required for existing files,
                omitted or None for new files.
            content : str
                Content of a new or changed file. Omit for existing files.
            
            Returns
            -------
            `File`
            
            """
            head, tail = os.path.split(name)
            if not head:
                # this file belongs in this directory
                if mode is None:
                    if tail in self.files:
                        # we're getting an update to an existing file
                        assert content is not None
                        mode = self.files[tail].mode
                        assert mode
                    else:
                        raise ValueError('Adding a new file with no mode.')
                        
                self.files[tail] = File(name, mode, sha, content)
            else:
                self.add_directory(head).add_file(tail, mode, sha, content)
        
        def delete_file(self, name):
            """
            Delete a named file.
            
            Parameters
            ----------
            name : str
                Name of file to delete. May contain path components.
            
            """
            head, tail = os.path.split(name)
            
            if not head:
                # should be in this directory
                del self.files[tail]
                self.changed = True
            else:
                self.add_directory(head).delete_file(tail)
        
        def create_tree(self, repo):
            """
            Post a new tree to GitHub.
            
            If this directory and everything in/below it 
            are unchanged nothing will be done.
            
            Parameters
            ----------
            repo : github3.repos.repo.Repository
                Authorized github3.py repository instance.
            
            Returns
            -------
            tree_info : dict
                'path': directory's name
                'mode': '040000'
                'sha': directory's up-to-date hash
                'type': 'tree'
                'changed': True if a new tree was posted to GitHub
            
            """
            tree = [f.create_blob(repo) for f in self.files.values()]
            tree = tree + [d.create_tree(repo) for d in self.directories.values()]
            tree = list(filter(None, tree))
    
            if not tree:
                # nothing left in this directory, it should be discarded
                return None
    
            # have any subdirectories or files changed (or been deleted)?
            changed = any(t['changed'] for t in tree) or self.changed
            
            if changed:
                print('Creating tree for {}'.format(self.name))
                tree = [{k: v for k, v in t.items() if k != 'changed'} for t in tree]
                self.sha = repo.create_tree(tree).sha
            else:
                print('Tree unchanged for {}'.format(self.name))
            assert self.sha
            return {'path': self.name,
                    'mode': '040000',
                    'sha': self.sha,
                    'type': 'tree',
                    'changed': changed}

With the `File` and `Directory` classes defined I can construct
the current repo state.
Everything starts with the unnamed root directory.
I filter out the blobs and trees so I can add the directories
first, though this isn't strictly necessary.


    trees = [h for h in tree.tree if h.type == 'tree']
    blobs = [h for h in tree.tree if h.type == 'blob']
    
    root = Directory('', branch.commit.commit.tree.sha)
    
    for h in trees:
        root.add_directory(h.path, h.sha)
    
    for h in blobs:
        root.add_file(h.path, h.mode, h.sha)

# Set up some changes and deletions

With the repo state reconstructed locally I'll configure some changes.
There are changes to existing files, new files, and file deletions.


    # 'mode': None indicates it's an existing file and the mode should be kept as is
    # New files must give a valid 'mode' parameter
    updates = [{'path': 'README.md', 
                'content': 'a', 
                'mode': None},
               {'path': 'dir1/dir1-1.txt', 
                'content': 'b', 
                'mode': None},
               {'path': 'dir1/dir2/dir3/dir3-2.txt', 
                'content': 'c', 
                'mode': None},
               {'path': 'dir1/dir2/dir3/dir3-3.txt', 
                'content': 'e', 
                'mode': '100644'},
               {'path': 'root3.txt', 
                'content': 'f', 
                'mode': '100644'},
               {'path': 'dir6/dir7/dir7-1.txt', 
                'content': 'g', 
                'mode': '100644'}]
    
    # paths to deleted files
    deleted = ['root1.txt',
               'dir1/dir2/dir2-1.txt',
               'dir1/dir2/dir2-2.txt',
               'dir4/dir4-1.txt',
               'dir8/dir8-1.txt']

The next step is to update the local repo representation:


    # make our local repo reflect how we want it to look
    # after changing/adding/deleting files
    for thing in updates:
        root.add_file(thing['path'], thing['mode'], content=thing['content'])
    
    for d in deleted:
        root.delete_file(d)

# Make all the new blobs and trees created by the changes

The local repo representation now has the same structure
I want the repo on GitHub to have.
To get all the updates sent to GitHub I call the
`.create_tree` method on the root directory.
That method in turn calls the `.create_tree`
and `.create_blob` methods on all the directories
and files below, which in turn do the same.
One by one each changed file and directory will have
its data sent to GitHub and finally I'll have the
hash of the new root tree that I can use in a commit.


    root_info = root.create_tree(repo)

    Making blob for root3.txt
    Blob unchanged for root2.txt
    Making blob for README.md
    Blob unchanged for setup.fish
    Blob unchanged for dir1-2.txt
    Making blob for dir1-1.txt
    Blob unchanged for dir3-1.txt
    Making blob for dir3-3.txt
    Making blob for dir3-2.txt
    Creating tree for dir3
    Creating tree for dir2
    Creating tree for dir1
    Making blob for dir7-1.txt
    Creating tree for dir7
    Creating tree for dir6
    Blob unchanged for dir8-2.txt
    Creating tree for dir8
    Blob unchanged for dir5-1.txt
    Blob unchanged for dir5-2.txt
    Tree unchanged for dir5
    Creating tree for 

    root_info

    {'sha': '3f1e781ebde83629df62de0a869169d29c10e435',
     'path': '',
     'type': 'tree',
     'mode': '040000',
     'changed': True}



# Make a new commit

At this point GitHub has all of my new data but there's
nothing in the history of my repo pointing at this new state.
That requires making a new commit.
The ingredients for a new commit are a message,
the sha hash of a tree (from which can be derived the
entire repo state), and a parent commit(s) for linking the
new commit to the rest of the project history.


    new_commit = repo.create_commit('Making a whole bunch of changes all over via the GitHub API.',
                                    tree=root_info['sha'],
                                    parents=[branch.commit.sha])


    new_commit

    <Commit [Matt Davis:753e75b9891afac88ecc9fae86ec0bc11fa009c6]>

    new_commit.html_url

    'https://github.com/jiffyclub/demodemo/commit/753e75b9891afac88ecc9fae86ec0bc11fa009c6'



# Update master branch to point to new commit

The commit is now part of my project's history,
but my working branch has not been updated to point
at the new commit.
This happens implicitly when you work with Git at the
command line, but when working via the API it has
to be done manually.

The procedure for this is to get a
[Reference](http://github3py.readthedocs.org/en/latest/git.html#github3.git.Refe
rence)
instance for the working branch and use its `.update`
method to point it at the new commit.

    ref = repo.ref('heads/{}'.format(branch_name))
    ref.update(new_commit.sha)

    True

A return value of `True` indicates success.

# What's Missing

I haven't made any attempt here to test symlinks or
binary content like images.
Those could require some special handling,
but I think it'll be maneagable.


    
