I recently started over with a fresh development environment and decided
to try something new: I'm using [Python 3][] via [miniconda][].
The first real hiccup I've run into is that [conda's][conda]
environment activation/deactivation scheme only works in [bash][]
or [zsh][]. I use [fish][]. There is an [open PR][fish-pr] to get fish
support for conda but in the meantime I hacked something together to
help me out.

"Activating" a [conda][] environment does a couple of things:

- Puts the environment's "bin" directory at the front of the `PATH`
  environment variable.
- Sets a `CONDA_DEFAULT_ENV` environment variable that tells conda
  in which environment to do things when none is specified.
- Adds the environment name to the prompt ala [virtualenv][].

Deactivating the environment resets everything to its pre-activation state.
The `fish` functions I put together work like this:

    ~ > type python
    python is /Users/---/miniconda3/bin/python
    ~ > condactivate env-name
    (env-name) ~ > type python
    python is /Users/---/miniconda3/envs/env-name/bin/python
    (env-name) ~ > deactivate
    ~ > type python
    python is /Users/---/miniconda3/bin/python

Here's the text of the functions:

[gist]https://gist.github.com/jiffyclub/10837487b053ad66ef08[/gist]

Or you can download it from
https://gist.github.com/jiffyclub/10837487b053ad66ef08.

To use these, add them to the `~/.config/fish/` directory and
[source][fish-source] them from the end of the `~/.config/fish/config.fish`
file:

    source $HOME/.config/fish/conda.fish

[Python 3]: http://docs.python.org/3/
[miniconda]: http://conda.pydata.org/miniconda.html
[conda]: http://conda.pydata.org/docs/index.html
[bash]: https://www.gnu.org/software/bash/
[zsh]: http://www.zsh.org/
[fish]: http://fishshell.com/
[fish-pr]: https://github.com/conda/conda/pull/545
[virtualenv]: http://www.virtualenv.org/en/latest/
[conda.fish]: https://gist.github.com/jiffyclub/10837487b053ad66ef08
[fish-source]: http://fishshell.com/docs/current/commands.html#source
