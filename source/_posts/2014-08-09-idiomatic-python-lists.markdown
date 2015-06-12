---
layout: post
title: "Idiomatic Python: Lists"
date: 2014-08-09 11:07:31 +0800
comments: true
categories: Python
tags: [Python]
---
##List Comprehension
{% codeblock lang:python %}
>>> print [i for i in range(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

>>> print [i for i in range(20) if i%2 == 0]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

>>> nums = [1,2,3,4]
>>> fruit = ["Apples", "Peaches", "Pears", "Bananas"]
>>> print [(i,f) for i in nums for f in fruit]
[(1, 'Apples'), (1, 'Peaches'), (1, 'Pears'), (1, 'Bananas'),
 (2, 'Apples'), (2, 'Peaches'), (2, 'Pears'), (2, 'Bananas'),
 (3, 'Apples'), (3, 'Peaches'), (3, 'Pears'), (3, 'Bananas'),
 (4, 'Apples'), (4, 'Peaches'), (4, 'Pears'), (4, 'Bananas')]
 
>>> print [(i,f) for i in nums for f in fruit if f[0] == "P"]
[(1, 'Peaches'), (1, 'Pears'),
 (2, 'Peaches'), (2, 'Pears'),
 (3, 'Peaches'), (3, 'Pears'),
 (4, 'Peaches'), (4, 'Pears')]
 
>>> print [(i,f) for i in nums for f in fruit if f[0] == "P" if i%2 == 1]
[(1, 'Peaches'), (1, 'Pears'), (3, 'Peaches'), (3, 'Pears')]

>>> print [i for i in zip(nums,fruit) if i[0]%2==0]
[(2, 'Peaches'), (4, 'Bananas')]
{% endcodeblock %}
