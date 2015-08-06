title: 再谈MVP与MVVM
date: 2015-08-06 18:50:42
comments: true
categories: Software Engineering
tags: [MVP, MVVM, Passive View, Supervising Controller]
---
#0.  前言
之前写过[3篇博客](http://codethink.me/tags/MVP/)分析Philm MVP架构的实现，最近在接触到一些关于clean architecture的话题后，又仔细读了一些关于MVP以及MVVM的文章，觉得当时的一些理解不够深入，因此针对应用架构的理论部分重新做出总结。
#1. Model-View-Presenter
Martin Fowler的这篇[Retirement note for Model View Presenter Pattern](http://martinfowler.com/eaaDev/ModelViewPresenter.html)文章明确表明了，这个模式已经“退休”了。作为MVC模式的变形之一，它被重新归纳为了MVC模式的两种变形：Passive View和Supervisor controller。
##1.1 Passive View
Passive View是MVC模式的一种变形。对于UI而言，它被分为负责显示的View和接收用户交互的Controller。Passive View是一个完全被动的View，不负责UI更新，因此也不依赖Model。

这种模式强调的是View与Model之间的独立，Controller负责全部的View与Model的同步工作，所有的消息都需要通过Controller传递。
<!-- more -->

##1.2 Supervising Controller
Supervising Controller模式强调Controller只处理复杂逻辑和部分与View和Model之间的同步。Controller中尽可能减少View与Model的同步工作，将其转交数据绑定来实现，Controller只处理同步机制无法处理的复杂的交互和同步。因此，这种模式中View与Model需要直接通信，存在依赖关系。

#2. Model-View-ViewModel
ViewModel可以同时和多个Domain Model交互，它并不是一个特定Domain Model为了适合View而构建的外观模式。我们可以认为ViewModel是一个特殊的，不依赖于系统GUI框架的View。此外，需要注意的是。一个ViewModel可以应用于多个View，但是一个View只能拥有一个ViewModel。

所有的事件和响应逻辑都需要ViewModel来处理，因此它的View变得很简单，这与Passive View模式很相似。它与Passive View的区别在于: ViewModel中不包含控件，控件需要依靠自己获取ViewModel中的状态。而Passive View中的Controller可以包含控件，这降低了控件自己获取状态带来的风险，但是，Passive View也因此引入了新的问题：它依赖于系统GUI控件，无法独立进行测试。

#3.小结
总结一下3种模式的特点：
1) Passive View： 强调View与Model的独立，Controller负责全部的同步工作。
2) Supervising Controller：强调Controller中只处理复杂逻辑，尽可能少的负责View与Model间的同步工作。View与Model相互依赖。
3）MVVM：强调ViewModel作为一个View的抽象层，其与实际的View一对多的关系，同时其自身不依赖系统控件，方便独立测试。

#参考
1. [Retirement note for Model View Presenter Pattern](http://martinfowler.com/eaaDev/ModelViewPresenter.html)
2. [GUI Architectures](http://martinfowler.com/eaaDev/uiArchs.html)
3. [Passive View](http://martinfowler.com/eaaDev/PassiveScreen.html)
4. [Supervising Controller](http://martinfowler.com/eaaDev/SupervisingPresenter.html)
5. [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html)
