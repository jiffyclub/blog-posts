Testing Python results is often as straightforward as
`assert result == expected`, especially with builtin types.
But that doesn't work with [NumPy][] or [Pandas][] data structures
because using `==` with those doesn't return `True` or `False`.
Instead, `==` results in new arrays filled with boolean values.
This is useful for [boolean indexing][], but leads to this
error when testing:

```python
In [2]: a = np.arange(10)

In [3]: b = np.arange(10)

In [4]: assert a == b
------------------------------------------------
ValueError     Traceback (most recent call last)
<ipython-input-4-6bf76ad3480a> in <module>()
----> 1 assert a == b

ValueError: The truth value of an array with more than one element is ambiguous.
            Use a.any() or a.all()
```

You can check whether all the elements in two arrays are equal using the
`.all()` method:

```python
In [5]: (a == b).all()
Out[5]: True
```

But that errs if the arrays are different sizes/shapes, and the result
is an uninformative `True` or `False` when they are the same size.
Luckily, NumPy has this situation covered.

# Library Versions

For reference, these are the versions of NumPy and Pandas I'm currently using:

```python
In [43]: np.version.version
Out[43]: '1.9.0'

In [44]: pd.version.version
Out[44]: '0.14.1'
```

# Testing with NumPy

NumPy has an [entire module][numpy.testing] devoted to testing support.
I like to import it via `import numpy.testing as npt` in my tests.
I'll be focusing here on two functions,
[`assert_array_equal`][] and [`assert_allclose`][].

## `assert_array_equal`

[`assert_array_equal`][] raises an `AssertionError` when to arrays are not
exactly equal. It can take anything array-like as inputs, including lists.

```python
In [10]: npt.assert_array_equal([1, 2, 3], [1, 2, 3])

In [11]: npt.assert_array_equal([1, 2, 3], [1, 2, 3, 4, 5])
----------------------------------------------------
AssertionError     Traceback (most recent call last)
<truncated>

AssertionError:
Arrays are not equal

(shapes (3,), (5,) mismatch)
 x: array([1, 2, 3])
 y: array([1, 2, 3, 4, 5])

In [12]: npt.assert_array_equal([1, 2, 3], [99, 2, 3])
----------------------------------------------------
AssertionError     Traceback (most recent call last)
<truncated>

AssertionError:
Arrays are not equal

(mismatch 33.33333333333333%)
 x: array([1, 2, 3])
 y: array([99,  2,  3])
```

The examples show how you get somewhat descriptive output when the
comparisons fail, including if the shapes are mismatched and what
percentage of elements differ between the two arrays.

Similar functionality is available in the [`array_equal`][] function,
which returns `True` or `False` instead of raising an exception.

## `assert_allclose`

`assert_array_equal` checks for exact equality.
That's fine for integer and boolean values, but often fails with
floating point values because of very slight differences in
the results of values calculated different ways or on different computers.
For comparing floating point values I use [`assert_allclose`][].

```python
In [17]: npt.assert_array_equal([np.pi], [np.sqrt(np.pi) ** 2])
-------------------------------------------------------
AssertionError        Traceback (most recent call last)
<truncated>

AssertionError:
Arrays are not equal

(mismatch 100.0%)
 x: array([ 3.141593])
 y: array([ 3.141593])

In [18]: npt.assert_allclose([np.pi], [np.sqrt(np.pi) ** 2])
```

`assert_allclose` takes `atol` and `rtol` arguments for specifying the
absolute and relative tolerance of the comparison.
For the most part I leave these at their defaults:
`atol=0` and `rtol=1e-07`.
That's a small enough tolerance that I'm confident the numbers are quite close,
but large enough to let floating point noise go through.
Sometimes, though, it's useful to choose custom tolerances.
For example, I was once writing tests based on numbers I copied out of
a paper.
The numbers were provided to four decimal places so in my tests I used
`npt.assert_allclose(result, expected, atol=0.0001)`.
Choosing appropriate tolerances for testing with `assert_allclose` can
be tricky depending how accurate you expect your code to be.
Unfortunately, I don't have any great advise on that.

`assert_allclose` also has a non-assertion version: [`allclose`][].

## Notes

One very handy thing about `assert_array_equal`
(and its scalar friendly cousin [`assert_equal`][])
is that it handles values like `nan` intelligently.
Normally `nan` compared to anything else, even `nan`, results in `False`.
That's the official, expected behavior, but it does make testing harder.
`assert_array_equal` handles this for you.

```python
In [29]: (np.array([np.nan, 2, 3]) == np.array([np.nan, 2, 3])).all()
Out[29]: False

In [30]: npt.assert_array_equal([np.nan, 2, 3], [np.nan, 2, 3])
```

Note that `array_equal` and [`equal`][] behave in the official manner and
will always return `False` for comparisons to `nan`.

# Testing with Pandas

Pandas also has a testing module, but it is apparently meant more for
internal testing of Pandas itself than for Pandas users.
There is no documentation page for it, but it's still available and I
use it in testing. I import it via `import pandas.util.testing as pdt`.

The three main things I use are
[`assert_frame_equal`][], [`assert_series_equal`][],
and [`assert_index_equal`][].
`assert_frame_equal` and `assert_series_equal` take arguments
that let you control whether the comparisons are exact or approximate,
and whether to compare types in addition to value equality.
By default they use an `allclose`-like comparison.

```python
In [39]: s1 = pd.Series([1, 2, 3], dtype='int')

In [40]: s2 = pd.Series([1, 2, 3], dtype='float')

In [41]: pdt.assert_series_equal(s1, s2)
-------------------------------------------------------
AssertionError        Traceback (most recent call last)
<truncated>

AssertionError: attr is not equal [dtype]: dtype('int64') != dtype('float64')

In [42]: pdt.assert_series_equal(s1, s2, check_dtype=False)
```

`assert_frame_equal` is sensitive to the order of columns and rows
in the tables. I've found this is not always what I want, sometimes
it's fine if ordering changes as long as the same column names and
index labels are in both tables.
I've made my own [`assert_frames_equal`][] function for testing that case.

***

Just because you're using complex data containers like arrays and DataFrames
in your code doesn't mean you can't test it.
NumPy and Pandas are themselves heavily tested and you can test your own code
using the same utilities the NumPy and Pandas developers use.
Happy testing!

[NumPy]: http://www.numpy.org/
[Pandas]: http://pandas.pydata.org/
[boolean indexing]: http://docs.scipy.org/doc/numpy/user/basics.indexing.html#boolean-or-mask-index-arrays
[numpy.testing]: http://docs.scipy.org/doc/numpy/reference/routines.testing.html
[`assert_array_equal`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.testing.assert_array_equal.html
[`assert_allclose`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.testing.assert_allclose.html
[`assert_equal`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.testing.assert_equal.html
[`array_equal`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.array_equal.html
[`allclose`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.allclose.html#numpy.allclose
[`equal`]: http://docs.scipy.org/doc/numpy/reference/generated/numpy.equal.html
[`assert_frame_equal`]: https://github.com/pydata/pandas/blob/29de89c1d961bea7aa030422b56b061c09255b96/pandas/util/testing.py#L621
[`assert_series_equal`]: https://github.com/pydata/pandas/blob/29de89c1d961bea7aa030422b56b061c09255b96/pandas/util/testing.py#L592
[`assert_index_equal`]: https://github.com/pydata/pandas/blob/29de89c1d961bea7aa030422b56b061c09255b96/pandas/util/testing.py#L568
[`assert_frames_equal`]: http://nbviewer.ipython.org/gist/jiffyclub/ac2e7506428d5e1d587b
