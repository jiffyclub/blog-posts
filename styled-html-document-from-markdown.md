There are many, many command line converters for turning [Markdown][] into
HTML, but for the most part these make HTML fragments, not full documents with
CSS styling. That's fine most of the time (e.g. when I'm writing
[blog posts][]), but sometimes I want a full, pretty document so I can print
it out (typically for presentation notes).

To fill this hole I put together a [small script][markdown_doc] that converts
Markdown and wraps the HTML result in a template that includes [Bootstrap][]
CSS. I set the fonts to `sans-serif` and `monospace` so that they are taken
from the defaults for your browser, making it easier to use your favorite
fonts.

The script requires the Python libraries [Python Markdown][py-markdown],
[mdx_smartypants][] (a Python-Markdown extension), and [Jinja2][].

[Markdown]: http://daringfireball.net/projects/markdown/
[blog posts]: https://github.com/jiffyclub/blog-posts
[markdown_doc]: https://gist.github.com/jiffyclub/5015986
[Bootstrap]: http://getbootstrap.com/
[py-markdown]: http://pythonhosted.org/Markdown/
[mdx_smartypants]: https://bitbucket.org/jeunice/mdx_smartypants
[Jinja2]: http://jinja.pocoo.org/
