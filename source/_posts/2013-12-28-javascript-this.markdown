---
layout: post
title: "Smashing JavaScript: this值详解"
date: 2013-12-28 16:23:56 +0800
comments: true
categories: JavaScript
tags: [JavaScript]
---
内容翻译自[ECMA-262-3 in detail. Chapter3. This](http://dmitrysoshnikov.com/ecmascript/chapter-3-this/)

##定义
this是执行上下文的一个属性，在一个执行上下文中，它包含变量对象、作用域链以及this值三个属性。this值和上下文的可执行代码类型密切相关。它在进入上下文时被确定，并且在这段上下文中的代码执行时被确定。

##全局代码中的this值
在全局代码中，this值永远指向全局对象本身
{% codeblock lang:javascript %}
// explicit property definition of
// the global object
this.a = 10; // global.a = 10
alert(a); // 10
 
// implicit definition via assigning
// to unqualified identifier
b = 20;
alert(this.b); // 20
 
// also implicit via variable declaration
// because variable object of the global context
// is the global object itself
var c = 30;
alert(this.c); // 30
{% endcodeblock %}
<!-- more -->
##函数代码中的this值

在函数代码中，this最主要的特性在于他没有被静态的绑定到函数上。如前文所述，this值在进入上下文时被确定，对于一个函数而言，这个值在每次调用的时候可能完全不同。

但是，在运行时this值是不可变的，即不能为this赋一个新值，因为它并不是一个变量。

有几个因素会影响到运行时的this值：

首先，在一个普通的函数调用中，this值由激活上下文的调用者提供，比如调用该函数的父级上下文。而this的值由调用表达式的形式决定。

记住这点非常重要，this值只受调用表达式的形式，也就是调用函数的方法影响。
往下看，我们会发现，即使一个普通的全局函数也可能会通过不同形式的调用表达式被激活，并使它获得一个不同的this值：
{% codeblock lang:javascript %}
function foo() {
  alert(this);
}
 
foo(); // global
 
alert(foo === foo.prototype.constructor); // true
 
// but with another form of the call expression
// of the same function, this value is different
 
foo.prototype.constructor(); // foo.prototype
{% endcodeblock %}

同样，我们可以像函数一样调用对象的方法，同时使this值并不指向该对象：
{% codeblock lang:javascript %}
var foo = {
  bar: function () {
    alert(this);
    alert(this === foo);
  }
};
 
foo.bar(); // foo, true
 
var exampleFunc = foo.bar;
 
alert(exampleFunc === foo.bar); // true
 
// again with another form of the call expression
// of the same function, we have different this value
 
exampleFunc(); // global, false
{% endcodeblock %}

调用表达式究竟是怎么影响this值的？为了完全理解如何确定this的值，我们有必要学习下内部类型Reference的实现细节。

###Reference

下面采用伪代码来表示Reference类型的值的细节，Reference被表示成一个对象，该对象具有两个属性：一个base属性和一个propertyName属性：

{% codeblock lang:javascript %}
var valueOfReferenceType = {
  base: <base object>,
  propertyName: <property name>
};
{% endcodeblock %}

我们只有在两种情况下会对Reference类型求值：
1. 当处理标识符时
2. 当处理属性访问时

标识符在标识符解析过程中被处理，我们需要了解的是，在这个过程中返回的一定是一个Reference类型。

标识符包括变量名，函数名，函数参数名和全局对象的未受限属性。

考虑下面这个例子中的标识符：
{% codeblock lang:javascript %}
var foo = 10;
function bar() {}
{% endcodeblock %}
在操作中，对应的Reference类型入如下：
{% codeblock lang:javascript %}
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
var barReference = {
  base: global,
  propertyName: 'bar'
};
{% endcodeblock %}

为了从Reference类型中获得真正的对象值，需要使用一个类似于下面伪代码表示的GetValue函数：
{% codeblock lang:javascript %}
function GetValue(value) {
 
  if (Type(value) != Reference) {
    return value;
  }
 
  var base = GetBase(value);
 
  if (base === null) {
    throw new ReferenceError;
  }
 
  return base.[[Get]](GetPropertyName(value));
 
}
{% endcodeblock %}

这里内部函数[[Get]]返回对象属性的真正的值，包括对原型链中继承的属性的分析。

属性访问也是一样，有两种类型的属性访问：点表示和括号表示：
{% codeblock lang:javascript %}
foo.bar();
foo['bar']();
{% endcodeblock %}

在计算中间值的过程中，我们仍然使用Reference类型：

{% codeblock lang:javascript %}
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
 
GetValue(fooBarReference); // function object "bar"
{% endcodeblock %}

下面将列出的是本文的重中之重，确定一个函数上下文中this值的规则是：
>函数上下文中的this值由调用提供，并且有当前的调用表达式（语法上函数调用的写法）确定。

>如果在调用括号()的左边是一个Reference类型的话，this的值被设计值这个Reference类型的base对象。

>在所有其他的情况下（比如其他任何不是Reference类型的值类型），this值总是被设置为null。但是由于对于this而言，null值是没有意义的，它被隐式的转化为了全局对象。

让我们来看一个例子：

{% codeblock lang:javascript %}
function foo() {
  return this;
}
 
foo(); // global
{% endcodeblock %}

在调用括号的左侧是一个Reference类型的值（因为foo是一个标识符）：

{% codeblock lang:javascript %}
var fooReference = {
  base: global,
  propertyName: 'foo'
};
{% endcodeblock %}

根据前面的规则，this值被设置为了Reference类型的base对象，也就是全局对象。

同样的，对于一个属性访问而言：

{% codeblock lang:javascript %}
var foo = {
  bar: function () {
    return this;
  }
};
 
foo.bar(); // foo
{% endcodeblock %}

这里我们得到了一个Reference类型，它的base对象，也就是foo对象在激活bar函数时被用作this的值。

{% codeblock lang:javascript %}
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
{% endcodeblock %}

但是，通过另一个调用表达式激活这个函数，我们会得到另一个this值：

{% codeblock lang:javascript %}
var test = foo.bar;
test(); // global
{% endcodeblock %}

因为这里的标识符test，产生了另一个Reference的值，而它的base对象-全局对象，被用作了this的值。
{% codeblock lang:javascript %}
var testReference = {
  base: global,
  propertyName: 'test'
};
{% endcodeblock %}

注意，在ES5中的strict mode中，this值不再指向全局变量，而是被设置为undefined。

现在我们可以准确的分析处，为什么通过一个函数在被不同形式的调用表达式激活时，会具有不同的this值-因为它们有不同的Reference中间值：

{% codeblock lang:javascript %}
function foo() {
  alert(this);
}
 
foo(); // global, because
 
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
alert(foo === foo.prototype.constructor); // true
 
// another form of the call expression
 
foo.prototype.constructor(); // foo.prototype, because
 
var fooPrototypeConstructorReference = {
  base: foo.prototype,
  propertyName: 'constructor'
};
{% endcodeblock %}

另一个由于调用表达式导致的动态this值的典型例子如下：

{% codeblock lang:javascript %}
function foo() {
  alert(this.bar);
}
 
var x = {bar: 10};
var y = {bar: 20};
 
x.test = foo;
y.test = foo;
 
x.test(); // 10
y.test(); // 20
{% endcodeblock %}
###函数调用和非Reference类型
所以，像我们注意到的，如果在调用括号的左侧的值算是其他类型，而不是Reference时，this的值就被甚至为了null，进而指向全局变量。

让我们考虑下面的表达式：
{% codeblock lang:javascript %}
(function () {
  alert(this); // null => global
})();
{% endcodeblock %}

在这种情况下，我们有一个函数对象，而不是一个Reference类型的对象，因此this的值指向了全局对象。

更复杂一些的例子如下：
{% codeblock lang:javascript %}
var foo = {
  bar: function () {
    alert(this);
  }
};
 
foo.bar(); // Reference, OK => foo
(foo.bar)(); // Reference, OK => foo
 
(foo.bar = foo.bar)(); // global?
(false || foo.bar)(); // global?
(foo.bar, foo.bar)(); // global?
{% endcodeblock %}

既然都是中间值为Reference类型的属性访问，为什么我们在有些情况中得到了base对象，而在有些情况中得到了全局对象？

这是因为后三种调用，在使用了一些操作符后，在调用括号的左侧已经不是Reference类型了。
第一个例子很清楚，对于一个Reference类型，this被设为了它的base对象， foo。

第二个例子中，grouping操作符没有出发获取实际值的GetValue方法，因此grouping操作返回的仍然hi一个引用类型的值，因此this被设为了base对象。

滴撒个例子中，赋值操作符，和grouping操作符不同，调用了GetValue方法求值，因此返回的是一个函数对象，而不是Reference类型的对象，因此this值被设为null，并最终指向全局对象。

类似的还有第四个和第五个例子-逗号运算符和逻辑或表达式调用了GetValue方法，因此我们失去了Reference类型的对象，得到了函数类型的对象，因此this被设置为了全局对象。

###Reference类型和值为null的this

有一种特殊情况在调用括号的左侧是一个Reference类型，但是this的值却被设置为null，进而变为全局变量。这种情况下，Reference值的base对象是一个活动对象。

我们把下面这个内部函数被父函数调用的情况当做例子。我们知道，局部变量，内部函数和形式参数都存储在给定函数的活动对象中：
{% codeblock lang:javascript %}
function foo() {
  function bar() {
    alert(this); // global
  }
  bar(); // the same as AO.bar()
}
{% endcodeblock %}

这个活动对象返回的this值总是为null，进而被设置为全局对象。

这种特殊情况还出现在一个with语句的内部一个调用函数，而这个函数又恰好是with对象包含的函数时。with语句将with对象放到作用域链的最前端，也就是活动对象之前。因此，对于Reference类型的值而言，base对象并不是活动对象，而是一个with语句的对象。在这种情况下，它不仅和内部有关，而且和全局函数有关，因为with对象掩盖了作用域链中更高层级的对象。
{% codeblock lang:javascript %}
var x = 10;
 
with ({
 
  foo: function () {
    alert(this.x);
  },
  x: 20
 
}) {
 
  foo(); // 20
 
}
 
// because
 
var  fooReference = {
  base: __withObject,
  propertyName: 'foo'
};
{% endcodeblock %}

同样的情况还出现在调用一个作为catch从句的实参的函数时：在这种情况下，catch对象也被添加到了作用域链的顶端，也就是活动对象，或者全局对象之前。但是，这种行为被认为是ECMA-262-3的一个bug，并在ECMA-262-5中被修正，即，给定的活动中的this值被设置为全局对象，而不是catch对象：
{% codeblock lang:javascript %}
try {
  throw function () {
    alert(this);
  };
} catch (e) {
  e(); // __catchObject - in ES3, global - fixed in ES5
}
 
// on idea
 
var eReference = {
  base: __catchObject,
  propertyName: 'e'
};
 
// but, as this is a bug
// then this value is forced to global
// null => global
 
var eReference = {
  base: global,
  propertyName: 'e'
};
{% endcodeblock %}

同样的情况还出现在函数的递归调用中。在第一次调用函数时，base对象是父活动对象（或者全局对象），在递归调用中，base对象应该是存储函数表达式名字的对象。但是，在这种情况下，this值始终为全局对象：

{% codeblock lang:javascript %}
(function foo(bar) {
 
  alert(this);
 
  !bar && foo(1); // "should" be special object, but always (correct) global
 
})(); // global
{% endcodeblock %}

###构造函数中的this

还有一种在函数上下文中被调用的this值的情况-调用作为构造函数的函数：
{% codeblock lang:javascript %}
function A() {
  alert(this); // newly created object, below - "a" object
  this.x = 10;
}
 
var a = new A();
alert(a.x); // 10
{% endcodeblock %}

在这种情况下，new操作符调用A函数的内部方法[[Contruct]]，并在创建对象后，调用同一个A函数内部的[[Call]]方法，并将this的值设置为新创建的对象的值。

###指定函数调用this值

在Function.prototype中，有两个方法允许设定函数调用的this值，他们是apply和call方法。

这两个方法都将第一个参数做为被调用上下文中使用的this值。两个方法的不同点是：apply方法的第二个参数应是一个数组，或者类似数组行为的对象，而calll可以接受任何参数。对于两个方法而言，第一个参数是必须的：也就是this的值。
例如：
{% codeblock lang:javascript %}
var b = 10;
 
function a(c) {
  alert(this.b);
  alert(c);
}
 
a(20); // this === global, this.b == 10, c == 20
 
a.call({b: 20}, 30); // this === {b: 20}, this.b == 20, c == 30
a.apply({b: 30}, [40]) // this === {b: 30}, this.b == 30, c == 40

{% endcodeblock %}

##总结
本文我们讨论了ECMAScript中的this关键字的特性，跟其他语言相比，这真的算是特性了。

