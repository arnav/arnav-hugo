---
lastmod:     2017-10-15
date:        "2017-10-15"
title:       "Useful Executable Modules in the Python Standard Library"
description: "Useful tools and modules in the Python Standard Library"
tags:        [ "python", "programming", "examples" ]
categories:  [ "Python" ]
series:      [ "Python in the real world" ]
slug:        "python-executable-modules"
project_url: ""
draft:       false
comment:     true
---
Python comes with many handy tools that can make our lives as developers or sysadmins easier. These tools are in the form of modules and libraries that are also executable. Many of these tools are known, but not all are as well known as they should be. I will mention a few useful tools that I have found in this post.

## How to write an executable Python script
First, for beginners, a quick introduction to how to write executable scripts in Python. It is actually really easy.

We can simply run a python script from the command line by prepending `python ` to the name, such as `python <script>`.

To run a module which is present in the current PYTHONPATH, you can run from command-line too
```sh
$ python -m <module>
```

Adding `python -m` to your script everytime can be tedious sometimes, so in Unix you can tell the shell how to execute your script. This is done by specifying the executing binary path in the first line of the script by appending `#!` (aka [she-bang](https://en.wikipedia.org/wiki/Shebang_(Unix))) to the command and then simply running the script.

```py
#!/usr/bin/python
```
or better
```py
#!/usr/bin/env python
```

When Python executes a script it runs the code top-down line by line. All the functions and classes at the top level of the script will get compiled and any module level statements will be executed. This process is the same as when Python imports a module from another module.

If you want to write code that **only** executes when the module is run as a script, you can write it in a `if __name__ == '__main__'` block as below.

The [argparse](https://docs.python.org/3/library/argparse.html) module in stdlib can be used to parse command-line parameters and ensuring that the interface is clearly specified.

~~~py
import argparse

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('-o', '--output')
    parser.add_argument('-v', dest='verbose', action='store_true')
    args = parser.parse_args()
    # ... do something with args.output ...
    # ... do something with args.verbose ..
~~~

Now let's look at some interesting and useful runnable modules in the Python Standard Library...

## File sharing
A very useful tool is the HTTP Server module. This module can be run to allow sharing of local file over your internal network easily using Python. To start a web server and serve files in current directory simply run the following command...

~~~sh
# Python 3
$ python3 -m http.server

# Python 2
$ python2 -m SimpleHTTPServer
~~~

Now the files in local directory are it's subdirectories are visible on `http://localhost:8000/`. Others in the local network can access the files by replcating *localhost* with our machine's IP address.

I have used this in the past to quickly share files with collegues where a file share was not readily available and the file is too big to send over email.

## JSON pretty printing
[json.tool](https://docs.python.org/3/library/json.html#module-json.tool) is a handy way of pretty printing and validating JSON format files from the command line.

~~~
$ echo '{"json":"obj"}' | python -m json.tool
{
    "json": "obj"
}
$ echo '{ 1.2:3.4}' | python -m json.tool
Expecting property name enclosed in double quotes: line 1 column 3 (char 2)
~~~

I have used JSON tool in the past to debug integration with HTTP APIs that serve JSON results.

## Debugging
The [Python debugger `pdb`](https://docs.python.org/3/library/pdb.html) makes it very easy to debug issues with Python scripts. Simply run the target script using `pdb` module rather than directly running it. If an unhandled exception is raised the debugger drops in a debug shell allowing us to run a post-mortem analysis by inspecting state and variables.

```sh
$ python -m pdb script.py
```

If you want the debugger to stop at a particular point in execution, simply add the below statement above it in the code.
```python
import pdb; pdb.set_trace()
```

## Performance analysis

**Timing**

The [timeit](https://docs.python.org/3/library/timeit.html) module is an easy way to time a piece of code. The module can run some setup code (import string), and then run test code many (default 10,000,000) times to time execution of the code.

```sh
$ python -m timeit -s 'text = "sample string"; char = "g"'  'char in text'
10000000 loops, best of 3: 0.0408 usec per loop
$ python -m timeit -s 'text = "sample string"; char = "g"'  'text.find(char)'
10000000 loops, best of 3: 0.195 usec per loop
```
Some useful [command-line options](https://docs.python.org/3/library/timeit.html#command-line-interface) are:

* `-n` - how many times to repeat the statement (default 10M)
* `-r` - how many times to repeat the timer (default 3)
* `-s` - setup statement to be executed once before the test

**Profiling**
`cProfile` module makes it easy to measure the time spent in executing a script and pinpoint the slow bits.
~~~sh
$ python cProfile scriptfile [arg] ...
~~~
A couple of useful flags are:

* `-o` - output file path
* `-s` - sort output

## Running tests - Doctests and Unit tests

Python [doctests](https://docs.python.org/3/library/doctest.html#module-doctest) can be run from the command-line using the doctest executable module.
```sh
$ python -m doctest -v example.py
```

Similarly Unit tests can executed using the [unittest](https://docs.python.org/3/library/unittest.html#command-line-interface) module

```sh
$ python3 -m unittest
```
This very useful command-line tool will scan the current directory and sub-modules to discover tests and run them. We can also run specific modules or functions by specifying them. Look at the various options [here](https://docs.python.org/3/library/unittest.html#command-line-interface).

## Working with archives

**Creating and opening Zip and TAR archive files**

In case you don't have `tar` or `Zip` tools handy, in Python 3 [tarfile](https://docs.python.org/3/library/tarfile.html#command-line-interface) and [zipfile](https://docs.python.org/3/library/zipfile.html#command-line-interface) modules allow us to bundle directories into archives and open existing ones.

```sh
# Create a new TAR archive
$ python3 -m tarfile -c <tarname>.tgz <file> <file>

# Extract from an existing TAR archive
$ python3 -m tarfile -e <tarname>.tgz
```


**Making executable Zip files**

In Python 3, the [zipapp](https://docs.python.org/3/library/zipapp.html#command-line-interface) module also allows us to pack up a directory into an archive, and makes it executable. When run, the archive will execute the `main` function in the `myapp` module.
~~~sh
$ python3 -m zipapp myapp
$ python3 myapp.pyz
<output from myapp>
~~~

## Other useful and interesting modules

### Opening a web page in a Browser locally
The [webbrowser](https://docs.python.org/3/library/webbrowser.html#module-webbrowser) module has a programatic as well as command-line interface.

```sh
$ python -m webbrowser -t http://www.yahoo.com
```

### Base64 encoding and decoding
When working with a REST APIs, especially where authentication tokens are involved, use of Base64 encoding is quite common. Python [`base64`](https://docs.python.org/3/library/base64.html#module-base64) module can be used from the command-line a tool in the shell as well.

```sh
$ python -m base64 -e
```

### Calendar
Did you know that Python comes with an in-built text calendar? The [calendar](https://docs.python.org/3/library/calendar.html) module is executable and can take many parameters that allow for customised display.

```sh
$ python -m calendar
                                  2017

      January                   February                   March
Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su
                   1             1  2  3  4  5             1  2  3  4  5
 2  3  4  5  6  7  8       6  7  8  9 10 11 12       6  7  8  9 10 11 12
 9 10 11 12 13 14 15      13 14 15 16 17 18 19      13 14 15 16 17 18 19
16 17 18 19 20 21 22      20 21 22 23 24 25 26      20 21 22 23 24 25 26
23 24 25 26 27 28 29      27 28                     27 28 29 30 31
30 31

       April                      May                       June
Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su
                1  2       1  2  3  4  5  6  7                1  2  3  4
 3  4  5  6  7  8  9       8  9 10 11 12 13 14       5  6  7  8  9 10 11
10 11 12 13 14 15 16      15 16 17 18 19 20 21      12 13 14 15 16 17 18
17 18 19 20 21 22 23      22 23 24 25 26 27 28      19 20 21 22 23 24 25
24 25 26 27 28 29 30      29 30 31                  26 27 28 29 30

        July                     August                  September
Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su
                1  2          1  2  3  4  5  6                   1  2  3
 3  4  5  6  7  8  9       7  8  9 10 11 12 13       4  5  6  7  8  9 10
10 11 12 13 14 15 16      14 15 16 17 18 19 20      11 12 13 14 15 16 17
17 18 19 20 21 22 23      21 22 23 24 25 26 27      18 19 20 21 22 23 24
24 25 26 27 28 29 30      28 29 30 31               25 26 27 28 29 30
31

      October                   November                  December
Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su      Mo Tu We Th Fr Sa Su
                   1             1  2  3  4  5                   1  2  3
 2  3  4  5  6  7  8       6  7  8  9 10 11 12       4  5  6  7  8  9 10
 9 10 11 12 13 14 15      13 14 15 16 17 18 19      11 12 13 14 15 16 17
16 17 18 19 20 21 22      20 21 22 23 24 25 26      18 19 20 21 22 23 24
23 24 25 26 27 28 29      27 28 29 30               25 26 27 28 29 30 31
30 31
```

### Print system configuration
[Sysconfig](https://docs.python.org/3/library/sysconfig.html#using-sysconfig-as-a-script) module allows you to print the detailed system configuration including environment variables which might be useful for debugging purposes.

```sh
$ python -m sysconfig
Platform: "macosx-10.6-intel"
Python version: "3.6"
Current installation scheme: "posix_prefix"

Paths:
    data = "/Library/Frameworks/Python.framework/Versions/3.6"
    include = "/Library/Frameworks/Python.framework/Versions/3.6/include/python3.6m"
    platinclude = "/Library/Frameworks/Python.framework/Versions/3.6/include/python3.6m"
    platlib = "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages"
    platstdlib = "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6"
    purelib = "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages"
    scripts = "/Library/Frameworks/Python.framework/Versions/3.6/bin"
    stdlib = "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6"

Variables:
    ABIFLAGS = "m"
    AC_APPLE_UNIVERSAL_BUILD = "1"
    AIX_GENUINE_CPLUSPLUS = "0"
    ANDROID_API_LEVEL = "0"
    ...
```
