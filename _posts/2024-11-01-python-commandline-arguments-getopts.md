---
layout: post
title: "Python getopts"
date: 2024-11-01
categories: [python]
tags: [python]
---

Hello, 

Today, I would like to give you some ideas of how you can make your python scripts more flexible and control them with arguments. In this post I will show you a method, that is very easy to understand and works very well with simple scripts, that require some data to be passed into. This is first part of two-posts mini-series about passing argument to python script.

## Little disclaimer

Method presented in this post is not the most convenient, but is easy to understand for people who came from C or Bash background. This is also mentioned in Python documentation:

>The getopt module is a parser for command line options whose API is designed to be familiar to users of the C getopt() function. Users who are unfamiliar with the C getopt() function or who would like to write less code and get better help and error messages should consider using the argparse module instead. [source](https://docs.python.org/3/library/getopt.html#module-getopt)

If you are looking for another method, that is more elegant and suited for python, please look at another post in this series - [link]()

First of all, we need to understand how we can fetch arguments passed to script. In order to do that, we need module called **sys.argv**.

# Retrieving of arguments

**sys.argv** returns list af all passed arguments. Lets try how it works. Create a file with any name you like, I called mine "script.py".

```python
import sys

print "first argument:", sys.argv[0]
print "second argument:", sys.argv[0]
print "number of passed arguments:", len(sys.argv)
```

Now run this script and pass 2 arguments:

```bash
python script.py arg1 arg2
```

This is the output you should get:

```bash
first argument: argparse-test.py
second argument: arg1
number of passed arguments: 3
```

As you can see, first argument is the name of the script, this is how sys.argv works. We usually don't need it, so this is how we can skip it. We will modify our script and declare variable, that will store list of arguments returned by sys.argv. We will also skip the firs object from this list, this is how to do that:

```python
import sys

argv = sys.argv[1:]

print "first argument:", argv[0]
print "second argument:", argv[1]
print "number of passed arguments:", len(argv)
```

Now we know how to get list of arguments without the name of the script. We have to consider, how to parse provided arguments to make them useful. Let's introduce **getopt** module.

# How to define, which arguments will work with our script

I like to work on real-life examples, so let's assume, that we are trying to make simple diary or logging system. Our script will take three arguments - date, time and message text.

**getopt** works like this:

```python
opts, args = getopt.getopt(​argv, shortopts, longopts​)
```

Values, that are passed to the function:

* argv - list of arguments, that were passed to the script (ones, that we retrieve with sys.argv)
* shortopts - this is a declaration of short arguments that we want to use (ones that are provided with single dash, eg. **-a value**)
* longopts - this is a declaration of long arguments that we want to use (ones that are provided with double dash, eg. **--argument value**)

Returned  values:

* opts - returned list of arguments and corresponding values
* args - returned list of other, than defined arguments, that were passed to the script

It is worth to mention, that it doesn't mether in which order you will pass arguments, that were defined as **shortopts** or **longopts**. On the other hand, list returned and assigned to **args** variable contains so called positional arguments and their order matters. This is outside of the scope of this article, but if you need additional information, please read [this](https://problemsolvingwithpython.com/07-Functions-and-Modules/07.07-Positional-and-Keyword-Arguments/).


Getting back to our example, implementation of **getopt** would've look like this:

```python
import sys, getopt

argv = sys.argv[1:]

opts, args = getopt.getopt(argv, "d:t:m:", ["date=", "time=", "message="])
```


# Parsing given arguments

We already know how to declare arguments, now we need to find out, how to parse values passed to our script. It is done with simple for loop, that iterates over **opts** list and if statement, to "catch" proper values.

```python
for opt, arg in opts:
    if opt in ['-d', '--date']:
        date = arg
    elif opt in ['-t', '--time']:
        time = arg
    elif opt in ['-m', '--message']
        message = arg
```

# Testing

Let's merge our script all together and test how it works.

```python
import sys, getopt

argv = sys.argv[1:]

opts, args = getopt.getopt(argv, "d:t:m:", ["date=", "time=", "message="])

for opt, arg in opts:
    if opt in ['-d', '--date']:
        date = arg
    elif opt in ['-t', '--time']:
        time = arg
    elif opt in ['-m', '--message']:
        message = arg

print('date: {}'.format(date))
print('time: {}'.format(time))
print('message: {}'.format(message))
```

Run it with command as below, order of provided arguments is not important. Please remember to close message in quotes.

```
python getopt-playground.py -d 2020 -t 20:00 -m "some message"
python getopt-playground.py --date 2020 --time 20:00 --message "some message"
```


That's it, I hope it was clear. If you have any questions, please drop me a post below and remember to check second post in this series - [link]().
