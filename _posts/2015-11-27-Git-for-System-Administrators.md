---
layout: post
title: Git for System Administrators
---

Git for System Administrators

Hello and Happy Thanksgiving, this is the belated fourth installment
of my blog where I will go over a slightly unconventional, but incredibly
powerful use for git the distributed version control system. Git was built
by Linus Torvalds and friends to help with Linux kernel development and
has since been used by millions of developers to help track changes to source
code, but at its core it tracks changes to text files not just source code.
This is one way that git has helped me and I thought I would share.

Most System Administrators would like to know when and why configuration
changes were made to their servers, and with a little setup git can help here.

### Starting out Simply

Let's take a simple example, you just wish to track changes to your local
system configuration. I will assume you are running Linux, but if you're
running Windows there are some ideas you will be able to walk away with.

So here is the basic idea, we will set up a local git repository to track
changes to the files in our `/etc` directory. We could easily use a remote
repository as well and if this was a production system I would definitely
recommend doing so, but for simple illustrative purposes we can skip that
part. My next blog post will probably be about using docker along with a
remote git repository for configuration management, but for now we will
just look at a few of the features git has which will be incredibly useful
in debugging malfunctioning systems and for simply keeping track of who
changed what when.

## Setting Up the Local Git Repository

In order to keep things simple we will set up the local repository in our
`/etc` directory itself. We could set it up somewhere else and had a tool
like rsync update our repo on changes, but unless there's some reason you
would like to keep things separate it will be a lot easier to keep track
of things right in the `/etc` directory.

__NOTE__: Since we are working in `/etc` permissions will become an issue,
so you can either run all of the commands with `sudo` (the easy option) or
change permissions on files paying special attention to security concerns
and what git needs to do it's job. I would recommend using sudo as this will
use the same sets of permissions to keep track of changes as implementing
changes.

```bash
$ cd /etc
$ git init
```

At this point we have an empty repository, but don't worry it didn't remove
or change any of your files it's just not tracking them yet. This is an
opportune time to specify any files you would like to ignore. If you
ignore them now then they won't be in your repository at all, otherwise
it can be a little frustrating to remove all copies of the file from your
repository (definitely not impossible or even really all that difficult).
But doing it now will save some frustration.

Only you will know what you want to ignore, but the process is fairly simple:

1. Find files you would like to ignore
2. Create a file in the root of your repository called `.gitignore` in our
case it will be `/etc/.gitignore`
3. Add [glob patterns](https://en.wikipedia.org/wiki/Glob_%28programming%29)
matching files you would like to ignore to the `.gitignore` file

That's it, so for instance if you'd like to ignore anything in `/etc/kernel`
you could issue the following command:

```bash
$ echo kernel/ >> .gitignore
```

or if you wanted to ignore `/etc/bashrc` you could issue the following command:

```bash
$ echo bashrc >> .gitignore
```

well you get the idea.

Once you have all the files you wish to ignore set up in the `.gitignore`,
we can add all other files to our local git repository with the following
commands (please note that everything is meant to be executed in the root
of our repository, or `/etc/` in our case, as git works off of our
`present working directory` or `pwd`):

```bash
$ git add *
$ git commit -m "Initial Commit"
```

The first command will add all changes (Our files are considered changes at this
point because git isn't tracking them yet) to be `staged`. Next the
`git commit` command actually commits the `staged` changes to the local
repository. That's it! Now git will be able to tell you if anything changes
from here on out and you can use it to track the whos, whens, whats and whys
of all future changes.

### Tracking Some Changes

So, we are going to make some changes to one of our configuration files and
track them using git.

First thing to do is check to see if anything has changed and that can be done
with the following command:

```bash
$ git status
```

If there are no changes, the output will look like this:

```
On branch master
nothing to commit, working directory clean
```

otherwise it will list new, modified and deleted files which are not tracked
in their current state. So let's change something and see how this works. On
Fedora 23, the system I'm using, you can disable root login from sshd by
changing the following line in `/etc/ssh/sshd_config`:

```
PermitRootLogin yes
``` 

to

```
PermitRootLogin no
```

This is a fairly common change to make for security reasons, so after making
that change (I used `sudo` and `vim`, but you can use your favorite text
editor or a carefully crafted `sed` command), so let's see how git sees this
change:

```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   ssh/sshd_config

no changes added to commit (use "git add" and/or "git commit -a")
```

So we have one modified file, let's see what changed:

```bash
$ git diff
diff --git a/ssh/sshd_config b/ssh/sshd_config
index 1861bde..e432c32 100644
--- a/ssh/sshd_config
+++ b/ssh/sshd_config
@@ -46,7 +46,7 @@ SyslogFacility AUTHPRIV
 # Authentication:
 
 #LoginGraceTime 2m
-PermitRootLogin yes
+PermitRootLogin no 
 #StrictModes yes
 #MaxAuthTries 6
 #MaxSessions 10

```

So, it has us removing the one line `PermitRootLogin yes` and adding the line
`PermitRootLogin no`, this is how git sees the changes, line by line not
character by character, this scheme makes reading the reports a little easier.
Next we need to stage and commit the change so it is officially recorded.

We can do that with the following commands:

```bash
$ git add ssh/sshd_config
$ git commit -m "Disallow root login via ssh"
```

and now, git knows that this change is officially `commit`ed.

### Reviewing Changes

Now that we have history of changes (although small) we can review the
change history with the following command:

```bash
$ git log
commit cc46d88f1e193472c748187c629ddb8c6640424e
Author: root <root@localhost.localdomain>
Date:   Fri Nov 27 11:13:35 2015 -0500

    Disallow ssh root login

commit 50120a579dcf2ebc16e5ac399e380656ca6e5224
Author: root <root@localhost.localdomain>
Date:   Fri Nov 27 10:32:39 2015 -0500

    Initial commit

```

You can note the date and time is recorded along with the author (which is root
since I am performing these commands with `sudo`) and the message which we
passed in with the `-m` option to `git commit`. Very informative commit
messages will go a long way when tracking down a mis-configuration or bug
in one or more of your systems.

We can also look at the differences between commits, simply add the to and from
commit ids to the `git diff` command like so:

```bash
$ git diff 50120a579dcf2ebc16e5ac399e380656ca6e5224 cc46d88f1e193472c748187c629ddb8c6640424e
diff --git a/ssh/sshd_config b/ssh/sshd_config
index 1861bde..e432c32 100644
--- a/ssh/sshd_config
+++ b/ssh/sshd_config
@@ -46,7 +46,7 @@ SyslogFacility AUTHPRIV
 # Authentication:
 
 #LoginGraceTime 2m
-PermitRootLogin yes
+PermitRootLogin no 
 #StrictModes yes
 #MaxAuthTries 6
 #MaxSessions 10
```

yeah that's the same differences as before, but that's because we're looking
at the same change, this command can be incredibly useful when looking at
older changes.

### Wrapping it Up

That's it for today, perhaps soon I will do a write-up on using this idea
along with [paramiko](https://github.com/paramiko/paramiko)
and [dulwich](https://github.com/jelmer/dulwich) to automate our workflow
with Python.

See you soon.

Happy Coding!
