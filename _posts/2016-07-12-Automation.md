---
layout: post
title: Automation 
---

There are several types of automation each with its own set of unique
challenges. In this paper, I will attempt to outline these categories of
automation tasks along with some ways to streamline the process of automating
these things.

## The Types of Automation

Automation, in IT, can be broken down into one of three types of tasks.
Workflow Automation is the process of automating a set of tasks which routinely
take up a significant portion of your workday such as taking backups of your
systems. Responsive Automation is the process of automating the response to
certain environmental conditions like rebooting a machine if CPU usage peaks 
above 90% for 30 seconds or more. Opportunistic Automation is a technique suited
for tasks which fit into either of the other categories with an additional
property, it isn't necessarily time-sensitive or critical so it can be done
at the most opportune time. Getting to know these types of automation can help
you break down the tasks you need to automate and can guide you towards certain
tool-sets which might better facilitate your goals.

## Workflow Automation

Workflow Automation is what you might think of when someone mentions automation.
The premise is pretty simple, you have a process you must follow to complete a
task. Manually following this process can become time-consuming, so you decide
to automate the whole process in order to get your involvement down to a
minimum. There are a number of tools and techniques you may be familiar with if
you have tackled this type of automation before.

The most common tool for this type of task is usually a shell script or batch
file. A shell script or batch file is great because they can be interpreted
by a default installation of your OS. I would however, recommend using the
Python programming language for these tasks as it would make it a lot easier
to maintain and improve these scripts over time.

Another great tool for these tasks is your OSs scheduler. Linux has cron and
Windows has task scheduler, most other Operating Systems have their own take on
one of these tools. Their purpose is simple, run a program periodically and they
all have their own ways of defining their behavior. It would behoove you to
learn how to proficiently use the scheduling tool which ships with your OS.

## Responsive Automation

Responsive Automation is what you might think of if some piece of software
provides mechanisms to "alert" on certain conditions. Splunk is one piece of
software which makes it simple enough to monitor inputs and alert on certain
conditions. Sometimes the goal is to simply make one or more person aware of
the situation, in this case you might call this type of automation "monitoring".
There is, however, another way you might aproach this.

In addition to "alerting" on certain conditions, you can automate the response
to certain conditions. This is also possible in Splunk but adds another layer of
complexity. I usually tackle this problem in one of two ways, either
real-time or periodically. With real-time responsive automation, you usually
have a service up and running in the background (maybe using a product like
Splunk or maybe with a home-spun service) monitoring for certain conditions
in real-time (or close to it). With periodic responsive automation, you usually
have a scheduled task which checks conditions periodically and responds when it
is run.

I have found Python to be incredibly useful here as well because it is pretty
simple to create either a background service or a command line tool to enable
either type of responsive automation. It also can stand alone without dependency
on any other software.

## Oportunistic Automation

With Oportunistic Automation, you have pretty much any other type of automation,
but the task is not critical or time-sensitive, so they can be performed
oportunistically like at 3 am when nothing else is happening. The main tennant
of this type of automation is that it can run and the results don't necessarily
need to be known right away. These tasks can offer tons of flexibility and
freedom and can apply to many different circumstances.

One example of oportunistic automation I recently tackled was to generate an
aggregate report of statistics about the previous day. While real-time
monitoring was already in place, I was able to generate a dashboard at 1 am that
would be used by many people in the organization throughout the day, but by
only running the computationally-intensive report once per day at a low-volume
time I was able to free up resorces during peak time to handle the more
critical real-time monitoring.

This can be done with just about any automation tool, but for consistencies
sake, I would recomend Python once again. This means that Python can be a great
fit for just about any automation task as it is incredibly versitile, has a
very readable syntax and will integrate into just about any process or workflow
you can throw at it.

# About Python as an Automation Platform

Python is uniquely qualified to assist with automating tasks. The main benefits
of choosing Python are:

* Lower development and debugging time
* More readable, maintainable code
* Batteries included standard library
* Tons of great 3rd party libraries

Some libraries to look at for automation:

* paramiko (a native Python implementation of SSHv2 protocol)
* pywin32 (for interacting with Windows)
* selenium (for automating web browsers)
* PyAutoGUI (Controlling keyboard and mouse from Python)
* Openpyxl (Automate reading/writing Excel spreadsheets)
* fabric (automate administration tasks)
* chef (automate provisioning)
* Pandas (for manipulating tabular data)

There are literally tons more to check out. You can usually get by with just a 
simple web search for "Python TASK" where TASK is what you are currently
trying to do.

# Some closing thoughts

Automation has been a part of IT since the beginning, first with C then with
bash and now with Python. A lot of research and work has gone into automation
and it would behoove anyone to use the tools which make the job easier. With
a modest investment of time, you can learn to use Python to effectively automate
most if not all of your work away.

As always, Happy Coding! See you next time.
