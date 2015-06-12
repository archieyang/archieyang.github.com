---
layout: post
title: "Smashing JavaScript: 执行上下文"
date: 2013-12-26 23:12:44 +0800
comments: true
categories: JavaScript
tags: [JavaScript]
---
本文内容翻译自 [ECMA-262 JavaScript. The core](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/#execution-context-stack)的部分内容。
##执行上下文栈
ECMAScript(JavaScript)中，有三种类型的代码：全局代码、函数代码和eval代码。每段代码都有自己的执行上下文。代码中只存在一个全局上下文，但是可以哟多个函数和eval上下文。每次发生函数调用，代码就进入了函数执行上下文并且执行函数的代码；每调用一个eval函数，就会进入eval执行上下文并执行其代码。

需要注意的是，一个函数可以产生无穷多个上下文，这是因为每次对于这个函数的调用（甚至是函数递归的调用自己）都会产生一个带有新状态的上下文。
<!-- more -->
{% codeblock lang:javascript %}
function foo(bar) {}
 
// call the same function,
// generate three different
// contexts in each call, with
// different context state (e.g. value
// of the "bar" argument)
 
foo(10);
foo(20);
foo(30);
{% endcodeblock %}

一个执行上下文肯能会激活另一个上下文，比如在一个函数中调用另一个函数（或者全局上下文调用一个全局函数），等等。逻辑上讲，这是通过一个栈实现的，这个栈被称为执行上下文栈。

当调用者调用被调用者，调用者挂起自己的执行，并将控制流传递给被调用者。被调用者被压入执行栈的栈顶,并且成为一个运行（活动）执行上下文。当这个被调用者的上下文结束时，它将控制权返还给调用者，而调用者继续执行之前的上下文（或者激活其他的上下文），直到它的上下文结束。一个被调用者可能return或者通过异常退出，一个没有被捕获的异常会导致一个或者多个上下文退出（出栈）。

例如，图中用一个执行上下文栈来表示所有的ECMAScript程序运行时，位于栈顶的是一个活动的上下文

![](/images/ec-stack.png)

在程序开始运行的时候，它进入全局执行上下文，全局执行上下文位于栈底，并且是栈的第一个元素。之后，全局代码进行一些初始化工作，创建需要的对象和函数。在全局上下文的执行过程中，它的代码可能会激活一些其他（已经被创建的）函数，这样执行就进入了这些函数的执行上下文，向栈中压入新的元素。在完成初始化之后，运行时系统等待事件的发生，这些事件会激活某些函数，并进入新的执行上下文。

在下图中，我们用EC1表示某函数的上下文，并用Global EC表示全局上下文，当进入和推出EC1时，上下文执行栈的变化如下：

![](/images/ec-stack-changes.png)

上图准确的表示了ECMA的运行时系统是如何管理代码的执行的。

##执行上下文

一个执行上下文可以被抽象的表示为一个简单的对象。每个执行上下文都是一个用于追踪其代码执行过程必要的属性的集合。下图显示了一个上下文的结构：

![](/images/execution-context.png)

除了以上列出的三个属性（变量对象、this值和作用域链，根据实现的不同执行上下文可能还会包含其他的属性。

下面我们来仔细分析这三种最重要的属性。
###变量对象
>一个变量对象是一组和执行上下文相关的数据。它是一个上下文相关的特殊对象，并且保存了在上下文中定义的变量和函数声明。

需要注意的是，函数表达式（和函数声明不同）并没有包含在变量对象中。

一个变量对象是一个抽象的概念。在不同的上下文类型中，它被作为不同的对象呈现。例如，在一个全局上下文中，变量对剑就是全局对象自身（这就是为什么我们可以通过全局对象的属性名来引用全局变量）。

我们看下面这个全局执行上下文的例子：
{% codeblock lang:javascript %}
var foo = 10;
 
function bar() {} // function declaration, FD
(function baz() {}); // function expression, FE
 
console.log(
  this.foo == foo, // true
  window.bar == bar // true
);
 
console.log(baz); // ReferenceError, "baz" is not defined
{%  endcodeblock %}

全局上下文的变量对象的属性表示如下：

![](/images/variable-object.png)

可以发现，函数baz作为一个函数表达式，并没有包含在变量对象中。因此当我们试图在函数外访问它时，产生了一个ReferenceError。

注意，和其他语言不同，在ECMAScript中，只有函数能够创建一个新的作用域。（注：ECMAScript中没有块作用域，这对于拥有自其他语言背景的学习者来说是很难接受的。）在一个函数作用域内部定义的变量和内部函数对于外部来说是不可见的，这样就防止了对全局变量对象的污染。

我们还可以通过使用eval进入一个新的（eval的）执行上下文。但是在eval中使用的变量对象是全局变量对象，或者是调用者的变量对象（比如调用eval的函数的变量对象）。（注：eval是JavaScript的毒瘤/糟粕之一，尽量不要使用）
那函数和函数的变量对象又是什么样？在一个函数上下文中，函数的变量对象被表示为一个活动对象。

###活动对象
当一个函数被激活（调用），一个被称为活动对象的特殊对象就被创建了。它被填入了形式参数（形参）和一个特殊的argments对象（一个带有索引的形参映射）。之后，这个活动对象就被当做当前函数上下文的变量对象使用。

例如，一个函数的变量对象还是前文提到的简单的变量对象，但是除了变量和函数声明外，它还包含了形式参数和arguments对象，并被称作活动对象。
看下面这个例子
{% codeblock lang:javascript %}
function foo(x, y) {
  var z = 30;
  function bar() {} // FD
  (function baz() {}); // FE
}
 
foo(10, 20);
{% endcodeblock %}
foo函数上下文的活动对象如下图所示：

![](/images/activation-object.png)

再次需要注意：函数表达式baz并没有被包含在变量/活动对象中。
注意：在ES5中变量对象和活动对象的概念被组合到了词法环境（lexical environments）模型中。

###作用域链
>作用域链是在查找上下文代码中的标识符时被搜索的一系列对象。

这里的规则很简单，和原型链类似：如果一个变量没有在它自己的作用域中被找到（在它自己的变量/活动对象中），那么就到上一级的变量对象中去寻找，并如此往复进行下去。

考虑上下文环境，标识符包括：变量名、函数声明、形参等等。但一个函数在其代码中引用了一个标识符，并且这个标识符并不是一个局部变量（或者局部函数或者形参）时，这个变量被称为自由变量。要找到自由变量，就需要使用作用域链。

通常情况下，作用域链是一个包含所有父级的变量对象的序列，加上（在作用域链的头部）函数自身的变量/活动对象。但是，作用域链也可能包含其他对象，比如在上下文执行过程中国被动态加入到作用域链中的对象---比如with对象或者特殊的catch从句对象。（注：with关键字是一个非常不好的特性）

当解析（寻找）一个标识符时，作用域链的搜索从活动对象开始，如果在活动对象中没有找到标识符，就沿着作用域链一直向上搜索，就像在原型链上那样。

{% codeblock lang:javascript %}
var x = 10;
 
(function foo() {
  var y = 20;
  (function bar() {
    var z = 30;
    // "x" and "y" are "free variables"
    // and are found in the next (after
    // bar's activation object) object
    // of the bar's scope chain
    console.log(x + y + z);
  })();
})();
{% endcodeblock %}

我们假设作用域链是通过隐式的\_\_parent\_\_属性来指向作用域链中的下一个对象的。这方方法可以再Rhino代码中验证，并且这种技术也被用于ES5的词法环境（ES5中有一个名为outer的连接）。另一种表示作用域链的方法是采用一个简单的数组。采用\_\_parent\_\_的概念，我们可以用下图表示上面的示例代码（父辈的变量对象被保存于函数的[[scope]]属性中）：

![](/images/scope-chain.png)

在代码执行时，可以通过with和catch来增加作用域链中的对象。由于这些对象是简单对象，他们有可能会包含原型和原型链。这导致了作用域链的查询包含两个维度：一个是作用域链的连接，另一个是每一个作用域链上深入连接对象的原型链（如果这个对象有原型的话）。

看下面这个例子：

{% codeblock lang:javascript %}
Object.prototype.x = 10;
 
var w = 20;
var y = 30;
 
// in SpiderMonkey global object
// i.e. variable object of the global
// context inherits from "Object.prototype",
// so we may refer "not defined global
// variable x", which is found in
// the prototype chain
 
console.log(x); // 10
 
(function foo() {
 
  // "foo" local variables
  var w = 40;
  var x = 100;
 
  // "x" is found in the
  // "Object.prototype", because
  // {z: 50} inherits from it
 
  with ({z: 50}) {
    console.log(w, x, y , z); // 40, 10, 30, 50
  }
 
  // after "with" object is removed
  // from the scope chain, "x" is
  // again found in the AO of "foo" context;
  // variable "w" is also local
  console.log(x, w); // 100, 40
 
  // and that's how we may refer
  // shadowed global "w" variable in
  // the browser host environment
  console.log(window.w); // 20
 
})();
{% endcodeblock %}

我们绘制的结构如下图（即，在我们搜索\_\_parent\_\_之前，我们先搜索\_\_proto\_\_链）：

![](/images/scope-chain-with.png)

注意，并不是在所有的实现中，全局对象都继承于Object.prototype。图中所示的行为可以再SpiderMonkey中验证。

在所有父亲对象存在的时候，从内部函数获取父亲和祖先的数据并没有什么特别之处，我们只需要遍历作用域链来解析（搜索）需要的变量。但是，如我们之前提到的，当一个上下文结束后，它所有的状态和它本身都被销毁了。但与此同时，内部函数则可能从父亲对象被返回。更进一步的，这个被返回的函数之后可能在另一上下文中被激活。__当某些自有变量所在的上下文已经不存在的时候，这样的激活会发生什么？在通用的理论上，有一个被称为（词法）闭包的概念被用于解决这个问题。__在ECMAScript中，这与作用域链的概念直接相关。

###闭包

在ECMAScript中，函数是一级的对象（first-class objects）。这个术语意味着函数可以被作为参数传递给其他函数（在这种情况下他们被称为"funargs"，"functional arguments"的缩写）。能够接受"funargs"的函数被称为高阶函数，或者按照数学上的名词，成为运算符。函数同样可以从其他函数被返回。返回其他函数的函数被称为以函数为值的函数。

有两个和"funargs"以及"functional values"相关的问题，这两个问题被统称为"Funarg problem"或者"A problem of a functional argument"。为了解决所有的"funarg problem"，闭包的概念应运而生。让我们更细节的描述下这两个子问题（我们将会看到这两个问题在ECMAScript中都是使用图中所提到的函数的[[Scope]]属性来解决的）。

第一中国"funarg problem"的子类型是"upward funarg problem"。它出现于一个函数从另一个函数中被返回到上一层（外部），而它包含了上面提到的自由变量的时候。为了能够在父亲上下文结束后，仍然访问父亲上下文中的变量，内部函数在创建时将它付钱的作用域链保存到自己的[[Scope]]属性中。之后，当函再次被激活时，它上下文的作用域链由活动对象和这个[[Scope]]属性组成。
{% codeblock lang:javascript %}
Scope chain = Activation object + [[Scope]]
{% endcodeblock %}

注意这里主要发生的事情：在创建的时候，一个函数保存了它父对象的作用域链，因为这个被保存的作用域链会在之后函数的调用中被用于查找变量。
{% codeblock lang:javascript %}
function foo() {
  var x = 10;
  return function bar() {
    console.log(x);
  };
}
 
// "foo" returns also a function
// and this returned function uses
// free variable "x"
 
var returnedFunction = foo();
 
// global variable "x"
var x = 20;
 
// execution of the returned function
returnedFunction(); // 10, but not 20
{% endcodeblock %}
这种形式的作用域被称为静态（或者词法）作用域。我们发现变量x在被返回的bar函数的[[Scope]]中被找到。在通用理论中，有一种动态作用域，在这种作用域中，示例中的变量x应被解析为20而不是10.但是，ECMAScript中没有使用动态作用域。

"funarg problem"的另一个问题是"downward funarg problem"。在这种情况中，父上下文仍然存在，但是在解析标识符时，会存在二义性。具体问题是：标识符应该使用来自哪个作用域的值？是在函数创建时静态存储的，还是在执行时动态生成的？为了避免这种二义性，ECMAScript决定使用静态的作用域。

{% codeblock lang:javascript %}
// global "x"
var x = 10;
 
// global function
function foo() {
  console.log(x);
}
 
(function (funArg) {
 
  // local "x"
  var x = 20;
 
  // there is no ambiguity,
  // because we use global "x",
  // which was statically saved in
  // [[Scope]] of the "foo" function,
  // but not the "x" of the caller's scope,
  // which activates the "funArg"
 
  funArg(); // 10, but not 20
 
})(foo); // pass "down" foo as a "funarg"
{% endcodeblock %}

我们可以得出结论，如果要在语言中包含闭包，那它必须具有静态的作用域。但是，某些语言可能提供一种动态和静态的作用域组合，允许程序员来选择哪些需要采用闭包，哪些不需要。由于ECMAScript中只有静态作用域，结论是：ECMAScript完全支持闭包，从技术上这是通过函数的[[Scope]]属性实现的。现在我们可以给出一个完整的闭包的定义了：
>闭包是一个代码块（在ECMAScript是一个函数）和以静态/词法方式存储的所有父作用域的一个集合体。因此，函数可以通过这些被存储的作用域很容易的找到自由变量。

注意，由于每个（正常情况下）函数都在被创建时保存[[Scope]]属性，理论上讲，ECMAScript中所有的函数都是闭包。

另一个需要注意的地方是，某些函数可能具有相同的父作用域（拥有两个内部或者全局函数是很常见的情况）。在这种情况下，存储在[[Scope]]中的变量被所有具有相同父作用域的函数共享。一个闭包中对变量的改变会反应到其它闭包对这个变量的读取中：

{% codeblock lang:javascript %}
function baz() {
  var x = 1;
  return {
    foo: function foo() { return ++x; },
    bar: function bar() { return --x; }
  };
}
 
var closures = baz();
 
console.log(
  closures.foo(), // 2
  closures.bar()  // 1
);
{% endcodeblock %}

这段代码可以通过一下这幅图表示：
![](/images/shared-scope.png)

当在一个循环中创建几个函数时，这个特性的迷惑性就显现出来了。在一个被创建的函数中使用循环计数器，很多程序员经常得到出乎意料的结果---这些函数中的计数器拥有相同的值。现在我们应清楚它为什么是这样：因为所有的函数拥有相同的[[Scope]]，而被创建的函数中的循环计数器保存的是最后被赋的值。

{% codeblock lang:javascript %}
var data = [];
 
for (var k = 0; k < 3; k++) {
  data[k] = function () {
    alert(k);
  };
}
 
data[0](); // 3, but not 0
data[1](); // 3, but not 1
data[2](); // 3, but not 2
{% endcodeblock %}

有许多技巧可以解决这个问题。其中一个是在作用域链中提供一个额外的对象-比如使用一个额外的函数：
{% codeblock lang:javascript %}
var data = [];
 
for (var k = 0; k < 3; k++) {
  data[k] = (function (x) {
    return function () {
      alert(x);
    };
  })(k); // pass "k" value
}
 
// now it is correct
data[0](); // 0
data[1](); // 1
data[2](); // 2
{% endcodeblock %}

下一节我们将讨论执行上下文的最后一个属性-this值。

###this值

>this值是一个和执行上下文相关的特殊的对象。因此，他可以被称为上下文对象（例如，一个在执行上下文被激活的上下文中的对象）。

任何对象都可以被用作上下文的this值。我想在这里澄清一些在描述ECMAScript中执行上下文中出现的错误的概念，特别是关于this值的。许多情况下，this值，被错误描述为一个变量对象的属性。请记住：
>this值是一个执行上下文的属性，而不是变量对象的属性。

这个特性非常重要，因为和变量不同，this值不参与标识符解析过程。例如，当在代码中访问this值时，它的值直接从执行上下文中获取，而不需要进行作用域链查找。this的值只在进入上下文时被确定。

和ECMAScript不同，Python把方法的self参数当做一个普通的变量，这个变量和其他值一样被解析，并且甚至可以在执行时被改变值。在ECMAScript中，是不可以为this赋新的值的，因为，再重复一遍，它不是一个变量，它并不存在于变量对象中。

在一个全局上下文中，this值就是全局对象本身（这意味着，this值在这里等于变量对象）：
{% codeblock lang:javascript %}
var x = 10;
 
console.log(
  x, // 10
  this.x, // 10
  window.x // 10
);
data[2](); // 2
{% endcodeblock %}

在一个函数上下文中，this值在每一次函数调用时都可能不同。这里this的值由调用者通过调用表达式（函数调用的形式）提供。例如，下面的foo函数是一个被调用者，在一个全局上下文中被调用，全局上下文就是调用者。我们看看下面的例子，对于同一个函数的代码，this值在不同调用中被调用者提供了不同的值。

{% codeblock lang:javascript %}
// the code of the "foo" function
// never changes, but the "this" value
// differs in every activation
 
function foo() {
  alert(this);
}
 
// caller activates "foo" (callee) and
// provides "this" for the callee
 
foo(); // global object
foo.prototype.constructor(); // foo.prototype
 
var bar = {
  baz: foo
};
 
bar.baz(); // bar
 
(bar.baz)(); // also bar
(bar.baz = bar.baz)(); // but here is global object
(bar.baz, bar.baz)(); // also global object
(false || bar.baz)(); // also global object
 
var otherFoo = bar.baz;
otherFoo(); // again global object
{% endcodeblock %}

