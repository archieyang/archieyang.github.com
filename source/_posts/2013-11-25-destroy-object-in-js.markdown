---
layout: post
title: "由console.log想到的对象内容破坏"
date: 2013-11-25 15:51:44 +0800
comments: true
categories: JavaScript
tags: [JavaScript]
---

##起因
在开发Chrome插件时候发现用console.log打印出的一个对象如下图所示：
![](/images/destroyed-object.jpg)
很明显Object行打印出的对象的startDate值和展开后内部的startDate值不一致
<!-- more -->
##分析
涉及这个部分的代码抽出后如下
{% codeblock lang:javascript %}
var log = function(s) {
	console.log(s);
};
var main = function (){
	var obj,
		functionX = function(param) {
			var localDate;
			//don't do this
			for (localDate = param.startDate; localDate <= param.endDate; localDate.setDate(localDate.getDate()+1)) {
				//do something
			}; 
		};

	obj = {};
	obj.startDate = new Date(2013,10,25);
	obj.endDate = new Date(2013,10,29);

	log(obj);
	functionX(obj);

};
main();
{% endcodeblock %}
这里functionX()在遍历startDate到endDate之间的值时，修改了param.startDate的值，因为functionX()只是利用startDate和endDate作为边界，没有必要修改startDate值，所以__这是一个非常坏的做法__。

但是__这不是导致对象打印和展开值不一致的理由。__因为log(obj)打印对象发生在functionX(obj)之前，打印的对象应该是对象的初始值，即new Date(2013,10,25)和new Date(2013,10,29)，正如在console第一行打印的那样。

但是为什么对象内部会显示修改后的值？这是由于初始打印时，只有object一行，这时执行的是log(obj),打印console的第一行，即对象的原始状态。当在console中点开这个对象时，显示下面的具体属性，这个操作发生在function(obj)之后，在这时再窥视对象内部，这个obj已经是被functionX()修改过的obj了。

__也就是说，在点开console中的一个对象时，展示在console中的对象内容，并不是调用console.log时候的对象内容，而是程序运行结束后的对象内容。从视觉上很容易误解为，展开的对象是执行console.log时的对象。__

##修改
这段代码除了在console中查看时容易引起误解外，本身是可以正确运行的。但是这是一个非常糟糕的做法，如果functionX()在另一个脚本文件中，functionX()调用者并不希望obj的状态被更改，尤其是作为上下界存在的startDate和endDate。最好的做法是对startDate进行拷贝，修改后的functionX()如下：
{% codeblock lang:javascript %}
functionX = function(param) {
	var localDate;
	//copy
	for (localDate = new Date(param.startDate.getTime()); localDate <= param.endDate; localDate.setDate(localDate.getDate()+1)) {
		//do something
	}; 
};
{% endcodeblock %}
打印效果如下，此时打印时对象状态和展开时的对象状态一致。
![](/images/restored-object.jpg)
##最后
1. 由于JavaScript函数传递的是参数的引用，因此如果不是必须的情况，一定不要修改参数内部的值，这可能会给函数的调用者带来__意想不到的的副作用__。
2. 在Chrome中， 打印对象时显示的是执行console.log时的对象状态，而在console中通过点击展开对象时，显示的是点击发生时的对象状态（很可能是程序已经执行结束），这种展示方法在视觉上还是很有误导性的。
