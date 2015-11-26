---
layout: post
title: A Single-File Python Interpreter in Under 100 Lines of Code
---

Well, I'll start off by saying that the title of this blog post is a bit misleading, I will not actually be writing a Python interpreter, compiler, lexer or parser. Instead I will share a method which has allowed me to perform minor miracles in certain (very restricted) IT environments.

We will write a simple Python script which will act like a Python Interpreter in as much as if you call it and pass in a Python source file it will execute it. This will either sound too good to be True or it will sound like a childish solution. Either way, Let's go!

If you want to check this project out, just head over to [the github repo](https://github.com/ilovetux/moon.py). Also, the source (as it stands now 10-16-2015) is listed at the end.

### Base Functionality

Let's get a skeleton going for our project. I start a lot of projects out like this:

```python
import os
import sys
import code
import runpy
import atexit
import argparse

__version__ = "moonpy 0.6.0"

def main(argv=None):
    pass

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    return parser.parse_args(argv)

if __name__ == "__main__":
    main()
```

I've included the imports, so if you are familiar with these libraries it should seem pretty obvious how I'm going to go about making an interpreter, if you are not, then no worries just read along as you're about to see some very cool libraries at work.

I've also highlighted a line above where I adjust argv to be `sys.argv[1:]` if no argv was passed in. This is important and took me a long time to figure out. Aparantly, if you do not pass anything to `argparse.ArgumentParser.parse_args` then it's smart enough to strip the first argument (the scripts name), but if you explicitly pass in a list of args, it does not. For testing purposes, I wanted to pass in lists instead of modifying sys.argv everytime, so this is one way to support that.

Now, I am going off of specs laid out in [this](https://docs.python.org/2/using/cmdline.html) document, so if you want to follow along you should open that page in a new tab. Now at first, we will not be supporting all arguments, but the most commonly used ones.

First up is:

> When called with standard input connected to a tty device, it prompts for commands and executes them until an EOF (an end-of-file character, you can produce that with Ctrl-D on UNIX or Ctrl-Z, Enter on Windows) is read.

you can read that as if there are no arguments, start a REPL (Read, Eval, Print Loop)

### Adding a REPL

Well, I've actually coded a few REPLs in my day, but there is no need to do this if you just want an interactive Python interpreter as the [code](https://docs.python.org/2/library/code.html) module (which is in the standard library) provides an interactive console. Since there are no other options to support yet, the logic might look bare, but don't worry it'll flesh out very soon.

```python
import os
import sys
import code
import runpy
import atexit
import argparse

__version__ = "moonpy 0.6.0"

def interact(console=None):
    """Launch an enhanced, interactive Python interpreter"""
    sys.argv = [""]
    if not console:
        console = code.InteractiveConsole(locals={"exit": sys.exit})
    console.interact("MoonPy - m-o-o-n that spells Python")

def main(argv=None):
    interact()

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    args = parser.parse_args(argv)
    return args

if __name__ == "__main__":
    main()
```

If you look at the `parse_args` function, you will see that I added a bit, but that's just to get it to run, however we will be going over that, but first let's look at the highlighted lines.

The `interact()` function will accept a console argument, but if none is provided it will create one. Because Python presents `sys.argv` as a single item list `['']` when called in this form (without arguments), we go ahead and take care of that in the first line of the function. Next, we create a new `code.InteractiveConsole()` if none was provided. Finally, we tell the console to `interact()` with the user presenting a custom banner. That's all we have to do to add an interactive mode to our program the `code` module will handle all of the hard parts.

In the `main` function, as we don't yet have support for any other modes of execution is very simple, it just invokes the `interact()` function. This will soon be filled up with logic to handle our various features.

Next up on our list is:

> When called with a file name argument or with a file as standard input, it reads and executes a script from that file.

Let's get started on that!

### Adding file support

So we need to allow a file to be specified, and that file needs to default to `sys.stdin` and it should be `stdin` if a `-` is supplied as the argument. This is pretty simple to implement. While we are at it we will also add support for `-i` which should drop you into an interactive interprete after executing `file`. Take a look below:

```python
def run_path(args):
    """Run a path (script, zip file, or dir) just like python <script>"""
    sys.argv.pop(0)
    _locals = runpy.run_path(args.file, run_name="__main__")
    if args.i:
        interact(code.InteractiveConsole(locals=_locals))

def main(argv=None):
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            pass
    else:
        run_path(args)

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", action="store_true")
    parser.add_argument("file", type=str, default="-", nargs="?")
    parser.add_argument("args", nargs=argparse.REMAINDER)
    args = parser.parse_args(argv)
    args.file = sys.stdin if args.file == "-" else args.file
    return args
```

So, let's first look at the changes to the `parse_args()` function. We added three arguments `-i`, `file` and `args`. These are all for supporting the script functionality. The `-i` will drop us into an interactive interpreter after executing the `file`, `args` collects the reminder of the arguments which is important becuse we don't want our program to try and interpret them. Finally, the file argument collects the `file` to execute. Since we don't want to actually open the file ourselves, we accept `str`s and default to `-` which we change to `sys.stdin` before returning the parsed arguments.

In the `main()` function, we now have two big `if` statements. Our control flow goes like this:

1. if `file` is `stdin` there are two possibilities
    1. user wants an interactive interpreter
    2. user piped in code like `cat example.py | python moon.py`
2. if `file` is not `sys.stdin`, then it is a source file to be executed

The first possibility of `file` being `sys.stdin` we already handled, this is when the user wants an interactive interpreter. The second (the user piped in source we will handle in the next section).

The big change to this is when the user wants to execute a script (or module) if this is the case then we call `run_path()` which we defined above.

The `run_path()` function does three things:

1. Remove our script's name from `sys.argv` so the script being executed thinks it was the first argument
2. Runs the script, while also capturing the state of `locals()`
3. If `-i` was provided, drop the user into an interactive interpreter with the same environment as the script had

The big feature here is provided courtesy of the `runpy.run_path` function, which adds a ton of functionality to us for free, namely:

1. will run a Python source file (eg. sample.py)
2. will run a module in a directory (eg. src/module/) (as long as a `__main__.py` is present)
3. Run a module contained in a zip file (if run with Python 2.6+), This is an incredibly useful feature which I just learrned about while working on this project I can't wait to use it in one of my future projects.
4. It returns the value of `locals()` so we can initialize the interactive interpreter with them

Next, let's handle the case when a user pipes in source code!

### Supporting piped `stdin` source

Check out the following code:

```python
def run_stdin(stdin):
    sys.argv[0] = ""; sys.path.insert(0, "")
    _locals = {"exit": sys.exit, "__file__": stdin.name}
    interpreter = code.InteractiveInterpreter(locals=_locals)
    [interpreter.runsource(line) for line in stdin]
    sys.exit(0)

def main(argv=None):
    """Business logic. respond to argv after parsing"""
    args = parse_args(argv)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)
```

First we add the `run_stdin()` function which will handle `stdin` for us then we add the call to `run_stdin()` to our `main` function.

The `run_stdin()` function performs the following steps:

1. we change `sys.argv[0]` to an empty string and insert an empty string as the first entry to sys.path
2. setup some locals for the interpreter.
    1. we need `exit()` to be defined
    2. we need `__file__` to point to the `stdin.name`
3. We set up an interpreter with `_locals`
4. We run through the lines of `stdin` and have the interpreter execute them. (I have an sub-optimal way of executing these within a list comprehention (sub-optimal because it generates a list which we immediatly discard), but we have to make some sacrifices to maintain the 100 lines of code requirement)
5. We exit

Next, let's add support for `-m`!

### Adding support for `-m`

So, while Python (and moonpy) supports executing a module be passing the directory name in as `file`, there is another use case which we need to support and that is the `-m`. The `-m` has very similar functionality except that instead of having the directory name, it operates on the module name (and searches `sys.path` for it).

Check out the following code:

```python
def run_m(args):
    """Run a module, just like python -m <module_name>"""
    sys.argv = sys.argv[sys.argv.index("-m"):]
    sys.argv.remove(args.m)
    if not len(sys.argv): sys.argv.append("")
    runpy.run_module(args.m, run_name="__main__", alter_sys=True)
    sys.exit(0)

def main(argv=None):
    """Business logic. respond to argv after parsing"""
    args = parse_args(argv)
    if args.m:
        run_m(args)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", action="store_true")
    parser.add_argument("-m", type=str, nargs="?")
    parser.add_argument("file", type=str, default="-", nargs="?")
    parser.add_argument("args", nargs=argparse.REMAINDER)
    args = parser.parse_args(argv)
    args.file = sys.stdin if args.file == "-" else args.file
    return args
```

First, we define a function called `run_m` which will handle the `-m` for us. It performs the following actions:

1. adjusts `sys.argv` and `sys.path` (some of this functionality is handled by `runpy.run_module`)
    1. `sys.argv` is adjusted by removing all arguments up to `-m` and the actual value passed in to `-m` is removed
    2. `sys.path` is prepended with the current working directory
2. The module is run
3. We exit

In the `main()` function we call `run_m()` if `args.m` is present. The function calls `sys.exit(0)` so execution will stop at that point.

Now we need to add support for `-c`!

### Adding support for `-c`

One of the most useful options Python provides is the ability to specify a Python command using the `-c` option. This allows us to do things like this (this is a trivial example) `python -c "import sys; print sys.path"`.

We can add this functionality with the following code changes:

```python
def run_c(args):
    """Run a command, just like python -c <command>"""
    sys.argv = sys.argv[sys.argv.index("-c"):]; sys.argv.remove(args.c)
    sys.path.insert(0, os.path.abspath("."))
    interpreter = code.InteractiveConsole()
    interpreter.runsource(args.c)
    if args.i:
        interpreter.locals["exit"] = sys.exit; interact(interpreter)
    sys.exit(0)

def main(argv=None):
    """Business logic. respond to argv after parsing"""
    args = parse_args(argv)
    if "-m" in sys.argv and "-c" in sys.argv:
        func = run_m if sys.argv.index("-m") < sys.argv.index("-c") else run_c
        func(args)
    elif args.c: run_c(args)
    elif args.m: run_m(args)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", action="store_true")
    parser.add_argument("-c", type=str, nargs="?")
    parser.add_argument("-m", type=str, nargs="?")
    parser.add_argument("file", type=str, default="-", nargs="?")
    parser.add_argument("args", nargs=argparse.REMAINDER)
    args = parser.parse_args(argv)
    args.file = sys.stdin if args.file == "-" else args.file
    return args
```

We added some more logic to `main` here, mainly because Python will honor whichever comes first (out of `-m` or `-c`) and argparse does not act this way so we have to workaround it. Then you need to read the `elif` statements carefully because they were collapsed to one line (I love the 100 lines thing).

The `run_c()` function does several things, first, sets `sys.argv` to `["-c",...]` where `...` is everything except the actual command to be executed. This is intended to match the behavior of Python. Next, the current directory is added to `sys.path`. Next it sets up an interpreter and runs the command which was passed in. Finally, dropping us into an interpreter if `-i` was provided.

Next up is adding support for `-V/--version`!

### Adding `-V/--version` suport.

This feature is very easy to implement. Check out the following code changes:

```python
__version__ = "moonpy 0.6.0"

def main(argv=None):
    """Business logic. respond to argv after parsing"""
    args = parse_args(argv)
    if args.version: print __version__; sys.exit(0)
    if "-m" in sys.argv and "-c" in sys.argv:
        func = run_m if sys.argv.index("-m") < sys.argv.index("-c") else run_c
        func(args)
    elif args.c: run_c(args)
    elif args.m: run_m(args)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)

def parse_args(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", action="store_true")
    parser.add_argument("-V", "--version", action="store_true")
    parser.add_argument("-c", type=str, nargs="?")
    parser.add_argument("-m", type=str, nargs="?")
    parser.add_argument("file", type=str, default="-", nargs="?")
    parser.add_argument("args", nargs=argparse.REMAINDER)
    args = parser.parse_args(argv)
    args.file = sys.stdin if args.file == "-" else args.file
    return args
```

First, At the top of the file we set up a `__version__` variable, this is easy to do, and many other Python tools will respect it. Perhaps in the future I will add a more dynamic way to determine the version of moonpy, but for now, this is only one line to change for each release which isn't too bad.

Next in the `main()` function, we add a line which, if args.version is True, prints `__version__` and exits.

Finally we add the option to our command line parser.

Next up we are going to enhance the Python interpreter with cross-session history support and tab-completion. This is mainly so I can say "an enhanced Python interpreter in 100 lines of code"!

### Adding history support and tab-completion

So it appears that Python provides support for history, but only within the current session (unless you provide a startup script like [this stackoverflow answer](http://stackoverflow.com/a/12334344/2723675) suggests), but it does not provide tab-completion (The answer linked does provide the tab completion). In order to claim that my project has an enhanced Python interpreter, I am prviding this functionality by default.

To see how this is done, check out the code changes below:

```python
def interact(console=None):
    """Launch an enhanced, interactive Python interpreter"""
    set_up_history()
    sys.argv = [""]
    if not console:
        console = code.InteractiveConsole(locals={"exit": sys.exit})
    console.interact("MoonPy - m-o-o-n that spells Python")

def set_up_history():
    """Taken from https://docs.python.org/2/library/readline.html#example"""
    try: import readline
    except ImportError: import pyreadline as readline
    else: import rlcompleter; readline.parse_and_bind("tab: complete")
    histfile = os.path.join(os.path.expanduser("~"), ".pyhist")
    try: readline.read_history_file(histfile)
    except IOError: pass
    atexit.register(readline.write_history_file, histfile)
```

This is a little difficult to read (100 lines of code, remember), but it essentially boils down to this:

1. when `interact()` is called it now calls `set_up_history()`
2. `set_up_history()` tries to import some things which will enable history
3. then `set_up_history()` calls a couple of funstions (`readline.parse_and_bind()` and `read_history_file()`) which actually do the work of setting up tab-completion and cross-session history

Now, let's add support for a `site-packages` directory!

### Adding support for `site-packages`

So, this feature we are departing from the way Python handles things a little bit. If you don't know the `site-packages` directory is where you're site-specific or third-party modules are installed. The location of this directory is different depending on your platform, but with moonpy, we will require that this directory be in the same directory as the moonpy executable itself. This is meant to simplify things a bit.

In order to facilitate this feature, we need to make the following changes to our `main` function:

```python
def main(argv=None):
    """Business logic. respond to argv after parsing"""
    here = os.path.dirname(os.path.abspath(__file__))
    if "site-packages" in os.listdir(here):
        sys.path.insert(0, os.path.abspath(os.path.join(here, "site-packages")))
    args = parse_args(argv)
    if args.version: print __version__; sys.exit(0)
    if "-m" in sys.argv and "-c" in sys.argv:
        func = run_m if sys.argv.index("-m") < sys.argv.index("-c") else run_c
        func(args)
    elif args.c: run_c(args)
    elif args.m: run_m(args)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)
```

That's it! Basically if the directory exists, it is prepended to the path.

Well, That's it! Everything we set out to do is done.

### Conclusion

OK, so this project isn't done by any means. I will try to write a blog post for each significant change, but I cannot promise anything. This (and all of my other blog posts) are part of a [github project](https://github.com/iLoveTux/ilovetux.github.io) If you notice any mistakes, omissions or typos please open an issue in the [issue tracker](https://github.com/iLoveTux/ilovetux.github.io/issues).

For your convenience, the code is presented in it's entirety is listed below:

```python
#!/usr/bin/env python
import os
import sys
import code
import runpy
import atexit
import argparse

__version__ = "moonpy 0.6.0"

def set_up_history():
    """Taken from https://docs.python.org/2/library/readline.html#example"""
    try: import readline
    except ImportError: import pyreadline as readline
    else: import rlcompleter; readline.parse_and_bind("tab: complete")
    histfile = os.path.join(os.path.expanduser("~"), ".pyhist")
    try: readline.read_history_file(histfile)
    except IOError: pass
    atexit.register(readline.write_history_file, histfile)

def run_path(args):
    """Run a path (script, zip file, or dir) just like python <script>"""
    sys.argv.pop(0)
    _locals = runpy.run_path(args.file, run_name="__main__")
    if args.i:
        interact(code.InteractiveConsole(locals=_locals))

def run_m(args):
    """Run a module, just like python -m <module_name>"""
    sys.argv = sys.argv[sys.argv.index("-m"):]
    sys.argv.remove(args.m)
    if not len(sys.argv): sys.argv.append("")
    runpy.run_module(args.m, run_name="__main__", alter_sys=True)
    sys.exit(0)

def run_c(args):
    """Run a command, just like python -c <command>"""
    sys.argv = sys.argv[sys.argv.index("-c"):]; sys.argv.remove(args.c)
    sys.path.insert(0, os.path.abspath("."))
    interpreter = code.InteractiveConsole()
    interpreter.runsource(args.c)
    if args.i:
        interpreter.locals["exit"] = sys.exit; interact(interpreter)
    sys.exit(0)

def run_stdin(stdin):
    sys.argv[0] = ""; sys.path.insert(0, "")
    _locals = {"exit": sys.exit, "__file__": stdin.name}
    interpreter = code.InteractiveInterpreter(locals=_locals)
    [interpreter.runsource(line) for line in stdin]
    sys.exit(0)

def interact(console=None):
    """Launch an enhanced, interactive Python interpreter"""
    set_up_history()
    sys.argv = [""]
    if not console:
        console = code.InteractiveConsole(locals={"exit": sys.exit})
    console.interact("MoonPy - m-o-o-n that spells Python")

def main(argv=None):
    """Business logic. respond to argv after parsing"""
    here = os.path.dirname(os.path.abspath(__file__))
    if "site-packages" in os.listdir(here):
        sys.path.insert(0, os.path.abspath(os.path.join(here, "site-packages")))
    args = parse_args(argv)
    if args.version: print __version__; sys.exit(0)
    if "-m" in sys.argv and "-c" in sys.argv:
        func = run_m if sys.argv.index("-m") < sys.argv.index("-c") else run_c
        func(args)
    elif args.c: run_c(args)
    elif args.m: run_m(args)
    if args.file is sys.stdin:
        if args.file.isatty():
            interact()
        else:
            run_stdin(args.file)
    else:
        run_path(args)

def parse_args(argv=None):
    """parse argv"""
    argv = sys.argv[1:] if argv is None else argv
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", action="store_true")
    parser.add_argument("-V", "--version", action="store_true")
    parser.add_argument("-c", type=str, nargs="?")
    parser.add_argument("-m", type=str, nargs="?")
    parser.add_argument("file", type=str, default="-", nargs="?")
    parser.add_argument("args", nargs=argparse.REMAINDER)
    args = parser.parse_args(argv)
    args.file = sys.stdin if args.file == "-" else args.file
    return args

if __name__ == "__main__":
    main()
```

There you go, a Python interpreter in 96 lines of code!

See you next time, Happy Coding!

