title: 使用throttleFirst解决按钮多次响应的问题
date: 2015-12-20 10:01:58
tags: [Android, RxJava, RxBinding]
categories: Real World RxJava
---
#0. 前言
本文是[Real World RxJava](http://codethink.me/categories/Real-World-RxJava/)系列的第一篇，这个系列主要介绍如何使用RxJava解决Android开发中的一些实际问题。如果你还不了解RxJava的基本概念（Observable, Subscriber, Subscription等），建议先阅读[RxJava初探](http://codethink.me/2015/05/09/intro-of-rxjava/)。
#1. 问题描述
我们首先实现一个Android中最常见的点击按钮，弹出一个新的Fragment的效果，其代码如下。
{% codeblock lang:java %}   
        findViewById(R.id.normal_button).setOnClickListener(view -> {
            FragmentTransaction transaction = getFragmentManager().beginTransaction();
            transaction
                    .setCustomAnimations(R.animator.slide_up, R.animator.slide_down, R.animator.slide_up, R.animator.slide_down)
                    .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);

            transaction
                    .add(android.R.id.content, new DummyFragment())
                    .addToBackStack(DummyFragment.class.getSimpleName()).commit();
        });
{% endcodeblock %}

这段代码存在的问题在于当用户无意中多次点击到按钮时，会弹出多个Fragment，如图所示。
![](/images/double-tap-issue.gif)
<!-- more -->
#2. 解决方案
我们通过使用<a href="http://reactivex.io/RxJava/javadoc/rx/Observable.html#throttleFirst(long,%20java.util.concurrent.TimeUnit)">throttleFirst</a>操作符来解决按钮被多次点击的问题。throttleFirst允许设置一个时间长度，之后它会发送固定时间长度内的第一个事件，而屏蔽其它事件，在间隔达到设置的时间后，可以再发送下一个事件，如下图所示。
![](/images/throttleFirst.png)
同时，我们使用[RxBinding](https://github.com/JakeWharton/RxBinding)将点击事件转化为一个Observable，将上述间隔时间设置为动画时间，这样，在点击按钮后只会弹出一个Fragment，避免了多次响应的问题。完整代码如下：

{% codeblock lang:java %}
        Button throttleFirstButton = (Button) findViewById(R.id.throttle_first_button);
        RxView.clicks(throttleFirstButton)
                .throttleFirst(ANIMATION_TIME, TimeUnit.MILLISECONDS)
                .subscribe(aVoid -> {
                    FragmentTransaction transaction = getFragmentManager().beginTransaction();
                    transaction
                            .setCustomAnimations(R.animator.slide_up, R.animator.slide_down, R.animator.slide_up, R.animator.slide_down)
                            .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);

                    transaction
                            .add(android.R.id.content, new DummyFragment())
                            .addToBackStack(DummyFragment.class.getSimpleName()).commit();
                });
{% endcodeblock %}

#参考
1. <a href="http://reactivex.io/RxJava/javadoc/rx/Observable.html#throttleFirst(long,%20java.util.concurrent.TimeUnit)">http://reactivex.io/RxJava/javadoc/rx/Observable.html#throttleFirst(long,%20java.util.concurrent.TimeUnit)</a>