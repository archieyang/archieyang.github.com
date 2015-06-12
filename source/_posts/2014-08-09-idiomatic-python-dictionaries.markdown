---
layout: post
title: "Idiomatic Python: Dictionaries"
date: 2014-08-09 11:19:12 +0800
comments: true
categories: Python
tags: [Python]
---
##Dict comprehension
Dict comprehensions can be used to create dictionaries from arbitrary key and value expressions:
{% codeblock lang:python %}
>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
{% endcodeblock %}
<!-- more -->
##Loop over dictionary keys
{% codeblock lang:python %}
d = {'matthew': 'blue', 'rachel': 'green', 'raymond':'red'}

for k in d:
    print k

for k in d.keys():
    if k.startswith('r'):
        del d[k]
        
d = {k: d[k] for k in d if not k.startswith('r')}
{% endcodeblock %}
##Loop over a dictionary keys and values
{% codeblock lang:python%}
#NOT SO GOOD
for k in d:
    print k, ' -> ', d[k]

#BETTER
for k, v in d.items():
    print k, ' -> ', v

#BEST
for k, v in d.iteritems():
    print k, ' -> ', v
{% endcodeblock%}
##Construct a dictionary from pairs
{% codeblock lang:python %}
from itertools import izip

names = ['raymond', 'rachel', 'matthew']
colors = ['red', 'green', 'blue']

d = dict(izip(names, colors))
{'matthew': 'blue', 'rachel': 'green', 'raymond': 'red'}

d = dict(enumerate(names))
{0: 'raymond', 1: 'rachel', 2: 'matthew'}
{% endcodeblock %}
##Counting with dict
{% codeblock lang:python %}
colors = ['red', 'green', 'red', 'blue', 'green', 'red']

#NOT SO GOOD
d = {}
for color in colors:
    if color not in d:
        d[color] = 0
    d[color] += 1
{'blue': 1, 'green': 2, 'red': 3}

#BETTER
d = {}
for color in colors:
    d[color] = d.get(color, 0) + 1

#BEST
from collections import defaultdict
d = defaultdict(int)
for color in colors:
    d[color] += 1
{% endcodeblock %}
##Grouping with dict
{% codeblock lang:python %}
#NOT SO GOOD
d = {}
for name in names:
    key = len(name)
    if key not in d:
        d[key] = []
    d[key].append(name)
{5: ['roger', 'betty'], 6: ['rachel', 'judith'],
 7: ['raymond', 'matthew', 'melissa', 'charlie']}

#BETTER
d = {}
for name in names:
    key = len(name)
    d.setdefault(key, []).append(name)

#BEST
from collections import defaultdict
d = defaultdict(list)
for name in names:
    key = len(name)
    d[key].append(name)
{% endcodeblock %}
##Use popitem
{% codeblock lang:python %}
d = {'matthew': 'blue', 'rachel': 'green', 'raymond':'red'}

while d:
    key, value = d.popitem()
    print key, '-->', value
{% endcodeblock %}
##Linking dictionaries
{% codeblock lang:python %}
defaults = {'color': 'red', 'user': 'guest'}

parser = argparse.ArgumentParser()
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args([])
command_line_args = {k: v for k, v in vars(namespace).items() if v}

#NOT SO GOOD
d = defaults.copy()
d.update(os.environ)
d.update(command_line_args)

#GOOD(ONLY FOR PYTHON 3)
d = ChainMap(command_line_args, os.environ, defaults)
{% endcodeblock %}
