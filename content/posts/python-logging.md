---
lastmod:     2016-02-22
date:        "2016-02-22"
title:       "Python Logging made simple"
description: "Using examples to explore how Python Context Managers are used in the real world."
tags:        [ "python", "programming" ]
categories:  [ "Python" ]
series:      []
slug:        "python-logging"
project_url: ""
draft:       true
---

Over the years as a Python developer, I found the logging library a bit opaque. The library has a distinct Java feel to it, because of it's heavy use of OO patterns, camel-case naming convention, and generically named classes like *Handlers* and *Loggers*.

Once you understand how the functionality is broken down into classes, their responsibilities and structure, it begins to make more sense. I hope to make it more accessible in this post.
<!--more-->

## Simple Logging

To simply write some log messages in a Python, it is easy:
~~~python
import logging
logging.debug('within func()')
logging.error('database is unavailable')
~~~

## Class structure

## 