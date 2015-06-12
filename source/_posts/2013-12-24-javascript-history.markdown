---
layout: post
title: "Smashing JavaScript: JavaScript的前世今生"
date: 2013-12-24 22:09:51 +0800
comments: true
categories: JavaScript
tags: [JavaScript]
---

##What is JavaScript？


>JavaScript，一种直译式脚本语言，是一种动态类型、弱类型、基于原型的语言，内置支持类型。它的解释器被称为JavaScript引擎，为浏览器的一部份，广泛用于客户端的脚本语言，最早是在HTML网页上使用，用来给HTML网页增加动态功能。然而现在JavaScript也可被用于网络服务器，如Node.js。


>在1995年时，由网景公司的布兰登·艾克，在网景导航者浏览器上首次设计实现而成。因为网景公司与升阳公司合作，网景公司管理层次结构希望它外观看起来像Java，因此取名为JavaScript。但实际上它的语法风格与Self及Scheme较为接近。



>为了取得技术优势，微软推出了JScript，CEnvi推出ScriptEase，与JavaScript同样可在浏览器上运行。为了统一规格，1997年，在ECMA（欧洲计算机制造商协会）的协调下，由Netscape、Sun、微软、Borland组成的工作组确定统一标准：ECMA-262。因为JavaScript兼容于ECMA标准，因此也称为ECMAScript。
												---维基百科

<!-- more -->
据说Brendan Eich只花了十天设计这门语言，但这门产生略显仓促的语言随着Web的风靡，逐渐成为了这个世界上最流行的编程语言之一。2009年3月，Ryan Dahl基于JavaScript的V8引擎创建了一个轻量级的Web服务器并提供了一套库，称为Node。这也标志的JavaScript作为浏览器语言向服务器端发起冲击。由于V8是Google开发的浏览器引擎，我曾误以为Node也是Google公司的作品，而实际上Node初期属于Ryan Dahl的个人项目，之后收到了Joyent公司的大力支持，除了采用了Google的V8引擎外，和Google公司并无关系。

初学JavaScript很好奇的一点就是这门语言的版本号，实际上JavaScript是有一个类似于Java 8、Python 2.7和Ruby 2.0这样的版本号的，比如JavaScript 1.7。但是当人们谈论JavaScript的特性时，基本都是基于ECMA标准讨论的，也就是ECMA-262。目前最新的版本是ECMA-262-6，也就是这个标准的第六版。这是因为JavaScript只是兼容于ECMA-262标准的一个实现，如维基百科所述，真正决定在这个语言中引入新特性的是ECMA-262这个标准。

##Is JavaScript good?
>JavaScript建立在一些非常优秀的想法和少数非常糟糕的想法之上。
									---Douglas Crockford
									
作为一门只用了十天时间设计的语言，JavaScript后来的发展超出了所有人的预期，在一统浏览器前端开发之后，随着Node的兴起，JavaScript大有一统前后端开发之势。但是早起仓促的开发时间不可避免的让这门语言在设计上有许多疏于考虑的地方，比如臭名昭著的基于全局变量的编程模型。

同时，能够发展到今天，JavaScript必然有其过人之处。我认为JavaScript最大的有点在于其first-class function的设计。目前编程语言的趋同性非常明显，许多语言开始借鉴其他语言中优秀的设计，比如Ruby中的内部迭代器就出现在了ECMA-262-5以及Java 8中，同时，现在还是草案的ECMA-262-6的重点就是包括引入生成器（Generators）和List Comprehesion，它的设计几乎和Python这两个特性一模一样。但是，这些特性的引入与语言本身的特性结合，究竟能产生怎样的化学反应，就要依赖于语言本身原有的特性了。而JavaScript中firts-class function和基于原型继承的实际，使得引入这些特性后具有更大的灵活性。

##Is JavaScript popular?
作为一门前端语言而言，JavaScript已经一统浏览器开发，成为了浏览器中的通用语言，这里无需多言。作为一门新兴的服务器端开发语言（虽然《深入浅出Node.js》中把服务器端的JavaScript起源追溯到了1994年的LiveWire，但是真正目前可用的服务器端JavaScript的历史还只能从2009年Node诞生算起。），Node在GitHub上的关注程度已经超过了Ruby on Rails，并被PayPal和Linkedin大规模应用到实际开发中。从开源框架发展看，类似于Sinatra的Express目前是最为流行的Node Web框架，他的作者TJ基于ECMA-262-6新特性开发的新框架Koa也得到了广泛关注。旨在取代Wordpress成为下一代私人博客平台的Ghost也在去年成功募集到资金。NPM上模块数量的增长速度也极惊人。从整体看，服务器端Node基于事件的设计和传统的服务器端开发有很大不同，其实时高并发的特性符合目前的趋势，这使其必然在服务器端开发中占有一席之地，甚至在某些Web服务中会出现JavaScript统一前后端开发的情景。

