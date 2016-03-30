---
layout: post
title: How to setup a QA server using Fedora 23 and tox
---

Tox is a tool which sits on top of virtualenv, the venerable Python
environment manager. Tox uses virtualenv to test your code-base against
multiple versions of Python. In this article, we will setup various
Python installations and tox to run tests against Python versions
2.7, 3.5.

The first step is to install all the prerequisites for building Python
ourselves because we would like to keep control of all the python
installations. We can do so with the following commands:

```bash
$ sudo dnf groupinstall "Development tools"
$ sudo dnf install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

We can do that with the following commands:

```bash

cd /tmp
wget https://www.python.org/ftp/python/3.5.1/Python-3.5.1.tgz
wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz

tar xvf Python-3.5.1.tgz
tar xvf Python-2.7.11.tgz

mkdir ~/python
cd Python-3.5.1 && ./configure --prefix ~/python/3.5 && make && make altinstall && cd ..
cd Python-2.7.11 && ./configure --prefix ~/python/2.7 && make && make altinstall && cd ..
```

So we will install tox into our 2.7 Python installation. This choice is
completely arbitrary, but if I run into problems I will update.

```bash
$ ~/python/2.7/bin/python -m ensurepip
$ ~/python/2.7/bin/pip install tox
```

Now we will have the tox command available in `~/python/2.7/bin/tox`. We could
execute tox by issuing the command with the path, but tox would not be able
to find the python executables. The solution to that is to add both Python's
`bin` directories to your `PATH`. We can do that with the following command:

```bash
$ export PATH=~/python/2.7/bin:~/python/3.5/bin:$PATH
```

Now we can run tox and it will find both Python interpreters. If you would
like to add these to your `PATH` permanently, you can add that above command
to your `~/.bashrc`.

So now we need to set up a project 






