---
layout: post
title: Implementing grep in Python 
---

## Implementing grep in Python

Hello, and welcome to the first installment of my blog. Today we will start
building a clone of grep in pure-python with no third-party extentions. We
will limit the number of features that we will implement today in the name
of space and time. Specifically, we will implement the following features:

1. pass arbitrary search pattern as first positional argument
2. make pattern act as a regular expression (this will differ slightly
from grep's regular expressions because we will use Python's regex engine)
3. pass arbitrary number of files to search (if your shell supports glob
patterns like bash this will mean that this script will as well)
4. Specify -i for case-insensitive search
5. specify -n to print line numbers for the matches
6. specify -H to print filename for the matches

__Note__: For numbers 4 and 5 we will not be implementing the color output
just yet, stay tuned if you are interested in that.

__Note__: If you want to see a semi-production ready version of this script,
checkout [the project on github](https://github.com/ilovetux/greppy). Also,
this is available to be installed from [pypi](https://pypi.python.org/pypi/greppy)
with a simple `pip install greppy`.

### Basic functionality

Let's define the basic structure of our program and include the imports
we are going to need.

```python
import re
import sys
import atexit
import argparse


def main(args):
    pass


def parse_args():
    pass


if __name__ == "__main__":
    main(parse_args())
```

That's basically it. We will put the main (business) logic in our function
`main` and we will put the command line parsing logic in our `parse_args`
function. Now for our basic functionality.

We will allow pattern to be specified positionally, meaning that the first
non-optional argument passed to our script will be the pattern for which to
search. We can do that by changing our `parse_args` function like so:

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("pattern", type=str)
    return parser.parse_args()
```

Now when we invoke our script, the first string passed in will be the pattern
to search for. Next, we need to accept files. This one is a little tricky,
because all of the following conditions have to be met:

1. if no files are passed, we need to default to sys.stdin
2. if more than one file is passed, we need to search all of them
3. if any files are provided, we need to make sure that they are closed
after execution

We can accomplish all of these with the following changes to our script:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        pass


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("pattern", type=str)
    parser.add_argument(
        "files", type=argparse.FileType("r"), default=[sys.stdin], nargs="*")
    return parser.parse_args()

```

So, now we are really cooking, most of our desired functionality is
implemented (at least the tricky parts). Let's explain what's going on here.

In parse_args, We made the following changes:

* we added a two-line statement which adds an argument called `files`.
    * The parameter `type=argparse.FileType("r")` says that anything
    passed in should be a valid filename, that the file should exist
    (ensured by opening in read `"r"` mode) and that the file should be
    open and ready for us to read.
    * The parameter `nargs="*"` says that we should accept 0+ arguments
    (meaning any number even zero)
        * these arguments will be passed to us as a list because of
        specifying `nargs="*"`.
    * The parameter `default=[sys.stdin]` says that if no arguments are
    passed we are to default to a list of one item (stdin).

Now, because argparse will not close our files for us, we had to add the
line `atexit.register(lambda files: [f.close() for f in files], files)`
which tells Python to loop through the files and close them when the script
exits. We also added a skeleton of a loop in our `main` function. This
will simply iterate through the files passed in (or sys.stdin), this is where
our main logic will go.

So, we have enough right now to implement our basic search functionality, so
let's do that. We can have a decently working grep clone with the following
changes:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        for line in file_in:
            if args.pattern in line:
                print line.strip()
```

We can now accept a pattern and a list of files (or pipe input from another
command), and our script will scan each line of input and print the line if
pattern is found in the line. Awesome! This is acutally enough functionality
to cover most of grep's use-cases, but there are a few options which are
common-enough edge cases that they will be sorely missed if they are not
included in our grep clone. Let's get started on those.

### Adding case-insensitive search

It is very common to need to search for a string regardless of case, so let's
get started on that.

This is a very simple change to implement. First let's change our `parse_args`
function to accept a `-i` option:

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--ignore-case", action="store_true")
    parser.add_argument("pattern", type=str)
    parser.add_argument(
        "files", type=argparse.FileType("r"), default=[sys.stdin], nargs="*")
    return parser.parse_args()
```

Now when we look at `args.ignore_case` it will be either `True` or `False`
(read boolean) as to whether to ignore the case when matching. Now that our
users can specify this option let's implement it in our `main` function:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)
    args.pattern = args.pattern.lower() if args.ignore_case else args.pattern

    for file_in in args.files:
        for line in file_in:
            line = line.lower() if args.ignore_case else line
            if args.pattern in line:
                print line.strip()
```

So, we added two lines to our function to offer a `-i` option. The first
line we added changes `args.pattern` to all lower case if `args.ignore_case`
is `True`. The second line does the same for every line of text we are
matching against. By ensuring that both pattern and our line is all lower-case
we can be sure that a match will occur regardless of case.

Let's move onto adding regular expression support.

### Adding RegEx support

Changing our script to treat pattern as a regular expression is fairly
straightforward. We simply need to change our `main` function to the
following:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        for line in file_in:
            flags = re.IGNORECASE if args.ignore_case else 0
            if re.search(args.pattern, line, flags):
                print line.strip()
```

Notice that I was able to replace the two lines dealing with case sensitivity
with just one line, because we are now using Python's regular expression
engine instead of simple string matching. We also had to replace our if
statement to use the `re.search` method.

This still acts as it did before, if you provide a sub-string it will still
match because substrings are valid regular expressions, but now we
will have to escape any special characters if we want them to match literally.

### Adding line numbers

A great addition to all of our current functionality is the ability to print
the line number of the file along with the line. This is really helpful for
large files, where we want to know where certain string(s) are located without
visually inspecting the file. First we need to add an option to our
`ArgumentParser` to allow our users to specify that they want to print the
line numbers. We can do that by changing our `parse_args` function to the
following:

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--ignore-case", action="store_true")
    parser.add_argument("-n", "--line-number", action="store_true")
    parser.add_argument("pattern", type=str)
    parser.add_argument(
        "files", type=argparse.FileType("r"), default=[sys.stdin], nargs="*")
    return parser.parse_args()
```

Now that we can tell when the user wants the line numbers displayed, we can
go ahead and add the functionality. We can do this with one additional line
(and a simple change to an existing line). We need to change our `main`
function to the following:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        for index, line in enumerate(file_in):
            flags = re.IGNORECASE if args.ignore_case else 0
            if re.search(args.pattern, line, flags):
                line = "{}:{}".format(index, line.strip()) if args.line_number else line
                print line.strip()
```

That's it, now line numbers will be printed for each line which matches.

We have one last change for this tutorial, and that is the ability
to add the filename to each line of output. Let's get to it.

### Adding filenames

So when users are scanning multiple files, it can be really helpful to add the
filename to each line of output. This is another very easy change. First, let's
add the argument to our `ArgumentParser`:

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--ignore-case", action="store_true")
    parser.add_argument("-n", "--line-number", action="store_true")
    parser.add_argument("-H", "--with-filename", action="store_true")
    parser.add_argument("pattern", type=str)
    parser.add_argument(
        "files", type=argparse.FileType("r"), default=[sys.stdin], nargs="*")
    return parser.parse_args()
```

Now, we can simply add one more line to add the filename to our output:

```python
def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        for index, line in enumerate(file_in):
            flags = re.IGNORECASE if args.ignore_case else 0
            if re.search(args.pattern, line, flags):
                line = "{}:{}".format(index, line) if args.line_number else line
                line = "{}:{}".format(file_in.name, line) if args.with_filename else line
                print line.strip()
```

That's it, all of our functionality for this tutorial is complete. For your
convenience, the complete code listing is below:

```python
import re
import sys
import atexit
import argparse


def main(args):
    atexit.register(lambda files: [f.close() for f in files], args.files)

    for file_in in args.files:
        for index, line in enumerate(file_in):
            flags = re.IGNORECASE if args.ignore_case else 0
            if re.search(args.pattern, line, flags):
                line = "{}:{}".format(index, line.strip()) if args.line_number else line
                line = "{}:{}".format(file_in.name, line.strip()) if args.with_filename else line
                print line.strip()


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--ignore-case", action="store_true")
    parser.add_argument("-n", "--line-number", action="store_true")
    parser.add_argument("-H", "--with-filename", action="store_true")
    parser.add_argument("pattern", type=str)
    parser.add_argument(
        "files", type=argparse.FileType("r"), default=[sys.stdin], nargs="*")
    return parser.parse_args()


if __name__ == "__main__":
    main(parse_args())
```

Have a good day! See you next time!
