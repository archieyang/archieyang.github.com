---
layout: post
title: "Philm项目源码分解析(1): 基本概念"
date: 2015-03-26 18:54:57
categories: Android
tags: [Android, MVP]
---
#0. 前言

Watch Philm这一系列文章主要是对近期阅读Chris Banes的开源项目Philm源码的整理。Android开发本身没有提供类似MVP之类的框架，只是提供了构建应用的基础组件。这一方面降低了上手难度，提供了极大的开发时的灵活度，但是对于新手极易写出高耦合的坏味道的代码。Philm本身构建了一套MVP架构的实现，本文也就基于Philm项目的源码来分析在Android开发中如何利用MVP模式构建程序。

本文为这个系列的第一篇，重点介绍MVC,MVP以及MVVM架构的基本概念以及他们的区别。
<!-- more -->


#1. MVC
MVC是一个传统的主流UI框架，现在的MVC框架经过演变主要分为两种。

第一种是传统的MVC框架，如下图所示。传统的MVC框架使用了一个长期存在的View和一个可更换的Controller。例如，在一个登录界面，根据用户是否已经登录，可能会有不同的Controller来响应。

由于传统MVC框架中保留一个长期存在的View，所有的用户交互都要通过这个View实现，它对于前端开发是一个极为有效的模式。
![](/images/mvc.png)

另一种MVC被称为Model2 MVC，如图所示。有争论说Model2并不是一个完全的MVC框架，但是大多数的现代实现（比如Ruby on Rails）都把它归为MVC。
![](/images/model2_mvc.png)
**Model2的聪明之处在于它要求"fat models and thin controllers”，也就是讲更多的代码放入Model中，而不是Controller中。由于需要监听并响应View的输入，传统的MVC更倾向于重型Controller（也就是使Controller容纳更多的业务逻辑）。**

由于HTTP协议本身是无状态的，在不同的请求之间View本身并不需要保存其状态。大多数服务器端开发框架都使用Session Cookies保存应用程序状态，因此他们将Controller和View的通信设计为单项的就十分恰当了，但是这并不适用于前端框架。实际上，很多框架的Controller被精简为一个简单的路由机制。

#2. MVP
MVP模式是一种分化的MVC，相比于传统的MVC模式的不同之处在于：

1）View不能直接于Model通信；

2）Presenter包含对View的引用，并根据Model的变化来更新View。

![](/images/mvp.png)

MVP模式主要有两种实现：

1）Passive View: View被设计的尽可能简单，所有的表现和业务逻辑都放在Presenter中实现。

2）Supervising Controller： View中可以包含简单的逻辑。View中无法满足的业务逻辑才放在Presenter中实现。

在Android的MVP实现里通常包含4个要素

(1) View: 负责绘制UI元素、与用户进行交互(Activity或Fragment);

(2) View interface: View需要实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试

(3) Model: 负责业务Bean的操作。

(4) Presenter: 作为View与Model交互的纽带，承载了大部分的复杂逻辑。



#3. MVVM
![](/images/mvvm.png)
Model-View-ViewModel模式于MVP模式极为类似，二者最大的不同在于MVVM假设ViewModel(相当于MVP中的Presenter)中的变化会被自动反应到View中，这种双向绑定并不需要开发者维护，而是基于框架提供了数据绑定引擎实现。Angular 1.x就是基于MVVM模式实现的。


#参考资料

1. [http://blog.nodejitsu.com/scaling-isomorphic-javascript-code/](http://blog.nodejitsu.com/scaling-isomorphic-javascript-code/)
2. [http://blog.csdn.net/vector_yi/article/details/24719873](http://blog.csdn.net/vector_yi/article/details/24719873)
3. [http://www.cnblogs.com/indream/p/3602348.html](http://www.cnblogs.com/indream/p/3602348.html)



