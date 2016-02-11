---
lastmod:     2016-02-10
date:        "2016-02-10"
title:       "How it's used: Python Context Managers"
description: "Using examples to explore how Python Context Managers are used in the real world."
tags:        [ "Python", "Programming", "examples" ]
categories:  [ "Python" ]
series:      [ "How it's used" ]
slug:        "python-context-managers"
project_url: ""
draft:       true
---


## Context Managers in summary

Context Managers are a language feature unique to Python. They are an essential tool in any Python developer's toolbox.

Context Managers are mainly used for allocation and release of resources, but that's not their only use case. They can be used for factoring out common setup and teardown code, or any other pair of operations that need to be performed before or after a procedure.

~~~python
setup() 
try:
    do_work()
finally:
    teardown()
~~~

Let's look at a few examples:

## Examples

### Making sure my file handle is closed at the end
This is a canonical example. 

~~~python
with open(path, mode) as f:
    f.read()
~~~

### Close after use 
Calls `close()` on a class if the method exists.

~~~python
from contextlib import closing
with closing(urllib.urlopen('http://google.com')) as page:
    page.readlines()
~~~

### Make writing tests easier
E.g. in Pytest

~~~python
with pytest.raises(ValueError):
    int('hello')
~~~

In unittest (2.7+):

~~~python
def test_split(self):
    with self.assertRaises(TypeError):
        'hello world'.split(2)

    with self.assertRaisesRegexp(TypeError, 'expected a string.*object$'):
        'hello world'.split(2)
~~~

### Setup mocks before testing

~~~python
with mock.patch('a.b'):
    call()
~~~

`mock.patch` is an example of [context managers that can be used as decorators](https://docs.python.org/3.5/library/contextlib.html#using-a-context-manager-as-a-function-decorator). Python 3 comes with a utility [ContextDecorator](https://docs.python.org/3.5/library/contextlib.html#contextlib.ContextDecorator) that makes it easy to write such context manager-decorators. 

When used as a decorator, `mock.patch` passes the newly created mock object (return value of `__enter__`) into the decorated function. ContextDecorator in Python 3, on the other hand, does not give you access to the return value of `__enter__` method. 


### Synchronising access to shared resources

~~~python
with threading.RLock():
    access_resource()
~~~

The with statement in this case, calls `lock.acquire()` at entry and `lock.release()` on exit. 

Locks are types of [Reentrant context managers](https://docs.python.org/3.5/library/contextlib.html#reentrant-context-managers) which allow you to re-enter the same context manager many times and also reuse it.

In the `threading` module, Lock, RLock, Condition, Semaphore, BoundedSemaphore can be used as context managers.


### Setting up the environment

~~~python
with decimal.localcontext() as ctx:
    ctx.prec = 42       # setup high precision operations within this thread
    math_operations()   # ...
~~~

### Managing database connections and transactions

~~~python
conn = sqlite3.connect(':memory:')
with conn:
    # start a new db transaction
    conn.execute('create table mytable (id int primary key, name char(50))')
    conn.execute('insert into mytable(id) values (?)', (1, 'avni'))
    # if there was an error, table would not created in database as whole transaction is rolled back
    ...
# conn.commit() called.
~~~

### Wrapping remote connections over a protocol:

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
