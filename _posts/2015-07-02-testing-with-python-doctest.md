---
layout: post
title: "Testing with Python: DocTest"
description: "An introduction to Test-Driven Development with Python and Docstrings"
category: code
tags: [python, testing, code]
---

This post covers a fundamental portion of testing with Python: doctest.  Other
posts will cover other parts of testing with Python.

# `doctest`?

`doctest` is a Python module that will read all of the docstrings in your code
and execute anything in those docstrings that looks like Python code typed into
the interactive interpreter.  Here's a brief example:

{% highlight python %}
"""
>>> 5 + 5
10
"""
{% endhighlight %}

If you were to save that as `doctest.py`, you could tell Python to run doctest
against it, and Python would execute the statement `5 + 5`.  You told it that
the answer is `10`, so it would expect the answer to be `10`.  It is, because
math.  Here's an example of a successful test.

{% highlight bash %}
~ ❯ python -m doctest -v test/doctest.py
Trying:
    5 + 5
Expecting:
    10
ok
1 items passed all tests:
   1 tests in doctest
1 tests in 1 items.
1 passed and 0 failed.
Test passed.
~ ❯ 
{% endhighlight %}

> Note that your `doctest.py` has to be in a subfolder for this to work.  You
> can't run `python -m doctest -v` at the same level as the Python file itself.

Cool, right?  What about breaking it?  Let's change that 10 to an 11 and see
the results.  For reference, here's what the test looks like now:

{% highlight python %}
"""
>>> 5 + 5
11
"""
{% endhighlight %}

Our brains, of course, tell us that this is wrong.  What does Python have to
say about it?

{% highlight bash %}
~ ❯ python -m doctest -v test/doctest.py
Trying:
    5 + 5
Expecting:
    11
**********************************************************************
File "test/doctest.py", line 2, in doctest
Failed example:
    5 + 5
Expected:
    11
Got:
    10
**********************************************************************
1 items had failures:
   1 of   1 in doctest
1 tests in 1 items.
0 passed and 1 failed.
***Test Failed*** 1 failures.
~ ❯ 

{% endhighlight %}

This tells you exactly where and why a doctest failed.  Cool, right?

# Proper `doctest` Example

Let's see a slightly more involved example (but not much).  You'll need to
clone the git repo that I'll be using for all of the Python testing posts.

```
git clone https://github.com/supertylerc/tdd-examples
```

This repo has some basic structure built-in.  Here's what it looks like:

{% highlight bash %}
git ❯ tree tdd-examples
tdd-examples
├── LICENSE
└── tdd-examples
    ├── hello_world.py
    ├── __init__.py

2 directories, 6 files
{% endhighlight %}

> There are actually `.pyc` files and a `__pycache__` folder (if you're using
> Python 3) once you run any code, but I've omitted them for brevity.

As you can see, this is not a complex directory structure.  Let's look at the
`tdd-examples/hello_world.py` file.  For reference, here it is:

{% highlight python %}
"""
This module was written to showcase Test-Driven Development for a blog post.
"""


class DocTest(object):
    """This class contains two methods used to showcase doctest.

    Example:
        >>> doc_test = DocTest()
    """
    def hello_world(self, s=None):
        """This method returns the string you pass to it.

        Args:
            s (str): The string to return.

        Returns:
            str: the string s.

        Examples:
            >>> doc_test = DocTest()
            >>> doc_test.hello_world()
            'Hello, World!'
            >>> doc_test.hello_world(s='Hodor!')
            'Hodor!'
        """
        if s is None:
            s = 'Hello, World!'
        return s

    def calculate(self, first_number, second_number, operation='+'):
        """This method performs some basic math calculations for you.

        Args:
            first_number (int): The first number to calculate.
            second_number (int): The second number to calculate.
            operation (str): The mathematical operation to perform.

        Returns:
            int: the result of the mathematical calculation.

        Raises:
            ValueError: if `first_number` or `second_number` cannot be cast
                to ``int``.  Also raised if `operation` is invalid.

        Examples:
            >>> doc_test = DocTest()
            >>> doc_test.calculate(1, 2)
            3
            >>> doc_test.calculate(2, 2, operation='/')
            1
            >>> doc_test.calculate(4, 6, operation='*')
            24
            >>> doc_test.calculate(8, 2, operation='**')
            64
            >>> doc_test.calculate(12, 3, operation='-')
            9
            >>> doc_test.calculate(
            ... 'hodor', 5) # doctest: +IGNORE_EXCEPTION_DETAIL
            Traceback (most recent call last):
            ...
            ValueError
        """

        first_number = int(first_number)
        second_number = int(second_number)
        if operation == '+':
            result = first_number + second_number
        elif operation == '-':
            result = first_number - second_number
        elif operation == '*':
            result = first_number * second_number
        elif operation == '/':
            result = first_number / second_number
        elif operation == '**':
            result = first_number ** second_number
        else:
            raise ValueError('Invalid operation: {}'.format(operation))
        return int(result)
{% endhighlight %}

This should be entirely self-documenting.  There is an object and two methods.
You can even see how the object is instantiated from the `doctest` sections
alone (they're under the `Examples` section).

From this, you can see that you can also test exceptions.  You can use a
special `doctest` directive to ignore the details of the exception (since
those will almost always vary) and instead match the fact that an exception
was raised and that it was of the correct type.

# Why `doctest`?

If you're familiar with unit tests, you may be asking what good doctests are.
The answer is pretty simple: documenting the usage of your code in docstrings
gives you free tests.  Consumers of your code will thank you because they can
see examples of how to use your code...and know that those examples are valid
and working (instead of missing a letter or space here or there).  You will
thank yourself because fewer issues will be raised saying that your
documentation sucks and the examples don't work.

> If you're not familiar with unit tests, I'll be covering them in a future
> blog post.

# Why NOT `doctest`?

`doctest` is not a good method for writing unit tests.  They're ugly to read
when they get complicated, and they won't cover _everything_ without being
really, really, _really_ long.  Another downside?  `doctest` treats the entire
test as a single test instead of as multiple tests.  So if you look at the
docstring test for `DocTest.calculate`, you'll see that I'm testing for several
things--correct answers, operations, exceptions--but they're all seen as a
single test.  Although using the `python -m doctest -v` syntax for running the
test will give you some insight as to where exactly the failure happens, other
testing frameworks won't.  And if you're collecting metrics on failed tests,
it's obviously going to make things look awful when only a single portion of
your `doctest` fails.

# Bottom Line

Do not use `doctest` as a replacement for unit tests.  Write unit tests!  Use
`doctest` as a way to test your documentation and examples.  Users will thank
you.
