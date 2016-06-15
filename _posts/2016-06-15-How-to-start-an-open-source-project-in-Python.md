---
layout: post
title: How to start an open source project in Python
---

A quick look at how to make the most of your time when writing open source Python code.

### What is this all about?

I have been programming in Python for a few years now and this is a collection of useful tools and processes to follow to make it as easy as possible to maintain your software. I had to stumble upon these things one-by-one each one teaching me how the great Python masters seemed to be performing their miracles.

### So how do they do it?

In a word, automation. There are a number of tools which make things amazingly easy as long as you adhere to their guidlines. For instance, writing docstrings in ReStructured Text makes it almost trivial to auto-generate great documentation.

### So what are the parts that can be automated?

Basically, everything, but there are a few key areas where you can get the best cost-benefit scenarios:

* setup.py
* Makefile
* Documentation
* Testing
* Version Control
* collaboration

Let's take a look.

### setup.py

The power of setup.py is immense, I could barely believe it when I learned all the stuff you could do with setup.py. You can run your tests, you can build and upload to PyPI, you can install your package, you can manage dependencies and much, much more. I will be writing about these capabilities in a blog post pretty soon, but take my word for it, study setuptools and learn to use this small Python script to the fullest extent of its abilities

### Makefile

As awesome as setup.py is, you can't match the maturity and power of make. Using make to drive setup.py along with other commands is a little known secret for productive programmers. Using make also makes it quite easy to add custom commands.

### Documentation

Using Sphinx, you can generate pretty awesome documents. There are many themes to choose from as well as plugins. And just by using ReStructured Text in your docstrings, you get auto-generated API documentation. You can use Sphinx to generate many formats of documents from epub to man pages or from PDF to static Web site.

### Testing

Use a test driver such as nose or py.test. That is the long and short of it. But be sure to include your test command and test dependencies in your setup.py to let people
be able to run `python setup.py test`. Also, test against multiple versions of Python and use a Continuous Integration service such as Travis CI to handle all of this stuff for you.

### Version Control

Just use GitHub.

### Collaboration

Again, just use GitHub


Remember to leave a comment below.

Have fun and Happy coding!
