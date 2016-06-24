Where in the world is this function
===================================

So a little back-story:

I am writing a Python package which will provide a number of commands which
are simplified versions of commands usually found on *nix systems and I needed
a way to get the terminal width for implementing `ls` since by default `ls`
displays in columns. In Python 3.3+ this is easy:

```python
import shutil

x, y = shutil.get_terminal_size()
```  

Short, sweet and to the point. However, this is not available in Python 2.7
which I am planning on supporting. I had hoped that it would be available in the
`__future__` module, but alas that is not the case. So I decided to trace down
the function definition. It is defined like so (I removed the docstring):

```python
def get_terminal_size(fallback=(80, 24)):
    # columns, lines are the working values
    try:
        columns = int(os.environ['COLUMNS'])
    except (KeyError, ValueError):
        columns = 0

    try:
        lines = int(os.environ['LINES'])
    except (KeyError, ValueError):
        lines = 0

    # only query if necessary
    if columns <= 0 or lines <= 0:
        try:
            size = os.get_terminal_size(sys.__stdout__.fileno())
        except (NameError, OSError):
            size = os.terminal_size(fallback)
        if columns <= 0:
            columns = size.columns
        if lines <= 0:
            lines = size.lines

    return os.terminal_size((columns, lines))
```

The return value is a subclass of `tuple` meant to carry only columns and lines.
If we look at the code, we try to get these values from environment variables
which are sometimes present, but if these environment variables are not present
we try to call a function called `os.get_terminal_size`. Which according to the
[docs](https://docs.python.org/3/library/os.html#os.get_terminal_size) is a
low-level implementation.

OK, so let's go look in the os.py file and get that function definition:

```bash
$ grep get_terminal_size ~/anaconda3/lib/python3.5/os.py
$
```

...WHAT?

Where to go from here, I know...the inspect module:

```python
>>> import os
>>> import inspect

>>> print(inpect.getsourcelines(os.get_terminal_size))
TypeError: <built-in function get_terminal_size> is not a module, class, method, function, traceback, frame, or code object
```

`built-in`, I don't think so...according to those
[docs](https://docs.python.org/3/library/functions.html#built-in-functions)
there is no built in function by the name `get_terminal_size` and in my own
experience, there aren't built in functions within the namespace of a module.

Back to the drawing board...ok let's try this:

```python
>>> import os
>>> os.get_terminal_size
<function posix.get_terminal_size>
```

posix?!?! I didn't import the posix module...I've never heard of the posix
module...Anyway let's try to get a look at that:

```python
>>> import inspect
>>> import posix
>>> print(inspect.getsourcelines(posix.get_terminal_size))
TypeError: <built-in function get_terminal_size> is not a module, class, method, function, traceback, frame, or code object
```

back where we started, but I feel like I'm getting closer...Let's take a look
at that module:

```python
>>> import posix
>>> print posix.__file__
AttributeError: module 'posix' has no attribute '__file__'
```

ummm. ok...

```python
>>> import posix
>>> posix
<module 'posix' (built-in)>
```

So, posix is built-in...Let's hit the
[docs](https://docs.python.org/3/library/posix.html), So __Do not import this
module directly.__, 
