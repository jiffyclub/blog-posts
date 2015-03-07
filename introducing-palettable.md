I [wrote brewer2mpl a couple years ago](http://penandpants.com/2012/07/27/brewer2mpl/)
to help people use [colorbrewer2][] color palettes in Python.
Since then it's expanded to include palettes from [Tableau][]
and the whimsical [Wes Anderson Palettes][] Tumblr;
and there's plenty of room for more palettes from other sources.
To encompass the growing scope, brewer2mpl has been renamed to Palettable!
(Thanks to [Paul Ivanov][] for the name.)

The Palettable API has also been updated for the [IPython][] age.
All available palettes are now loaded at import and are available
for your tab-completion pleasure.
Need the `YlGnBu` palette with nine colors?
That's now available at `palettable.colorbrewer.sequential.YlGnBu_9`.
Reversed palettes are also available with a `_r` suffix.

I hope you find Palettable useful!
You can find it on the web at:

- Docs: https://jiffyclub.github.io/palettable/
- GitHub: https://github.com/jiffyclub/palettable
- PyPI: https://pypi.python.org/pypi/palettable/

P.S.: Here's a little
[demo notebook](http://nbviewer.ipython.org/github/jiffyclub/palettable/blob/master/demo/Palettable%20Demo.ipynb).

[colorbrewer2]: http://colorbrewer2.org/
[Tableau]: http://www.tableau.com/
[Wes Anderson Palettes]: http://wesandersonpalettes.tumblr.com/
[Paul Ivanov]: http://pirsquared.org/
[IPython]: http://ipython.org/
