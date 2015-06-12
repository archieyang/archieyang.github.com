---
layout: post
title: "Idiomatic Python: Improve Clarity"
date: 2014-08-09 20:36:24 +0800
comments: true
categories: Python 
tags: [Python]
---

##Clarify function calls with keyword arguments
{% codeblock lang:python %}
#NOT SO GOOD
twitter_search('@obama', False, 20, True)

#GOOD
twitter_search('@obama', retweets=False, numtweets=20,popular=True)
{% endcodeblock %}
<!-- more -->

##Clarify multiple return values with named tuples
<code>namedtuple</code> is a factory function for creating tuple subclasses with named fields.

{% codeblock lang:python %}
#NOT SO GOOD
def test():
    #some test here
    attempted = 4
    failed = 0
    return failed, attempted# -> (0, 4)

#GOOD
from collections import namedtuple

TestResult = namedtuple('TestResult', ['failed', 'attempt'])


def test():
    #some test here
    attempted = 4
    failed = 0
    return TestResult(failed, attempted)# -> TestResults(failed=0, attempted=4)
{% endcodeblock %}

##Unpacking sequences
{% codeblock lang:python %}
p = 'Raymond', 'Hettinger', 0x30, 'python@example.com'

# NOT SO GOOD
fname = p[0]
lname = p[1]
age = p[2]
email = p[3]

#GOOD
fname, lname, age, email = p
{% endcodeblock %}

