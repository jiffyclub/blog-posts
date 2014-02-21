[Pen and Pants][] is hosted by [WordPress][], but I write my blog posts
in [my favorite text editor][sublime] using [Markdown][]. That way I have
all the conveniences those afford and I can [archive the posts in plain
text on GitHub][blog-posts].

The tricky part is going from the `.md` files to some text I can paste into
the input box in WordPress. I learned today that you can
[write posts in Markdown][wp-markdown], but that still doesn't work perfectly
for me because WordPress treats new lines within blocks as hard breaks.
(When writing posts I break all lines before 80 characters for more
convenient editing and diffing. Keeping all those breaks literal doesn't
translate well to web pages.)

Today, thanks to [Ethan White][], I figured out that [Pandoc][] can help.
By converting my Markdown to Markdown with the `--no-wrap` flag Pandoc
will output paragraphs on a single line but otherwise give me regular Markdown.
The command I use looks like this:

    pandoc -f markdown -t markdown --no-wrap blog-post.md

I can take the output of that and past it into WordPress' text input box
(after ticking the box to allow Markdown when writing posts).

Note that if you use fenced codeblocks ([as on GitHub][gh-code]) WordPress will
convert that into its special source code widget. If instead you want something
presented using only `<pre><code>` tags then [use indentation][md-code] to
indicate it is pre-formatted text.

#### Tips for Mac Users

If you use [Homebrew][] you can install [Pandoc][] via the [cask][] add on:

    brew cask install pandoc

To copy the output of `pandoc` straight to the clipboard you can use the
`pbcopy` command:

    pandoc -f markdown -t markdown --no-wrap blog-post.md | pbcopy

[Pen and Pants]: http://penandpants.com
[WordPress]: http://wordpress.com/
[sublime]: http://www.sublimetext.com/3
[Markdown]: http://daringfireball.net/projects/markdown/
[blog-posts]: https://github.com/jiffyclub/blog-posts
[wp-markdown]: http://en.support.wordpress.com/markdown/
[Ethan White]: https://twitter.com/ethanwhite
[Pandoc]: http://johnmacfarlane.net/pandoc/
[Homebrew]: http://brew.sh/
[cask]: https://github.com/phinze/homebrew-cask
[gh-code]: https://help.github.com/articles/github-flavored-markdown#fenced-code-blocks
[md-code]: http://daringfireball.net/projects/markdown/syntax#precode
