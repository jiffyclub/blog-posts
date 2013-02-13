> "Yield and overcome" -- Lao Tsu, Tao Te Ching

Over the past year or so I have on a few occasions taught git to people
accustomed to using Subversion. These are experienced software developers so it
seems like it should be easy, but it is almost always a difficult process
filled with [acrimony][], resistance, and gasps. (And this from people who seem
to use Emacs, vim, and Linux without complaint.)

And yet when I teach git to people who don't already have a preferred
version control system it seems to go pretty well. It takes time, and there
may be confusion, but we demo and practice and learn as with any topic.
There is no hate. I think I'm beginning to understand the situation:

1. Git and svn are **different**.
2. People [do not like their workflow changed][xkcd1172].

Git isn't just a little different, it's completely different. But, dear
svn users, git is not different because it doesn't like you. Git is different
because it's optimized to facilitate the collaboration of a number of
developers working on a rapidly changing code base. Some examples:

- Branching is easy in git so that you can work in peace isolated from
  upstream changes until you are ready to send your work back.
- When it's time to merge back git provides `rebase` to facilitate cleanly
  integrating upstream changes.
- Git is distributed so that you and everyone else can make all manner of
  branches and experiments without dirtying up the master repo.

Git **is** different from svn, but it does well what it is designed to do.
This brings me back to the Lao Tsu quote from the top and a simple message
to svn users learning git: You will have to drop your svn workflow to make
the most of git, but it's not because git is bad, it's just different. Yield,
use git in the way it was meant to be used, and be happy.

[acrimony]: https://twitter.com/gvwilson/status/295922302070169600
[xkcd1172]: http://xkcd.com/1172/
