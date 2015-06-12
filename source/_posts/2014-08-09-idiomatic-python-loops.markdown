---
layout: post
title: "Idiomatic Python: Loops"
date: 2014-08-09 09:06:37 +0800
comments: true
categories: Python
tags: [Python]
---

##Use xrange instead of tranditional index manipulation
{% codeblock lang:python %}
# NOT SO GOOD
for i in [0, 1, 2, 3, 4, 5]:
    print i * 2

#BETTER
for i in range(6):
    print i*2

#BEST
for i in xrange(6):
    print i*2
{% endcodeblock %}
<!-- more -->
##Use reverse to loop backwards
{% codeblock lang:python %}
colors = ['red', 'green', 'blue', 'yellow']

#NOT SO GOOD
for i in range(len(colors)-1, -1, -1):
    print colors[i]

#GOOD
for color in reversed(colors):
    print color
{% endcodeblock %}
##Use enumerate to loop over a collection and indices#2. List
{% codeblock lang:python %}
colors = ['red', 'green', 'blue', 'yellow']

#NOT SO GOOD
for i in range(len(colors)):
    print i, " -> ", colors[i]
    
#GOOD
for i, color in enumerate(colors):
    print i, " -> ", colors[i]
{% endcodeblock %}
##Use izip to loop over two collections
{% codeblock lang:python %}
colors = ['red', 'green', 'blue', 'yellow']
names = ['Archie', 'Ben', 'Chris', 'Dave']

#NOT SO GOOD
n = min(len(colors), len(names))
for i in range(n):
    print colors[i], ' -> ', names[i]

#BETTER
for name, color in zip(names, colors):
    print color, ' -> ', name

#BEST
from itertools import izip
for name, color in izip(names, colors):
    print color, ' -> ', name
{% endcodeblock %}
##Use sorted to loop in sorted order
{% codeblock lang:python %}
colors = ['red', 'green', 'blue', 'yellow']

#NORMAL
for color in sorted(colors):
    print color

#REVERSE
for color in sorted(colors, reversed=True):
    print color


#CUSTOM COMPARATOR
def compare_length(c1, c2):
    if len(c1) < len(c2):
        return -1
    if len(c1) > len(c2):
        return 1
    return 0

print sorted(colors, cmp=compare_length())
print sorted(colors, key=len)
{% endcodeblock %}
##Use for…else… to distinguish multiple exit points in loops
The "else" here is better explained by "no break".
{% codeblock lang:python%}
#NOT SO GOOD
def find(seq, target):
    found = False
    for i, value in enumerate(seq):
        if value == target:
            found = True
            break
    if not found:
        return -1
    return i


#GOOD
def find(seq, target):
    for i, value in enumerate(seq):
        if value == target:
            break
    else:
        return -1
    return i
{% endcodeblock %}
