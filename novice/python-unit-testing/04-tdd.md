---
layout: page
title: Writing Robust Code and Unit Testing
subtitle: Test-Driven Development
minutes: 20
---

> ## Learning Objectives {.objectives}
>
> * How to choose which unit tests to write to test as much of the code as possible
> * How to use code coverage to determine how much of our code is being tested
> * Using tests to drive the development of the code, through Test-Driven Development (TDD)

Libraries like `nose` can't think of test cases for us. We still have to decide what to test and how many tests to run. Our best guide here is economics: we want the tests that are most likely to give us useful information that we don't already have. For example, if `rectangle_area([0, 0, 1, 1])` works, there's probably not much point testing `rectangle_area((0, 0, 2, 2))`, since it's hard to think of a bug that would show up in one case but not in the other.

A simple way to check the code coverage for a set of tests is to use `nose` to tell us how many statements in our code are being tested. By installing a Python package called `coverage`, that is used by `nose`, we can find this out.

Python has this really handy package manager called `pip` that you can use to install other Python packages. So let's see how we would install the coverage package using `pip`:

~~~ {.in}
$ pip install coverage
~~~

And you'll see something like...

~~~ {.output}
Collecting coverage
  Downloading coverage-4.2.tar.gz (359kB)
    100% |████████████████████████████████| 360kB 873kB/s 
Installing collected packages: coverage
  Running setup.py install for coverage
Successfully installed coverage-4.2
~~~

Then we can simply ask `nose` what our code coverage is for our `test_rectangle2.py` set of tests:

~~~ {.in}
$ nosetests --with-coverage test_rectangle2.py
~~~

So we get something like:

~~~ {.output}
...
Name            Stmts   Miss  Cover
-----------------------------------
rectangle2.py       3      0   100%
----------------------------------------------------------------------
Ran 3 tests in 0.002s

OK
~~~

This tells us that three tests have passed, as before, but also tells us the three statements in `running.py` (the function declaration, the assignment, and returning the calculated value) are all being covered by our tests - 100%! A perfect score, but then we only have three statements! With a larger codebase of a greater number of more complex functions, we need to decide which tests to write to test as much of the code as possible given the amount of time we have to write the tests.

And so for our `test_running.py` set of tests we might get:

~~~ {.in}
$ nosetests --with-coverage test_running.py
~~~

~~~ {.output}
....
Name         Stmts   Miss  Cover
--------------------------------
running.py       8      0   100%
----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
~~~

Now, we should try to choose tests that are as different from each other as possible, so that we force the code we're testing to execute in all the different ways it can - to ensure our tests have a high degree of *code coverage*. Another way of thinking about this is that we should try to find *boundary cases*. If a function works for zero, one, and a million values, it will probably work for eighteen values.

Using boundary values as tests has another advantage: it can help us design our software. 
To see how, consider this test case for our rectangle area function, adding it to 
`test_rectangle2.py`:

~~~ {.python}
def test_inverted_rectangle():
    assert rectangle_area([1, 5, 5, 2]) == -12.0
~~~

Then re-running `nosetests` gives us...
 
~~~ {.in}
$ nosetests test_rectangle2.py
~~~

~~~ {.output}
....
----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
~~~

It passes the test, but is that test correct? i.e., are rectangles with `x1<x0` or `y1<y0` legal, and do they have negative area? Or should the test be:

~~~ {.python}
def test_inverted_rectangle():
    try:
        rectangle_area([1, 5, 5, 2])
        assert False, 'Function did not raise exception for invalid rectangle'
    except ValueError:
        pass # rectangle_area failed with the expected kind of exception
    except Exception:
        assert False, 'Function did not raise correct kind of exception for invalid rectangle'
~~~

The logic in this second version may take a moment to work out, but the idea is straightforward: we want to check that `rectangle_area` raises a `ValueError` exception if it's given a rectangle whose upper edge is below or to the left of its lower edge.

Here's another test case that can help us design our software:

~~~ {.python}
def test_zero_width():
    assert rectangle_area([2, 1, 2, 8]) == 0
~~~

We might decide that rectangles with negative areas aren't allowed, but what about rectangles with zero area, i.e., rectangles that are actually lines? Any actual implementation of `rectangle_area` will do *something* with one of these; writing unit tests for boundary cases is a good way to specify exactly what that something is.

Unit tests are actually such a good way to define how functions ought to behave that many programmers use a practice called *test-driven development* (TDD). Instead of writing code, then figuring out how to test it, these programmers:

1. write some unit tests for a function that doesn't exist yet,
2. write that function,
3. modify it until it passes all of the tests, then
4. clean up the function, i.e., make it more readable or more efficient without breaking any of the tests.

The mantra often used during TDD is "*red, green, refactor*":

-   Get a red light (i.e., some failing tests)
-   Make it turn green (i.e., get something working)
-   Then clean it up by refactoring

This cycle should take anywhere from a couple of minutes to an hour or so. If it takes longer than that, the change being made is probably too large, and should be broken down into smaller (and more comprehensible) steps.

TDD's proponents argue that it helps people produce better code for two reasons:

-   It encourages them to write code in small, self-contained chunks, and to actually write tests for those chunks
-   It frees them from *confirmation bias*: since they haven't written their function yet, their subconscious cannot steer their testing toward proving it correct rather than finding errors.

Empirical studies of TDD have had mixed results: some have found it beneficial,
while others have found no effect. But even if you don't use it day to day,
trying it a few times helps you learn how to design functions and programs that are easier to test.

> ## Challenges {.challenge}
> 
> 1.  Write a function called `addnumbers` in `addnumbers.py` that adds together all the numbers in a given list and returns the total. Make sure it passes the following unit tests:
>     ```
>     from addnumbers import addnumbers
> 
>     def test_empty():
>         assert addnumbers([]) == None
> 
>     def test_single_value():
>         assert addnumbers([1]) == 1
> 
>     def test_two_values():
>         assert addnumbers([1, 2]) == 3
> 
>     def test_three_values():
>         assert addnumbers([1, 2, 3]) == 6
> 
>     ```
>     (You can find this set of tests in `test_addnumbers.py`).
>
> 2. Use the `--with-coverage` argument to `nose` to check your code coverage.
