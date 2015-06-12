---
layout: post
title: "Descriptor Howto Guide 的翻译"
date: 2013-03-18 16:05:39 +0800
comments: true
categories: Python
tags: [Python]
---
##摘要##
本文定义了描述符，阐述了它的规则并解释了描述符是如何被调用的。除此之外，本文还详细介绍了一般的描述符和一些Python内置的描述符（包括函数，属性property，静态方法和类方法）的实现。针对这些概念，作者给出了等价的纯Python实现和对应的示例代码。

了解描述符不仅会提供一个更大的工具集合，更会增加对Python工作机制及其优雅设计的理解。
##定义及简介##
通常，descriptor是指具有“绑定行为”的对象特性，这些特性的访问方法被descriptor规则中的方法重写了。这些方法包括\_\_get\_\_(), \_\_set\_\_()和\_\_del\_\_()。实现了其中任何一个方法的对象就被称为描述符（descriptor）。
<!-- more -->

特性（attribute）访问的基本行为是从一个对象的字典中获取（get），设置（set）或者删除特性。例如，a.x有这样一条查找链，它首先查找a.\_\_dict\_\_['x']，然后查找type(a).\_\_dict\_\_['x']，然后继续type(a)不包含metaclass的基类。如果查找到的值是一个定义了某个描述符方法的对象，那么Python就会重写默认的行为，转而调用描述的方法。这种替换发生的优先级取决于定义了那些描述符方法（data descriptor 还是non-data descriptor，后面有详细介绍）。注意，描述符只适用于新式的类和对象。

描述符是一种十分强大且用途广泛的规则（protocol）。它是实现属性(properties)，方法，静态方法，类方法和super()函数背后的机制。它被Python用于实现新式类型（在2.2版本引入）。描述符简化了底层实现的C代码，并且为Python编程提供了一套更加灵活的工具。
##描述符规则##
    descr.__get__(self, obj, type=None) --> value
    descr.__set__(self, obj, value) --> None
    descr.__delete__(self, obj) --> None
这就是关于描述符的全部方法。定义了其中任意一个方法的对象就被称为描述符，它会在查找特性时，重写默认的行为。

如果一个对象定义了\_\_get\_\_()和\_\_set\_\_()两个函数，它就被称为一个数据描述符。而只定义了\_\_get\_\_()函数的描述符被称为非数据描述符（他们通常被用于方法，当然也可以有其他的用途）。

数据描述符和非数据描述符的不同在于他们重写的行为和实例字典中条目的关系。如果一个实例字典中的条目与一个数据描述符重名，那么优先返回的是数据描述符。如果一个实例字典中的条目与一个非数据描述符重名，那么优先返回实例字典中的条目。

如果要创建一个只读的数据描述符，需要定义\_\_get\_\_()和\_\_set()\_\_两个函数，\_\_set()\_\_函数需要在被调用时抛出一个AttributeError异常。定义一个抛出异常的占位符的\_\_set\_\_()函数就可以使得描述符变为数据描述符。
##描述符的调用##
一个描述符可以直接通过它的函数名被调用。例如，d.\_\_get\_\_(obj)。

但是，通过属性访问来自动调用描述符的方法更为常见。例如，obj.d首先在obj的字典中查询d。如果d定义了\_\_get\_\_()方法，那么就会根据下面列出的优先级原则调用d.\_\_get\_\_(obj)。

调用的细节取决于obj是一个对象还是一个类。不管哪种情况，描述符都只作用于新式的对象和类。一个类如果是object的子类，它就是新式的。

对于对象而言，描述符的调用机制存在于object.\_\_getattribute\_\_()，它将b.x转换为type(b).\_\_dict\_\_['x'].\_\_get\_\_(b, type(b))。它的实现包含一个优先级链，在优先级链中，数据描述符具有高于实例变量的优先级，而实例变量具有高于非数据描述符的优先级，而\_\_getattr\_\_()被赋予了最低的优先级。完整的C语言实现可以再Object/object.c中的PyObject_GenericGetAttr()中找到。

对于类型而言，描述符的调用机制存在于type.\_\_getattribute\_\_()中，它将B.x转化为B.\_\_dict\_\_['x'].\_\_get\_\_(None, B)。它的Python语言描述如下：
{% codeblock lang:python %}
def __getattribute__(self, key):
	"Emulate type_getattro() in Objects/typeobject.c"
	v = object.__getattribute__(self, key)
	if hasattr(v, '__get__'):
		return v.__get__(None, self)
	return v
{% endcodeblock %}
这里需要注意的是

描述符通过\_\_getattribute\_\_()方法被调用	重写\_\_getattribute\_\_()方法会阻止描述符的自动调用	\_\_getattribute\_\_()只存在于新式的类和对象中	object.\_\_getattribute\_\_()和type.\_\_getattribute\_\_()对于\_\_get\_\_()的调用不同	数据描述符会重写实例的字典	非数据描述符会被实例的字典重写
由super()函数返回的对象也具有一个普通的\_\_getattribute\_\_()方法来调用操作符。super(B, obj).m()搜索obj.\_\_class\_\_.\_\_mro\_\_来寻找B类型的直接的基类A，并返回A.\_\_dict\_\_['m'].\_\_get\_\_(obj, A). 如果m不是一个描述符，那么它会直接被返回。如果m不在字典中，Python会继续通过object.\_\_getattribute\_\_()查找。

注意，在Python 2.2中，只有在m是一个数据描述符的情况下，super(B, obj).m()才会调用\_\_get\_\_()。在Python 2.3中，非数据描述符也会被调用，除非是在一个旧式的类中。这里的实现细节在Objects/typeobject.c的super_getattro()中，一个等价的纯Python的实现可以在Guido’s Tutorial中找到。

以上的细节表明描述符机制内置于object，type和super()的\_\_getattribute\_\_()方法中。类型从object中继承了这种机制，或则是从某个具有类似功能的meta-class中获得。同样，类型可以通过重写\_\_getattribute\_\_()关闭描述符的调用。
##描述符示例##
下面的代码创建了一个类型，他的对象是能够在get和set操作时，打印信息的数据描述符。重写\_\_getattribute\_\_()方法可以为所有的特性实现这种行为。但是，在只需要选择监视几个特性的情况下，描述符更加有用。
{% codeblock lang:python %}
class RevealAccess(object):
	def __init__(self, initval=None, name='var'):
		self.val = initval
		self.name = name
	def __get__(self,obj, objtype):
		print 'Retrieving', self.name
		return self.val
	def __set__(self, obj, val):
		print 'Updating', self.name
	    self.val = val
{% endcodeblock %}
    >>> class MyClass(object):
    x = RevealClass(10, 'var "x"')
    y = 5
    >>> m = MyClass()
    >>> m.x
    Retrieving var "x"
    10
    >>> m.x = 20
    Updating var "x"
    >>> m.x
    Retrieving var "x"
    20
    >>> m.y
    5
可以看出，描述符的规则很简单却有很广泛的用途。某些应用场景十分常见，以至于他们已经被封装成了函数调用。属性(properties)，绑定和非绑定方法，静态方法和类方法都是基于描述符规则实现的。
##属性Properties##
数据描述符在特性被访问时，自动触发函数调用。使用property()函数是实现数据描述符的一种简洁的方法。它的函数签名（signature）如下：

    property(fget=None, fset=None, fdel=None, doc=None) ->property attribute
文档中给出了一个定义受控属性x的典型例子：

{% codeblock lang:python %}
class C(object):
	def getx(self): return self.__x
	def setx(self, value): self.__x = value
	def delx(self): del self.__x
	x = property(getx, setx, delx, "I'm the 'x' property.")
{% endcodeblock %}
就property()函数关于描述符机制的实现，下面给出了等价的纯Python实现。

{% codeblock lang:python %}
class Property(object):
	"Emulate PyProperty_Type() in Objects/descrobject.c"

	def __init__(self, fget=None, fset=None, fdel=None, doc=None):
		self.fget = fget
		self.fset = fset
		self.fdel = fdel
		if doc is None and fget is not None:
			doc = fget.__doc__
			self.__doc__ = doc

	def __get__(self, obj, objtype=None):
		if obj is None:
			return self
		if self.fget is None:
			raise AttributeError("unreadable attribute")
		return self.fget(obj)

	def __set__(self, obj, value):
		if self.fset is None:
			raise AttributeError("can't set attribute")
		self.fset(obj, value)

	def __delete__(self, obj):
		if self.fdel is None:
			raise AttributeError("can't delete attribute")
		self.fdel(obj)

	def getter(self, fget):
		return type(self)(fget, self.fset, self.fdel, self.__doc__)

	def setter(self, fset):
		return type(self)(self.fget, fset, self.fdel, self.__doc__)

	def deleter(self, fdel):
	    return type(self)(self.fget, self.fset, fdel, self.__doc__)
{% endcodeblock %}
内建函数property()在用户接口提供特性访问权限，并且随后对特性的改变需要有函数介入的情况下，是非常有用的。
例如，一个电子表格的类可能会通过Cell(‘b10′).value提供对一个单元格值的访问。后来对程序的改进可能要求每次访问时，单元格的值需要被重新计算。但是，程序员可能不想影响到现存的直接访问特性的客户代码。解决的方法就是采用一个属性（property）数据描述符将对值特性的访问包装起来。
{% codeblock lang:python %}
class Cell(object):
	. . .
	def getvalue(self, obj):
		"Recalculate cell before returning value"
		self.recalc()
		return obj._value
	value = property(getvalue)
{% endcodeblock %}
##函数和方法##
Python的面向对象特性建立在一个基于函数的环境上。非数据描述符将这两者紧密的联系起来。

类的字典将方法作为函数存储。在一个类的定义中，方法通过关键字def和lambda被定义，这也是建立函数的一般方法。方法和普通函数唯一的区别在于，方法的第一个参数保留给了对象实例。根据Python的习惯，这个实例的引用被称为self，但是它也可以被命名为this或者其他的变量名。

为了支持方法的调用，函数中包含了\_\_get\_\_()方法来在访问特性时绑定方法。这意味所有的函数都是非数据描述符，他们返回绑定还是非绑定函数取决于他们是被对象还是被类型调用。用纯Python表示，它的工作方式如下：
{% codeblock lang:python %}
class Function(object):
	. . .
	def __get__(self, obj, objtype=None):
		"Simulate func_descr_get() in Objects/funcobject.c"
	    return types.MethodType(self, obj, objtype)
{% endcodeblock %}
启动一个解释器来看一下函数描述符在是如何工作的：

    >>> class D(object):
     def f(self, x):
      return x
    >>> d = D()
    >>> D.__dict__['f'] # Stored internally as a function
    
    >>> D.f # Get from a class becomes an unbound method
    
    >>> d.f # Get from an instance becomes a bound method
    >
输出显示了绑定和非绑定的函数属于两种不同的类型。尽管他们可以被作为不同类型实现，Objects/classobject.c中PyMethod\_Type的 C实现采用的是同一个对象的两种不同表示，使用时需要根据im\_self的值是被设置，还是为NULL(C中等价的None)来判断。
同时，调用一个函数对象的效果也由im\_self域决定。如果设置了im\_self(表示绑定)，原始的函数（存储在im\_func中）在被调用时，第一个参数被设置为实例，正如期望那样。如果没有绑定，所有的参数被直接传递给原始函数而不做任何修改。instancemethod\_call()的C实现除了包含一些类型检查，并不是特别复杂。
##静态类和方法##
非数据的描述符提供了一种简单的机制来使得从函数到方法的转化更加多样化。

我们回顾一下，函数因为实现了\_\_get\_\_()方法，所以当他们被作为属性访问时，可以被转化为方法。非数据描述符将obj.f(\*args)调用转化为f(obj, \*args)。klass.f(\*args)变成了f(\*args)。

下面这个表单总结了绑定，和绑定最常见的两种变形。
<table border="1">
<tbody>
<tr>
<th>变形</th>
<th>从对象调用</th>
<th>从类型调用</th>
</tr>
<tr>
<th>函数</th>
<th>f(obj, *args)</th>
<th>f(*args)</th>
</tr>
<tr>
<th>静态方法</th>
<th>f(*args)</th>
<th>f(*args)</th>
</tr>
<tr>
<th>类方法</th>
<th>f(type(obj), *args)</th>
<th>f(klass, *args)</th>
</tr>
</tbody>
</table>
静态方法直接返回底层的实现函数不做任何改变。调用c.f 或者C.f都等价于直接查找object.\_\_getattribute\_\_(c, “f”) 或者object.\_\_getattribute\_\_(C, “f”)。因此，这个函数在对象和类中的访问是等价的。

不需要引用self的方法适合采用静态方法实现。

例如，一个统计的包或许会为实验数据设计一个容器类。这个类为数据提供了常见的计算平均值的方法，平均值，中值，和其他描述这组数据的统计值。但是，或许有些函数在概念上和数据相关，但又不依赖于数据。例如，erf(x)利用统计方法处理变化曲线，但是它并不直接依赖于某个数据集合。它可以被一个对象或者类型调用：s.erf(1.5) --> .9332 或者Sample.erf(1.5) --> .9332。

由于静态方法不加改变的返回底层的函数，下面的实例应该很容易理解：
    
    >>> class E(object):
		     def f(x):
		      print x
		     f = staticmethod(f)
    >>> print E.f(3)
    3
    >>> print E().f(3)
    3
采用非数据描述符规则，一个纯Python版的staticmethond()的实现如下：
{% codeblock lang:python %}
class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"
    
	def __init__(self, f):
	    self.f = f
	    
	def __get__(self, obj, objtype=None):
	    return self.f
{% endcodeblock %}
与静态方法不同，类方法需要在调用函数前将类型的引用添加到参数表里。无论调用者是对象还是类型，这种形式是一致的：

    >>> class E(object):
	     def f(klass, x):
	          return klass.__name__, x
	     f = classmethod(f)
    >>> print E.f(3)
    ('E', 3)
    >>> print E().f(3)
    ('E', 3)
类方法的这种行为在一个函数需要使用类型的引用而不关心底层数据时，是十分有用的。类方法的一个常见用法是创建类的替代构造函数。在Python 2.3中，类函数dict.fromkeys()可以使用一个键的序列创建一个新的字典。纯Python的等价实现如下：
{% codeblock lang:python %}
class Dict(object):
	. . .
	def fromkeys(klass, iterable, value=None):
	    "Emulate dict_fromkeys() in Objects/dictobject.c"
		d = klass()
	    for key in iterable:
	        d[key] = value
	    return d
	fromkeys = classmethod(fromkeys)
{% endcodeblock %}
现在一个具有唯一键的新的字典可以按照如下方法构建：

    >>> Dict.fromkeys('abracadabra')
    {'a': None, 'r': None, 'b': None, 'c': None, 'd': None}
运用非数据描述符规则，一个纯Python版本的classmethod()实现如下：
{% codeblock lang:python %}
class ClassMethod(object):
	"Emulate PyClassMethod_Type() in Objects/funcobject.c"
	
	def __init__(self, f):
	    self.f = f
	
	def __get__(self, obj, klass=None):
	    if klass is None:
	        klass = type(obj)
	    def newfunc(*args):
	        return self.f(klass, *args)
	    return newfunc
{% endcodeblock %}
###参考资料
[Descriptor HowTo Guide](http://docs.python.org/2/howto/descriptor.html)
