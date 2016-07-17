---
layout: post
title: Workflow Automation 
---

A more detailed look at Workflow Automation. In this paper, I will look into
how you can consistently release rock-solid automation to an ever-changing
workflow.

# The Problem

Your IT workflow changes day-to-day. In some cases because of newly available
insight or because of transfers of responsibility within an organization.
Regardless of how these tasks came to you, it is helpful to make it as simple
as possible for you. One way to do this consistently is to use the same method
to automate whatever you can. This is where Python comes in.

# The Solution

Python is a strongly-typed, dynamic multi-paradigm programming language. If
that sounds complex, it can also be read as "a dependable, convenient and
helpful programming language". Most tasks which assist in workflow automation
follow one of three common scenarios. Data Aggregation consists of data
ingestion, sanitization and presentation of data in new and useful ways. System
Administration consists of tasks which need to be performed as part of either
maintenance, migrations or auditing. Testing consists of running short, quick
routines to assure you that applications and services work as expected. With
Python we can create quick, consistent and dynamic solutions to problems
we hit every day.

## Data Aggregation

Data Aggregation, as I'm using the term, encompases the life-cycle of data
from just after birth to long-term storage and beyond. Wherever your data
originates, it's helpful to have a consistent manner in which to deliver,
parse, analyze and visualize your data. I have found it helpful to use purpose
built solutions to many of these problems such as syslog for delivery, Splunk
or Anaconda for analysis, Splunk or awk for searching. Whatever your chosen
tools just about every professional can benefit from learning to use the tools
at their disposal.

One tool I really like is Graphite which aggregates and graphs numerical values.
It is open source and can scale from light to heavy usage. I have found awk
to be a great pairing for Graphite. Jupyter notebooks also fit great here,
they are very  capable in their own right and can be use along with graphite
quite easily. A different but related tool is openpyxl which is a library which
makes it simple to read and write Excel workbooks from within Python. 

## System Administration

System Administration encompases a broad range of activities which comonly 
consists of sending commands to local or remote systems. This can be aided by
tools such as fabric, saltstack, chef, puppet or paramiko. The important part is
to standardize and document everything you are automating and audit-trails are
always a good idea. Python provides a strong base in system administration,
learning about Sphinx and ReStructured Text can go a long way towards readable,
up-to-date and searchable documentation, and the Python logging module can go
a long way towards an audit trail.

## Testing

Software developers as a whole have come a long way towards automated testing,
which is great because if the software developers use it then it will probably
work pretty well. Testing, however, is best done when everyone keeps track
of and tests assumptions they routinely make. It is great to have a testsuite
come with an application or framework, but they cannot take into account
all of the assumptions people other than developers are making about your
environment. Python's unittest library works well, particularly when paired with
a test runner such as nose and is simple enough to figure out that
non-developers to get something that works. This can be as simple as pinging
each one of your servers or making sure some services are up and running.

If you are developing anything in Python, including a testsuite, you should
learn to use Sphinx and ReStructured Text to document your code.

# Conclusion

In the end all that really matters is that you have a system in place with which
to automate repetitive parts of your job, you can easily explain to another
person how and why you are doing these things and you can provide an audit trail
of everything you are doing, why and how.

As always, Happy Coding! See you soon.
