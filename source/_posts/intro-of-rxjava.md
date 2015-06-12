title: "RxJava初探"
date: 2015-05-09 08:19:28
categories: Android
tags: [RxJava, Android]
---
#0.前言
本文主要记录了初步学习RxJava后的总结，希望用最短的篇幅讲清楚RxJava的主要用法。部分内容来自Dan Lew的[Grokking RxJava](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)。

本文的示例代码在[这里](https://github.com/archieyang/RxJavaDemo)。

#1 基本概念
##1.1 Rx结构
响应式编程的主要组成部分是observable, operator和susbscriber（与Dan Lew的文章不同，这里把Operator也做为组成部分介绍，这样对结构的整体性会有更全面的认识）。
一般响应式编程的信息流如下所示：

Observable -> Operator 1 -> Operator 2 -> Operator 3 -> Subscriber

也就是说，observable是事件的生产者，subscriber是事件最终的消费者。

**因为subscriber通常在主线程中执行，因此设计上要求其代码尽可能简单，只对事件进行响应，而修改事件的工作全部由operator执行。**
##1.2 最简单的模式
如果我们不需要修改事件，就不需要在observable和subscriber中插入operator。这时的Rx结构如下：

Obsevable -> Subscriber

这看起来很像设计模式中的观察者模式，他们最重要的区别之一在于在没有subscriber之前，observable不会产生事件。

一个简单的RxJava HelloWorld的代码如下。
{% codeblock lang:java %}
        // 创建observable
        Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("Hello RxJava");
                subscriber.onCompleted();
            }
        });

        // 创建subscriber
        Subscriber<String> subscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }
            @Override
            public void onNext(String s) {
                log(s);
            }
        };
        
        // 订阅
        observable.subscribe(subscriber);
{% endcodeblock%}

<!-- more -->
这里的代码对于一句简单的HelloWorld的而言太繁琐了。因此，RxJava提供了一些简化的方法。

首先是创建observable，如果我们只需要发送一个事件（这里的事件是字符串"Hellow RxJava"），我们可以使用Observable类的just方法，简化后的代码如下

{% codeblock lang:java %}
        // 创建observable
        Observable<String> observable = Observable.just("Hello RxJava");
{% endcodeblock%}

同样，如果我们不关心subscriber是否结束（onComplete())或者发生错误(onError()),subscriber的代码可以简化为

{% codeblock lang:java %}
        // 创建subscriber
        Action1<String> subscriber = new Action1<String>() {
            @Override
            public void call(String s) {
                log(s);
            }
        };
{% endcodeblock%}
我们直接把创建和订阅连接起来，完整的代码如下。
{% codeblock lang:java %}
        Observable.just("Hello RxJava").subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                log(s);
            }
        });
{% endcodeblock%}

最后，使用Java 8的lambda(Android上可以使用[Retrolambda](https://github.com/evant/gradle-retrolambda))，这个HelloWorld的最终版本如下：

{% codeblock lang:java %}
Observable.just("Hello RxJava")
    .subscribe(s -> log(s));
{% endcodeblock%}

##1.3 加入operator

很多时候，我们需要针对处理过的事件做出响应，而不仅仅是Observable产生的原始事件。由于1.1中阐述的原因，这里就需要引入operator来处理原始事件。

这里以一个极简单的Markdown处理为例：假设输入的是Markdown格式的文件，最终展示文字的是一个WebView，这里就需要引入一个将Markdown转为HTML的operator，其代码如下：

{% codeblock lang:java %}
        Observable.just("#Basic Markdown to HTML").map(new Func1<String, String>() {
            @Override
            public String call(String s) {
                if(s != null && s.startsWith("#")) {
                    return "<h1>" + s.substring(1, s.length()) + "</h1>";
                }
                return null;
            }
        }).subscribe(s -> log(s));
{% endcodeblock%}

这里使用了名为map()的operator，它的作用很简单，就是接收一个事件，并返回处理后的事件。Func1的第一个泛型参数表示输入类型，第二个繁星参数表示返回类型。

我们这里同样可以采用lambda来简化代码，简化后的代码如下：
{% codeblock lang:java %}
        Observable.just("#Basic Markdown to HTML with lambda")
                .map(s -> s != null && s.startsWith("#") ? "<h1>" + s.substring(1, s.length()) + "</h1>" : null)
                .subscribe(s -> log(s));
{% endcodeblock%}

##1.4 Subscription

前三小节有意隐藏了RxJava的一个细节，实际上执行Observable.subscribe()时，它会返回一个Subscrition,它代表了Observable和Subscriber之间的关系。你可以通过Subscrition解除Observable和Subscriber之间的订阅关系，并立即停止执行整个订阅链。示例代码如下：
{% codeblock lang:java %}
        Subscription subscription = Observable.just("Unsubscribe me later").subscribe(s -> log(s));
        subscription.unsubscribe();
        log("isSubscribed = " + subscription.isUnsubscribed());
{% endcodeblock%}

#3 多线程
在开发过程中，为了避免阻塞UI线程，我们可能需要将某些工作放到指定线程执行。在RxJava中，你可以通过subscribeOn()来指定Observer的运行线程，通过observeOn()指定Subscriber的运行线程。这两个方法都是operator，因此它们可以像所有operator那样作用于任何的Observable。一个简单的例子如下：

{% codeblock lang:java %}
        Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                log("Observable on Thread -> " + Thread.currentThread().getName());
                subscriber.onNext("MultiThreading");
                subscriber.onCompleted();
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(s -> {
                    log("Subscriber on Thread -> " + Thread.currentThread().getName());
                });
{% endcodeblock%}
#4 错误处理
RxJava使用Subscriber的onError()进行错误处理。每一个Obervable的执行最后一定会调用onCompleted()和onError()方法中的一个。相比于传统的回调处理错误的方式，订阅链中任何时候出现的错误，都只需要在Subscriber的onError()方法中处理，operator不需要处理异常。


#5 小结

相比于Otto这种总线式的处理方式，RxJava对于订阅事件的处理更精细。同时，它还引入了许多函数式编程的特性，对于信息流处理有更好的解耦。目前只是通过阅读以及一些玩具代码初步了解了其用法，这仅仅是个开始。希望在实际项目中使用后，能有时间总结诸如自定义Operator等更多的高级用法。

##参考
[Grokking RxJava](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)