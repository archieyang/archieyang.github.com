---
layout: post
title: "Smashing JavaScript: 对象和原型继承"
date: 2013-12-25 21:10:41 +0800
comments: true
categories: JavaScript
tags: [JavaScript]
---
##基于原型的继承
之前说过，我觉JavaScript最优秀的特性在于它提供了first-class function，但是它最有特点的特性应该是原型继承了。许多函数式编程语言都提供了first-class function，而原型继承现在常见的编程语言中只有JavaScript还在使用。

我们常见的继承都是基于类的继承，对象是类的实例，一个类可以从另一个类继承，从而实现代码的复用。在发现JavaScript基于原型继承的方法后，我的第一反映是：这还是面向对象吗。当然是，面向对象的名字中包含的是对象，它并不是面向类。实际上类只是对象的一个抽象，既然类最终都是要用来建立对象的，那不如就直接让对象继承对象，这也就是另一种继承的实现方式：基于原型的继承。

<!-- more -->
##原型是什么
原型实际上是对象的一个隐式的属性，在对象内部通过[[Prototype]]这个属性来表示其原型。在下文的图中，将用\_\_proto\_\_来表示这个属性。
原型究竟是怎么样一种关系？看下面这个例子，foo作为一个对象，拥有两个自有属性x和y，同时它还具有一个隐式的\_\_proto\_\_属性，这个属性指向的就是foo对象的原型。为什么要生成原型这个属性？在回答这个问题之前，我们需要了解下原型链。
{% codeblock lang:javascript %}
var foo = {
  x: 10,
  y: 20
};
{% endcodeblock %}

![](/images/basic-object.png)

##原型链
原型对象（一个对象原型属性所指向的对象）和普通的对象没有区别，他们也拥有自己的自有属性。一个对象的原型属性指向其原型对象，而其原型对象的原型属性指向其原型对象的原型对象，这样一直下去，就形成了一条原型链。
>原型链是用于实现继承和属性共享的由有限个对象组成的对象链。

考虑这样一种情况，我们需要两个对象，a和b，他们只有很少一部分是不同的，我们需要通过继承来复用他们相同部分的代码。JavaScript中没有类型的概念，所以需要通过原型链来实现继承，从某种程度上说，这种继承方式更加灵活。这种继承方式也被称为基于委托的继承。看下面这段代码，对象a中存储了b和c中相同的部分，而b和c只需要存储他们独有的部分。
{% codeblock lang:javascript %}
var a = {
  x: 10,
  calculate: function (z) {
    return this.x + this.y + z
  }
};
 
var b = {
  y: 20,
  __proto__: a
};
 
var c = {
  y: 30,
  __proto__: a
};
 
// call the inherited method
b.calculate(30); // 60
c.calculate(40); // 80
{% endcodeblock %}

看起来非常简单不是吗？我们发现b和c能够使用a中定义的calculate方法，这正是通过原型链实现的。

原型继承的规则很简单：如果对象在自己的自有属性中没有找到某个属性活方法，那么它就会尝试到原型链中寻找这个属性或者方法。如果仍然没找到，那么它会继续到其原型对象的原型对象中寻找，如此继续下去，直到找遍整个原型链。在寻找过程中，第一个被找到的名字符合的属性或方法会被使用，而寻找也就此停止。被找到的属性被称为继承属性，如果找遍整个原型链仍然没有找到符合的属性或方法，那将反悔undefined。

需要注意的是，在一个继承函数中的this值指向的是原来的对象，而不是包含该函数的（原型）对象。例如，上面程序中的this.y来自对象b和c，而不是a，这就验证了this指向的是原来的对象b和c。而this.x通过原型链获取到了a中x的值。

如果没有指定一个对象的原型，那么默认的\_\_proto\_\_指向Object.prototype。而Object.prototype自己仍然有一个\_\_proto\_\_属性，它指向null。

下图表示了对象a,b,c的继承关系：

![](/images/prototype-chain.png)

注意，ES5标准化了一种采用Object.create函数的方式来实现基于原型的继承：
{% codeblock lang:javascript %}
var b = Object.create(a, {y: {value: 20}});
var c = Object.create(a, {y: {value: 30}});
{% endcodeblock %}
ES6标准化了\_\_proto\_\_属性，它可以被用来初始化对象。

##蹩脚的/识时务的构造函数
Douglas Crockford在JavaScript: The Good Parts中认为JavaScript的构造函数掩盖了它原型继承的机制，是一个多余的中间层。JavaScript的构造函数是一个见仁见智的问题，它的确给从其他主流语言过度到JavaScript的程序员带来的亲切感，而且从发展趋势看，ES6中甚至引入了class，但就我个人而言，我觉得这种设计确实有些多余，一方面学习原型继承并不难理解，另一方面引入class无疑又增加了JavaScript语言的复杂程度。
构造函数可以创建指定形式的对象，同时自动为其设置原型对象。这个原型对象存储在构造函数的prototype属性中。
我们以构造函数的形式重写上面对象b，c从a继承属性和函数的例子如下：
{% codeblock lang:javascript %}
// a constructor function
function Foo(y) {
  // which may create objects
  // by specified pattern: they have after
  // creation own "y" property
  this.y = y;
}
 
// also "Foo.prototype" stores reference
// to the prototype of newly created objects,
// so we may use it to define shared/inherited
// properties or methods, so the same as in
// previous example we have:
 
// inherited property "x"
Foo.prototype.x = 10;
 
// and inherited method "calculate"
Foo.prototype.calculate = function (z) {
  return this.x + this.y + z;
};
 
// now create our "b" and "c"
// objects using "pattern" Foo
var b = new Foo(20);
var c = new Foo(30);
 
// call the inherited method
b.calculate(30); // 60
c.calculate(40); // 80
 
// let's show that we reference
// properties we expect
 
console.log(
 
  b.__proto__ === Foo.prototype, // true
  c.__proto__ === Foo.prototype, // true
 
  // also "Foo.prototype" automatically creates
  // a special property "constructor", which is a
  // reference to the constructor function itself;
  // instances "b" and "c" may found it via
  // delegation and use to check their constructor
 
  b.constructor === Foo, // true
  c.constructor === Foo, // true
  Foo.prototype.constructor === Foo // true
 
  b.calculate === b.__proto__.calculate, // true
  b.__proto__.calculate === Foo.prototype.calculate // true
 
);
{% endcodeblock %}

代码对应的关系如下图所示：

![](/images/constructor-proto-chain.png)

从图中我们发现：如我们前面所述，每一个对象都有自己的原型。甚至构造函数Foo也有自己的\_\_proto\_\_，它指向的是Function.prototype。而Function.prototype的\_\_proto\_\_属性又指向了Object.prototype。而Foo的prototype属性只是它自身一个普通的属性，唯一特殊的在于它指定了有Foo创建的对象b和c的原型。

如果从形式上讨论类或者分类这个概念，那么构造函数和原型对象的合起来应该被称作一个类。实际上，Python的一阶动态类属性和方法解析的实现与此完全相同。从这个角度看，Python的类只是一种语法糖，它的实质和JavaScript中基于委托的继承相同。

注意：ES6中类的概念被标准化了，并且在构造函数之上实现了如前文描述的语法糖。从这个角度看，__原型链成为了基于类型继承的实现细节。__

###参考资料
[ECMA-262 JavaScript. The core](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/)

