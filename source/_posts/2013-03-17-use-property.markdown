---
layout: post
title: "使用Property函数创建属性"
date: 2013-03-17 16:05:39 +0800
comments: true
categories: Python
tags: [Python]
---
##访问器##
访问器（accessor methods）一个简单的方法，能够使用个一些函数来获得属性，或者给属性赋值。下面是一个访问器的例子：
{% codeblock lang:python %}    
class Rectangle:
	def __init__(self):
		self.width = 0
		self.height = 0
	def setSize(self, size):
		self.width, self.height = size
	def getSize(self):
		return self.width, self.height
{% endcodeblock %}
这里的getSize和setSize方法是一个名为size的假想特性(attributes)的访问器方法。这里的问题是，如果某天真的为这个类设计了一个size属性，那之前的代码都要修改。（比如获取size从rectangle.getSize()改为rectangle.size，修改size从rectangle.setSize(value)改为rectangle.size=value）。实际上Python提供了隐藏访问器方法，使用这些方法可以让所有特性（attribute）看起来一样，这些通过访问器定义的特性（attributes）被称为属性(properties)。Python中有两种创建属性(properties)的机制，本文主要讨论新式类中提供的Property函数。
<!-- more -->
##使用property函数##
如果已经编写了上面的Rectangle类，只要稍作就该就可以将size变为类的属性。
{% codeblock lang:python %}
__metaclass__ = type
class Rectangle:
	def __init__(self):
		self.width = 0
		self.height = 0
	def setSize(self, size):
		self.width, self.height = size
		def getSize(self):
		return self.width, self.height
	size = propery(getSize, setSize)
{% endcodeblock %}
property函数有四个参数，分别为fget, fset, fdel和doc。前三个属性分别控制属性的读，写和删除，doc是一个文档字符串。可以通过设置这些参数中的一个或几个（或0个）来实现只读，只写，读写，不可删除等具有不同操作的属性。
下面的例子展示了如何操作这个新的Rectangle类的实例。

    >>> r = Rectangle()
    >>> r.width = 10
    >>> r.height = 5
    >>> r.size
    (10, 5)
    >>> r.size = 150, 100
    >>> r.width
    150
##property函数的实现##
property函数实际上并不是一个真正函数，而是一个具有特殊方法的类。在Python中这种类的对象被称为描述符（descriptor）。想更深入的了解descriptor可以看这篇Descriptor Howto Guide，也可以看我后面翻译的版本。
