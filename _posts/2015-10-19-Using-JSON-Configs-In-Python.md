---
layout: post
title: Using JSON for Configuration in Python
---

This post is about configuration file handling in Python, and how I feel
about it. The views are completely subjective and debate is welcome in
the comment section below.

### Why not ConfigParser

The standard in the Python community when it comes to configuration files
is to use ConfigParser (or configparser in Python 3), and I can see why.
It is a robust tool allowing for a lot of features you might need.
Interpolation is one of these, it allows you to do something like this:

```ini
[heading]
home_dir: /home/user
app_dir: %(home_dir)s/.apps/app
```

This is a cool feature and I've used it for a number of my projects to
provide a nice set of defaulted options, but doesn't that just look a
little complicated?

It's not that complicated though and your users might even understand how to
use it, but do you really want to explain that the `s` means `string`...and
then what a string is to someone? Not me, that's what tutorials are for, but
you have to be willing to read tutorials and you have to be interested enough
to persue it. Has anyone else been burned by trying to help someone by writing
a magic little script for someone in payroll or marketing before? I can't be
the only one.

There's also another really cool, albeit lesser known feature of ConfigParser
which simplifies some deployment scenarios. Like configuration file precedence.
This is when you have 2 different directories (or files) which configure the
same thing. One is for default configuration, the other is for user-local
configuration. This helps out in a lot of ways, for example, if you want to
return to the default configuration you simply have to remove/rename/delete
the user-local configuration files. Also, this can simplify an update/upgrade
process by allowing the update to overwrite the default file(s) with new
defaults and new options while leaving the user's modifications alone. You'd
implement this feature like this:

```python
import ConfigParser

config = ConfigParser.SafeConfigParser()
config.read("default.conf")
config.read("user.conf")
```

When implemented this way, all sections, options and values from `default.conf`
are loaded then anything defined in `user.conf` will overwrite the default
values. This is pretty safe because if `user.conf` doesn't exist it will not
change anything. But then again it will not load anything if neither file
exists.

Like I said these are some pretty cool features and there are a couple more,
that I won't get into now, but what if this is all too much? From a coder's
standpoint `ConfigParser` provides a pretty nice easy API along with some
options which can be incredibly useful when you end up needing them.

But what if you just wanted something simple for both you and your user?

### Enter JSON

JSON (JavaScript Object Notation) is a very popular format for storing data,
and it even has a number of projects already using it for configuration. Also,
it's available in the Python Standard Library, so if you don't really need all
the features I described above and you want to present your users with a
somewhat simplified interface for configuration then JSON might be for you.

I can tell you from experience that for large-scale projects `ConfigParser`
is incredible, but for small one-off scripts to concatenate two of friends
spreadsheets together you may want to just go with a JSON configuration
file. The code would look like this:

```python
import json

with open("config.json", "r") as fin:
    config = json.load(fin)
```

and your configuration files would look like this:

```json
{
    "logFile": "/var/log/app.log",
    "logLevel": 10
}
```

at this point you would have a Python `dict` which contains two keys, one a
string and one an integer. Pretty simple.

Now, I know you might be saying "But now we can't use interpolation", and you'd
be correct. This approach makes things simpler on the end user simply by
removing features that might confuse them.

This has the added benefit of possibly centralising your configuration
(this would only make sense with an internal application), for example
if you wanted to retrieve your json config from a url, you could do something
like this:

```python
import urllib2
import json
from contextlib import closing

with closing(urllib2.urlopen('http://blah')) as fin:
    config = json.load(fin)
```

That's a little more code, and even more if you wanted to handle both
file and url configurations, but I've used it before.

Well, that's it for now. Check back soon, and Happy Coding!
