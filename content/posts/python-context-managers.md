---
lastmod:     2016-02-22
date:        "2016-02-20"
title:       "Python in the real world: Context Managers"
description: "Using examples to explore how Python Context Managers are used in the real world."
tags:        [ "python", "programming", "examples" ]
categories:  [ "Python" ]
series:      [ "Python in the real world" ]
slug:        "python-context-managers"
project_url: ""
draft:       false
comment:     true
---



Context Managers are one of the core language features that make Python unique. The `with` statement allows developers to write a common programming pattern in a concise and readable way. The following indented block gives a visual cue and make understanding the code easier. Understanding Context Managers is key to understanding the idea of *Pythonic* code.

Context Managers are usually used for allocation and releasing of resources, but that is not their only use-case. They are useful for factoring out common setup and teardown code, or any pair of operations that need to be performed before or after a procedure.

In this article, I will present some interesting real-world examples of their use, and hopefully encourage budding Pythonistas to explore them.
<!--more-->

## Context Managers in summary

In simple terms, Context Managers make writing **try-finally** blocks easier. 

~~~python
# In Python, code with common setup and teardown:

setup()
try:
    do_work()
finally:
    teardown()

# can be translated roughly to:

cm = ContextManager()
obj = cm.__enter__()
try:
    do_work(obj)    # use obj as needed (see examples below)
finally:
    cm.__exit__()

# this can be written more succinctly using the `with` statement:

with ContextManager() as obj:
    do_work()

~~~

The expression following the `with` keyword should return an object that follows the Context Manager protocol. This can be a class instantiation or a function call that returns the Context Manager object.

The object returned must follow the **Context Manager protocol** which specifies two special methods, `__enter__` and `__exit__`.

Some important points to remember are:

* `__enter__` should return an object that is assigned to the variable after `as`. By default it is `None`, and is optional. A common pattern is to return `self` and keep the functionality required within the same class.
* `__exit__` is called on the original Context Manager object, not the object returned by `__enter__`.
* If an error is raised in `__init__` or `__enter__` then the code block is never executed and `__exit__` is not called.
* Once the code block is entered, `__exit__` is always called, even if an exception is raised in the code block. 
* If `__exit__` returns `True`, the exception is suppressed.


~~~python
class CManager(object):
    def __init__(self):
        print '__init__'
    
    def __enter__(self):
        print '__enter__'
        return self
    
    def __exit__(self, type, value, traceback):
        print '__exit__:', type, value
        return True  # Suppress this exception

    def __del__(self):
        print '__del__', self

with CManager() as c:
    print 'doing something with c:', c
    raise RuntimeError()
    print 'finished doing something'
print 'done something'

"""
# outputs:
__init__
__enter__
doing something with c: <__main__.CManager object at 0x103fd8ed0>
__exit__: <type 'exceptions.RuntimeError'> 
done something
"""
~~~


Simple context managers can also be written using Generators and the `contextmanager` decorator:

~~~python
from contextlib import contextmanager

@contextmanager
def context_manager_func():
    setup()

    # yield must be wrapped in try/finally
    # to trap any exceptions raised in the calling code
    try:
        yield
    finally:
        teardown()
~~~

## Reliable destructors

Context Managers give us a reliable method to clean up resources. Unlike other OO languages like C++ and Java, the Python destructor method `__del__` is not always guaranteed to be called. It is only called when the reference count to the object reaches zero. This might happen at the end of the current function, or at the end of the program, or never in case of circular references.

Up until Python 3.4, if all objects in a reference cycle have `__del__` method, Python would not call it, and garbage collect the objects. This is because Python has no safe way of knowing which order to destroy these objects in.

Starting from Python 3.4, cycles with `__del__` objects [can now be garbage collected](https://www.python.org/dev/peps/pep-0442/). The order in which they are call is undefined though. 


Let's look at a few examples of Context Managers in the real world. Experienced Python developers would have seen many of these examples, but maybe some of them are new.


## Making sure an open stream is closed at the end
The `open()` function is is the canonical example of context managers. It provides an open file handle which will be closed at the end of the block.

~~~python
with open(path, mode) as f:
    f.read()
~~~

`StringIO` objects behave in the same way:

~~~python
with io.StringIO() as b:
     b.write(‘foo’)
~~~

The `closing` context manager calls `close()` on any object if the method exists. 

~~~python
from contextlib import closing
with closing(urllib.urlopen('http://google.com')) as page:
    page.readlines()
~~~

## Making sure an exception is raised in testing

In unittest (2.7+):

~~~python
def test_split(self):
    with self.assertRaises(TypeError):
        'hello world'.split(2)

    with self.assertRaisesRegexp(TypeError, 'expected a string.*object$'):
        'hello world'.split(2)
~~~

E.g. in Pytest

~~~python
with pytest.raises(ValueError):
    int('hello')
~~~

## Setting up mocks before testing

~~~python
with mock.patch('a.b'):
    call()
~~~

`mock.patch` is an example of a [context manager that can also be used as a decorator](https://docs.python.org/3.5/library/contextlib.html#using-a-context-manager-as-a-function-decorator). Python 3 comes with a utility [ContextDecorator](https://docs.python.org/3.5/library/contextlib.html#contextlib.ContextDecorator) that makes it easy to write such context manager-decorators. 

When used as a decorator, `mock.patch` passes the newly created mock object (return value of `__enter__`) into the decorated function. ContextDecorator in Python 3, on the other hand, does not give you access to the return value of `__enter__` method. 


## Synchronising access to shared resources

~~~python
with threading.RLock():
    access_resource()
~~~

The with statement in this case, calls `lock.acquire()` at entry and `lock.release()` on exit. 

Locks are types of [Reentrant context managers](https://docs.python.org/3.5/library/contextlib.html#reentrant-context-managers) which allow you to re-enter the same context manager many times and also reuse it.

In the `threading` module, Lock, RLock, Condition, Semaphore, BoundedSemaphore can be used as context managers.

A similar idea can be used for locking files while accessing them. For e.g. the [pidfile context manager](https://code.activestate.com/recipes/577911/) uses `fcntl.flock()` to obtain a file lock on a daemon pidfile.  

~~~python
import fcntl, csv, sys

class open_locked:
    def __init__(self, *args, **kwargs):
        self.fd = open(*args, **kwargs)

    def __enter__(self):
        fcntl.flock(self.fd, fcntl.LOCK_EX)
        return self.fd.__enter__()

    def __exit__(self, type, value, traceback):
        fcntl.flock(self.fd, fcntl.LOCK_UN)
        return self.fd.__exit__()


with open_locked('data.csv', 'w') as outf:
    writer = csv.writer(outf)
    writer.writerows(someiterable)
~~~

## Setting up the Python execution environment

~~~python
with decimal.localcontext() as ctx:
    # setup high precision operations within this thread
    ctx.prec = 42       
    math_operations()
~~~

## Managing database connections and transactions

~~~python
conn = sqlite3.connect(':memory:')
with conn:
    # start a new db transaction
    conn.execute('create table mytable (id int primary key, name char(50))')
    conn.execute('insert into mytable(id) values (?)', (1, 'avni'))
    # if there was an error, table would not be created in database 
    # as whole transaction is rolled back
    ...
# conn.commit() called.
~~~



## Wrapping remote connections over a protocol:

~~~python
class SomeProtocol:
     def __init__(self, host, port):
          self.host, self.port = host, port
     def __enter__(self):
          self._client = socket()
          self._client.connect((self.host, self.port))
          return self
     def __exit__(self, exception, value, traceback):
          self._client.close()
     def send(self, payload): ...
     def receive(self): ...

with SomeProtocol(host, port) as protocol:
     protocol.send(['get', signal])
     result = protocol.receive()
~~~

## Timing the execution of a code block

~~~python
import time

class Timer:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        self.start = time.time()

    def __exit__(self, *args):
        self.end = time.time()
        self.interval = self.end - self.start
        print("%s took: %0.3f seconds" % (self.name, self.interval))
        return False
~~~

~~~python
>>> with Timer('fetching google homepage'):
...     conn = httplib.HTTPConnection('google.com')
...     conn.request('GET', '/')
... 
fetching google homepage took: 0.047 seconds
~~~

## Automating remote server tasks using Fabric
Fabric provides many interesting [context managers](http://docs.fabfile.org/en/latest/api/core/context_managers.html) to help automate deployment and other local and remote tasks.

~~~python
# In Python 2.7+ multilple context managers can be combined using commas
# In Python 2.6, `contextlib.nested` can be used for the same purpose

with cd('/path/to/app'), prefix('workon myvenv'):
    run('./manage.py syncdb')
    run('./manage.py loaddata myfixture')
~~~

## Create and use a temporary file and delete at the end

~~~python
import tempfile

with tempfile.NamedTemporaryFile() as tf:
    print('Writing to tempfile:', tf.name)
    tf.write('Some data')
    tf.flush()

# By default the temporary file created is deleted when file is closed
# Pass `delete=False` to NamedTemporaryFile to disable auto-delete.
~~~

## Redirecting input and output streams

In Python 3.4+, the [`redirect_stdout`](https://docs.python.org/3/library/contextlib.html?highlight=contextlib#contextlib.redirect_stdout) and [`redirect_stderr`](https://docs.python.org/3/library/contextlib.html?highlight=contextlib#contextlib.redirect_stderr) context manager can be used to temporarily redirect `stdout` and `stderr` streams.

~~~python
with open('output.txt', 'w') as f:
    with redirect_stdout(f):
        print('Hello World!')
~~~

`redirect_stdout` only redirects stdout calls from Python, but not from C library code. See this [blog post by Eli Bendersky](http://eli.thegreenplace.net/2015/redirecting-all-kinds-of-stdout-in-python/) on how to redirect all streams.

## Reading and writing to a file in-place
See [this example by Martijn Pieters](http://www.zopatista.com/python/2013/11/26/inplace-file-rewriting/) of a context manager that allow you to update the same file while reading it line-by-line.
~~~python
import csv

with inplace(csvfilename, 'rb') as (infh, outfh):
    reader = csv.reader(infh)
    writer = csv.writer(outfh)

    for row in reader:
        row += ['new', 'columns']
        writer.writerow(row)
~~~

## Managing a pool of processes
Python's `multiprocessing` library provides a bunch of context managers to manage connections, [pools](https://docs.python.org/3/library/multiprocessing.html?highlight=multiprocessing#module-multiprocessing.pool) and OS-level resource locks.

~~~python
from multiprocessing import Pool

def f(x):
    return x*x

with Pool(processes=4) as pool:  # start 4 worker processes

    # evaluate "f(10)" asynchronously in a single process
    result = pool.apply_async(f, (10,)) 
    print(result.get(timeout=1)) 
    # prints "100"

    print(pool.map(f, range(10)))       
    # prints "[0, 1, 4,..., 81]"

~~~

## Summary
So in short, Context Managers can be used in a great variety of cases. Start using them now whenever you see the "setup-teardown" pattern, to write your Python code in a more Pythonic way.

In future posts, we will explore Decorators and Metaclasses through examples of use as well.
