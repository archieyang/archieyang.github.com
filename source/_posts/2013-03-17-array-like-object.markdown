---
layout: post
title: "实现类似于序列和字典行为的类"
date: 2013-03-17 16:05:39 +0800
comments: true
categories: Python
tags: [Python]
---
##基本概念##

使用一些有用的魔法方法，可以创建具有类似于序列和字典行为的对象。这里涉及的方法有\_\_len\_\_(self), \_\_getitem\_\_(self, key)\_\_, \_\_setitem\_\_(self, key, value)以及\_\_delitem\_\_(self, key)。

\_\_len\_\_(self)
返回集合中所含item的数量。

\_\_getitem\_\_(self, key)
返回指定key对应的item的value。

\_\_setitem\_\_(self, key, value)
按一定的方式存储key的对应的value。

\_\_delitem\_\_(self, key)
删除key对应的item（包含key和value）。
<!-- more -->
##实例##
下面创建一个无穷序列类型ArithmeticSequence作为一个具有类似于序列行为的实例。
{% codeblock lang:python %}
def checkIndex(key)
	if not is instance(key, (int, long)): raise TypeError
	if  key < 0: raise IndexError
class ArithmeticSequence：
	def __init__(self , start=0, step=1 ):
		self.start = start
		self.step = step
		self.changed = {}
	def __getitem__(self, key):
		checkIndex(key）
		try: 
			return self.changed[key]
		except:
			return self.start + key * self.step
	def __setitem__(self, key, value):
		checkIndex(key)
		self.changed[key] = value
{% endcodeblock %}
我们希望这个序列是不可被删除的，所以这里没有实现\_\_delitem\_\_(self, key)函数。由于是无穷序列，这里也没有实现\_\_len\_\_(self)函数。

现在，在初始化这个类之后，可以像一个普通的序列一样操作它：
    
    >>> s = ArithmeticSequence(1, 2)
    >>> s[4]
    9
    >>> s[4] = 2
    >>> s[4]
    2
    >>> s[5]
    11
从上面可以看出，类型ArithmeticSequence的实例具有和序列类似的操作。
##通过继承简化类型的定义##
本文之前描述了为了使得一个类具有类似序列和字典行为时，需要实现的4种基本参数。官方的语言规范也列出了推荐实现的一些其他的函数，包括之前提到的__iter__。但是实现所有这些方法的工作是很繁重的，这里可以使用继承来简化操作。
例如，可以通过子类化list(内建列表)来实现一个和list行为相似的序列。
{% codeblock lang:python %}
class CounterList(list):
	def __init__(self, *args):
		super(CounterList, self).__init__(*args)
		self.counter = 0
	def __getitem__(self, key):
		self.counter = 1
		return super(CounterList, self).__getitem__(index)
{% endcodeblock %}
这里只重写了list的两个方法：\_\_init\_\_和\_\_getitem\_\_，实现了对访问次数的统计。
注意，这里只是为示例使用，实际上这个方法统计访问列表次数并不严谨，因为用户还能通过pop等方法反问列表元素。
实际使用CounterList的例子:

    >>> cl = CounterList(range(10))
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >>> cl.reverse()
    >>> cl
    [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
    >>> cl.del[3:6]
    [9, 8, 7, 3, 2, 1, 0]
    >>> cl.counter
    0
    >>> cl[2] + cl[4]
    12
    >>> cl.counter
    2
##结语##
本文实现了一种具有类似列表和词典行为的类型。实际上Python中还有很多伪装的方法，来实现特定的操作。比如通过拦截对类型属性的访问来改变在访问其属性时类型内部的行为。下一篇将详细介绍用于拦截对类属性的访问的方法Property
