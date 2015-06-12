---
layout: post
title: "使用wraps装饰器工厂"
date: 2014-07-13 12:13:39 +0800
comments: true
categories: Python
tags: [Python]
---
wraps位于functools模块，通过调用partial(update_wrapper, wrapped=wrapped,assigned=assigned, updated=updated)来定义一个wrapper函数。

Python中decorator的定义并不需要使用wraps装饰器工厂，一个典型的Python decorator如下：
{% codeblock lang:python %}
from functools import wraps

def my_decorator(f):
    def wrapper(*args, **kwds):
        print 'Calling decorated function'
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """Docstring"""
    print 'Called example function'

example()
print example.__name__
print example.__doc__


Results:

Calling decorated function
Called example function
wrapper
wrapper doc

{% endcodeblock %}
<!-- more -->

可以看到函example在别装饰后，丢失了自己原有的名称和文档，被替换为了wrapper的函数名和文档。
wraps的作用就是让被装饰函数仍然保留原来的名称和文档，只需要加上一行代码就可以实现这种效果。修改后的代码如下：
{% codeblock lang:python %}
from functools import wraps

def my_decorator(f):
    @wraps(f)#Add wraps decorator factory
    def wrapper(*args, **kwds):
        """wrapper doc"""
        print 'Calling decorated function'
        return f(*args, **kwds)
    return wrapper

@my_decorator
def example():
    """example doc"""
    print 'Called example function'

example()
print example.__name__
print example.__doc__


Results:

Calling decorated function
Called example function
example
example doc

{% endcodeblock %}

