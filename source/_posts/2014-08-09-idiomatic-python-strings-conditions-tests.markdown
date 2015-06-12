---
layout: post
title: "Idiomatic Python: Strings, Conditions and Tests"
date: 2014-08-09 21:16:48 +0800
comments: true
categories: Python 
tags: [Python]
---
##Concatenating strings
{% codeblock lang:python %}
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']

#NOT SO GOOD
s = names[0]
for name in names[1:]:
    s += ', ' + name
print s

#GOOD
print ', '.join(names)
{% endcodeblock %}
<!-- more -->
##EAFP is preferable to LBYL
"It's **E**asier to **A**sk for **F**orgiveness than **P**ermission."

"**L**ook **B**efore **Y**ou **L**eap."

{% codeblock lang:python %}
d = {'x': '5'}

#NOT SO GOOD
if 'x' in d and isinstance(d['x'], str) and d['x'].isdigit():
    value = int(d['x'])
else:
    value = None

# GOOD
try:
    value = int(d['x'])
except (KeyError, TypeError, ValueError):
    value = None
{% endcodeblock %}
Throwing exceptions is **not** "expensive" in Python unlike e.g. Java.
Rely on duck typing rather than checking for a specific type.

##Use truthy and falsy values

{% codeblock lang:python%}
name = 'Safe'
pets = ['Dog', 'Cat', 'Hamster']
owners = {'Safe': 'Cat', 'George': 'Dog'}

# NOT SO GOOD
if name != '' and len(pets) > 0 and owners != {}:
    print('We have pets!')

# GOOD
if name and pets and owners:
    print('We have pets!')
{% endcodeblock %}

##Use pdb

{% codeblock lang:python%}
import pdb
pdb.set_trace()
{% endcodeblock %}
