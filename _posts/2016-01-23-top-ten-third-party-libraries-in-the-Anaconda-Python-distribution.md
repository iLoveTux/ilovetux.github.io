---
layout: post
title: Top ten third party libraries in the Anaconda Python distribution
---

Hello and Welcome back! It's been a very long time since my last post (almost
two months), but I'm back and I would like to share with you my top ten
libraries which the
[Anaconda Python Distribution](https://www.continuum.io/downloads) includes
by default. The criteria for selecting these top ten will be somewhat
subjective, but I will try to be unbiased. I will choose based on:

1. Usefulness - How useful the library can be for a developer
2. Convenience - How much the folks at Continuum make our lives easier by
compiling these libraries themselves
3. Ease of use - How user-friendly the library is
4. compatibility - How compatible the library is between Python version and
between Operating systems

### Disclaimer

Like I stated before, this list will be somewhat subjective, and the
criteria for ranking will lead to some strange results. For example,
because we are considering compatibility, it is unlikely that a Linux-only
library will make the list.

## Number 10: Colorama

[on github](https://github.com/tartley/colorama)

[on PyPI](http://pypi.python.org/pypi/colorama)

I use this library extensively as it allows a simple interface to colored
terminal output. For Windows support they override `sys.stdout.write` with
a function which removes the ANSI color codes and makes the appropriate
Win32 API calls. It's simple to use and works on every platform I've tested
it on. For an example let's look at a classic hello world example.

```python
import colorama
from colorama import Fore

colorama.init()

print(Fore.RED + "Hello" + Fore.GREEN + "World!")
```  

The call to `colorama.init()` is the function which overrides `sys.stdout.write`
on Windows. On other systems it is a no-op, meaning it doesn't do anything.

The next part where we use `Fore.RED` and `Fore.GREEN` are constants holding
the value for the `ANSI Escape Sequence` to change the fore-ground color.

There are also constants representing changes to the fore-ground, background,
and styles. There are also ways to modify the cursor position and clear the
screen. I'll leave it as an exercise to the user to learn more about this
package by using it.

A last note, this library can be extremely useful for making a command line
program easier to use for non-experts, but beware that manipulating the
color of output tends to raise expectations from you as people tend to
see this as wizardry, so if you decide to go down this road, you better
brush up on how to manipulate excel spreadsheets.

## Number 9: openpyxl

[website](https://openpyxl.readthedocs.org/en/)

[bitbucket](https://bitbucket.org/openpyxl/openpyxl/)

[pypi](https://pypi.python.org/pypi/openpyxl)

One of my friends was spending three hours per day putting together
spreadsheets for management, when he told me this I was appalled. Now,
I have always been one for automating things and having reproducible
processes, but I've never really "preachy" about it. To each his own,
you know. However, three hours every single day.

After spending just a few minutes with the `openpyxl`
[documentation](http://openpyxl.readthedocs.org/en/2.3.3/) and spending
another few minutes with my friend going over requirements I was able to
hack together a little script which brought the whole process down to 20
minutes of simply gathering the data. I told him it would be a great idea to
automate the process of gathering the data, but there were issues relating
to non-disclosure which prevented us from talking about the specifics.

The greatest thing about it all is that when he passed this script up the
chain to have it vetted, I was offered a job automating processes for their
IT department. This gig lasted 24 months until the contract expired.

For an example, I will create a simple demo of placing directory listings
including information about the files in a workbook. Also to show
off the ability to create multiple worksheets, I will allow the user to
specify multiple directories and have each in it's own worksheet.

```python
import os
import sys
import openpyxl
import argparse


def parse_args(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument("outfile",
                        help="the path and name of the workbook to write, "
                             "should end with '.xlsx'")
    parser.add_argument("directories",
                        nargs="+",
                        help="directories to list in workbook")
    return parser.parse_args(argv)


def main(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    args = parse_args(argv)
    # Filter out files and non-existant directories
    directories = filter(
        lambda x: os.path.exists(x) and os.path.isdir(x),
        args.directories)
    # Create the workbook
    workbook = openpyxl.Workbook()
    # Remove the first sheet, so my code in the loop can be simplified
    workbook.remove_sheet(workbook.active)

    header = ("path", "size", "created", "modified")
    for directory in directories:
        # Create a new worksheet and name it after the directory
        worksheet = workbook.create_sheet()
        worksheet.title = os.path.basename(directory)
        worksheet.append(header)
        # Recurse through the directory
        for root, dirs, files in os.walk(directory):
            # Look at both files and sub-directories
            for _file in files + dirs:
                path = os.path.join(root, _file)
                worksheet.append(
                    (
                        path,
                        os.path.getsize(path),
                        os.path.getctime(path),
                        os.path.getmtime(path)
                    )
                )
    workbook.save(args.outfile)


if __name__ == "__main__":
    main()
```

Now, there may be some subtle bugs here, but I've tested it in Windows 10
and it seems to work very well. I've even used this to find the largest files
on my computer in order to perform a pretty easy clean-up.

## Number 8: psutil

[github](https://github.com/giampaolo/psutil)

[pypi](https://pypi.python.org/pypi/psutil)

[Python Hosted](http://pythonhosted.org/psutil/)

This module has become one of my favorites for a number of tasks, including
monitoring my servers and profiling my code. It is cross-platform (supporting
Linux, Windows, OSX, Sun Solaris, FreeBSD, OpenBSD and NetBSD, both 32-bit and
64-bit architectures) and supports Python 2.6 through 3.5. It is a great way
to get a look at system performance.

For an example, we will look at a program which prints out a table listing
the cpu, memory, disk and network usage of the system. Please note that we
will be simplifying the output a bit for code-readability so if you really
want an in-depth look at resource usage, maybe check out some of the tools
included with your operating system or roll your own solution, this code will
give you a nice head start.

```python
import os
import psutil
from functools import partial
from time import time, sleep, ctime

# A function which should work on most platforms to clear the screen
clear = partial(os.system, 'cls' if os.name == 'nt' else 'clear')


def display_table(table):
    """Send a formatted table to stdout. table should be a sequence
    of sequences with the first sequence being the header."""
    # Make a copy of the list so we don't modify the original
    _table = list(table)
    _header_row = _table.pop(0)
    _template = ""
    for index, field in enumerate(_header_row):
        column = [field] + [str(x[index]) for x in _table]
        column_width = len(max(column, key=len))
        _template += " {%s: <%s} " % (index, column_width)
    clear()
    print(ctime(time()) + "\n")
    header_row = _template.format(*_header_row)
    print header_row
    print("-" * len(header_row))
    for row in _table:
        print(_template.format(*row))


def main():
    header = (
        "cpu (%)",
        "memory (%)",
        "disk-usage (%)",
        "network-sent (bytes)",
        "network-received (bytes)"
    )
    while True:
        table = [header]
        table.append(
            (
                psutil.cpu_percent(),
                psutil.virtual_memory().percent,
                psutil.disk_usage("/").percent,
                psutil.net_io_counters().bytes_sent,
                psutil.net_io_counters().bytes_recv
            )
        )
        display_table(table)
        sleep(1)


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        # We don't need a traceback when the user interrupts the program
        pass
```

So, pretty useful if you just want a quick glimpse at system resource usage,
but also can be built upon since the `psutil` package is extremely useful and
robust. Also, that function in the code example `display_table` has come in
extremely handy especially since it doesn't use any third party modules which
is sometimes an issue.

## Number 7: pygments

[website](http://pygments.org/)

[pypi](https://pypi.python.org/pypi/Pygments)

[bitbucket](http://bitbucket.org/birkenfeld/pygments-main)


`Pygments` is a generic syntax highlighter suitable for a number of scenarios,
and supports over 300 languages.

I love this package, it has made me appear to have super powers, check out our
example for how easy it is to use. In this example we will accept either `stdin`
or any number of files and print it out with syntax highlighting (Please note
you'll probably get better results using the pygmentize command line utility
this is just for demonstration):

```python
from __future__ import print_function
import sys
from pygments import highlight
from pygments.lexers import guess_lexer
from pygments.formatters import TerminalFormatter
# Work-around for some scenarios in Windows
import colorama; colorama.init()


def main():
    print()
    files = [sys.stdin] if len(sys.argv) == 1  else sys.argv[1:]
    for _file in files:
        if _file is not sys.stdin:
            with open(_file, "r") as fin:
                print(fin.name)
                print()
                code = fin.read()
        else:
            print(_file.name)
            print()
            code = sys.stdin.read()
        print(
            highlight(
                code,
                guess_lexer(code),
                TerminalFormatter()
            )
        )


if __name__ == "__main__":
    main()
```

## Number 6: pytest

[website](http://pytest.org/latest/)

[pypi](https://pypi.python.org/pypi/pytest)

[GitHub](https://github.com/pytest-dev/pytest/)

`pytest` is a minimalistic testing framework which can be used for unit testing
or for simple tests you'd like to be able to group together. I use it for unit
testing as well as checking on my servers and other tasks I like to automate
just to be sure that everything is working as expected.

For an example I will show you a simple test script I use for testing my internet
connectivity:

```python
import requests

def test_can_connect_to_google():
    resp = requests.get("https://www.google.com")
    assert resp.ok
```

If you save this as `test-internet.py` and run the command
`py.test test-internet.py`, it will collect one test and execute it,
hopefully telling you that you have connectivity.

This brings us to our next library `requests`

## Number 5: requests

[website](http://docs.python-requests.org/en/latest/)

[GitHub](http://github.com/kennethreitz/requests)

[pypi](http://pypi.python.org/pypi/requests)

`requests` is a library for making requests over http(s) which can come in
quite handy when consuming services without a Python client library, and even
when it does, it likely uses `requests` under the hood. This is a library by
one of my favorite developers/photographers with the moto "Python HTTP for
Humans".

When getting started with HTTP, it can be a bit daunting and the last thing you
need is a library which makes things difficult for you. `requests` fits the bill
perfectly and let's you use almost natural language to make these requests. While
you still need to know the basics of HTTP, `requests` will make it easier to use
simply by not getting in the way.

For our example, we will write a simple script which will grab the archive
zip from a github project simply by accepting an author and a project, then
we will unzip the archive into the current directory. So this will work like
`git clone`, but without grabbing the repository history, just the code.

```python
import os
import sys
import zipfile
import argparse
import requests
from cStringIO import StringIO


def parse_args(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument("author",
                        help="The users whose github repo we "
                             "will be retrieving")
    parser.add_argument("project",
                        help="The project we will be retrieving")
    return parser.parse_args(argv)


def main(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    args = parse_args(argv)
    url = "https://github.com/{}/{}/archive/master.zip".format(args.author, args.project)
    archive = requests.get(url)
    if not archive.ok:
        print "An error occurred while retrieving the project"
    zip_file = zipfile.ZipFile(StringIO(archive.content))
    zip_file.extractall()
    # GitHub places "-master" on the archive name, I like to remove it
    # so, here is how:
    dir_name = zip_file.infolist()[0].filename
    os.rename(dir_name, dir_name.replace("-master", ""))


if __name__ == "__main__":
    main()
```

## Number 4: spyder

[GitHub](https://github.com/spyder-ide/spyder)

[pypi](https://pypi.python.org/pypi/spyder)

I love this, it's almost the perfect Python IDE. If not perfect at least adequate
way more so than `idle`. Although, I've been using [atom](https://atom.io/) from
GitHub lately (I'm almost addicted to trying out new toolsets) I still love
`spyder`, it has integration with IPython, it's own profiler, intelegent
code-completion and much, much more.

To try it out after installing Anaconda simply type spyder at the command prompt
and away you go.

## Number 3: conda and pip

While it may seem weird to have two libraries taking up number three,
it is similarly strange to have two package managers included in a Python
distribution. It is however wonderful, conda is great for packages which require
extra compilation steps and which Anaconda packages up for you, but for other
packages, it is great to be able to use pip. I regularly use both, although it may
be better to take the packages I use and write `conda recipes` for them, but
it's so much quicker and easier to just use `pip`.

## Number 2: lxml

[website](http://lxml.de/)

[GitHub](https://github.com/lxml/lxml/)

[pypi](https://pypi.python.org/pypi/lxml/3.5.0)

While the standard library's `xml` module is passable for most scenarios,
whenever I try to do something advanced I seem to hit my head on one thing
or another. That's why I love having `lxml` just ready for me after a fresh
install of Anaconda. `lxml` is a Python(ic) binding for `libxml2` which is
the de-facto standard for xml processing in the `C` world. It comes with
full xpath support, support for schema validation and support for `XSLT`
transformations.

I've been trying to decide which of two scripts I would use as a sample script
here, but after a while, I figure they have both been very useful, so I'll
include both. The first one will take an `xpath` and an xml file either by
name or from `stdin` and print the results. The second one will take an xslt
filename and an xml file (either by name or from `stdin`) and print the results
to `stdout`.


### xpath.py

```python
import os
import sys
import argparse
from lxml import etree


def parse_args(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument("xpath", help="The xpath to evaluate")
    parser.add_argument(
        "file",
        default="-",
        nargs="?",
        help="The XML file to xpath through defaults to stdin")
    args = parser.parse_args(argv)
    if args.file is "-":
        args.file = sys.stdin
    return args


def main(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    args = parse_args(argv)
    tree = etree.parse(args.file)
    results = tree.xpath(args.xpath)
    for index, result in enumerate(results):
        _index = str(index + 1)
        if isinstance(result, etree._ElementStringResult):
            sys.stdout.write(_index + ": " + result)
        else:
            sys.stdout.write(_index + ": " + etree.tostring(result))
        sys.stdout.write(os.linesep)
        sys.stdout.flush()


if __name__ == "__main__":
    main()
```

### xslt.py

```python
import sys
import argparse
from lxml import etree


def parse_args(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument("xslt", help="The path and filename of the XSLT file")
    parser.add_argument("xml",
                        default="-",
                        nargs="?",
                        help="The XML file to transform, defaults to stdin")
    args = parser.parse_args(argv)
    args.xml = sys.stdin if args.xml == "-" else args.xml
    return args


def main(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    args = parse_args(argv)
    xslt = etree.parse(args.xslt)
    transform = etree.XSLT(xslt)
    xml = etree.parse(args.xml)
    result = transform(xml)
    print etree.tostring(result, pretty_print=True)


if __name__ == "__main__":
    main()
```

## Number 1: pandas

[website](http://pandas.pydata.org/)

[GitHub](https://github.com/pydata/pandas)

[pypi](https://pypi.python.org/pypi/pandas/0.17.1/)

While pandas provides almost everything a fledgling data scientist could want,
it's amazingly easy to get started with. Here is a simple script which takes
tabular data from your system's clipboard and writes it out to an Excel
spreadsheet.

```Python
import sys
import pandas


def main():
    out_file = sys.argv[1]
    dataframe = pandas.read_clipboard()
    dataframe.to_excel(out_file)


if __name__ == "__main__":
    main()
```

See, it's very simple (for some scenarios, you know simple things easy and
difficult things possible) to get started with pandas. The
 [merge](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.merge.html)
 method is extremely useful when you need to correlate some data as well, but
 I'll leave these things as an exercise for the reader.

## Conclusion

Thanks for taking the time to read this list, let me know in the comments
if you liked or hated anything or if anything was useful. This was a joy to
write as it let me look back at all the things I take for granted now that I
have successfully gotten my current employer to switch to using the Anaconda
Python distribution. I almost have them convinced to buy a support contract
from Continuum, I can't wait to have those guys available to help, they
definitely know what they're doing.

Anyway, See you soon, and Happy Coding!
