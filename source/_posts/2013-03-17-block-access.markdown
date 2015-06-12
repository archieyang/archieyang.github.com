---
layout: post
title: "拦截对象的属性访问"
date: 2013-03-17 16:05:39 +0800
comments: true
categories: Python
tags: [Python]
---
拦截对象所有的属性访问是可能的，这样可以在不使用property方法的情况下实现属性（property）。下面的4种方法提供了需要的功能。

\_\_getattribute\_\_(self, name)
当name被访问时自动被调用。

\_\_getattr\_\_(self, name)
当name被访问，且对象没有响应特性时自动被调用。

\_\_setattr\_\_(self, name, value)
当试图给特性name赋值时会被自动调用。

\_\_delattr\_\_(self, name)
当试图删除特性name时会被自动调用。


尽管和Property函数相比有些复杂，并且在某些情况下效率更低，但是这些方法是很强大的。
<!-- more -->
下面还是实现Rectangle的例子
{% codeblock lang:python %}
class Rectangle:
	def __init__(self):
		self.width = 0
		self.width = 0
	def __setattr__(self, name, value):
		if name == 'size':
			self.width, self.height = value
		else:
			self.__dict__[name] = value
	def __getattr__(self, name):
		if name == 'size':
			return self.width, self.height
		else:
			raise AttributeError
{% endcodeblock %}
这里有两个需要注意的细节：

\_\_setattr\_\_方法在试图给所有name赋值时都会被调用，也就是说，在调用的特性名不为”size”时也会被调用。因此，这个方法需要考虑两种情况：在被访问特性为size时，就执行对self.width和self.height的赋值；在被访问特性不是size时，使用\_\_dict\_\_字典，\_\_dict\_\_字典里包含所有的属性，通过\_\_dict\_\_[name]设置对应的值。注意，这里不能直接写成self.name = value。这是因为self.name = value会再次调用\_\_setattr\_\_，导致陷入死循环。同样需要避免陷入死循环的还有\_\_getattribute\_\_：因为\_\_getattribute\_\_会拦截对所有特性的访问，这里也包括对\_\_dict\_\_的访问！因此，在\_\_getattribute\_\_中访问self的相关属性时，使用超类的\_\_getattribute\_\_方法（使用super函数）是唯一安全的方法。

\_\_getattr\_\_方法只在被访问对象没有name属性时才被访问，因此如果name不是size，就说明这个特性不存在，这时只需要抛出一个AttributeError异常。
