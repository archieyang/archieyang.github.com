title: "Philm项目源码分解析(3): Presenter与Model的设计"
date: 2015-03-27 22:20:31
categories: Android
tags: [Android, MVP]
---
#0. 前言
本文是Philm源码分析系列的第三篇，重点分析Philm MVP结构中Presenter与Model的设计。本文选取了Philm中核心的部分Movie作为示例。

#1.Philm中的Model
Philm中的Model通过State来管理，具体到Movie中，MovieController中包含一个对于MoviesState的引用。而State的实现在ApplicationState中，与第二篇提到的AndroidDisplay类似，ApplicationState是一个包含了所有Model总和的类。
{% codeblock lang:java %}
public final class ApplicationState implements BaseState, MoviesState, UserState {
	...
}

{% endcodeblock%}
<!-- more -->
#2.Presenter与Model的设计
##2.1 View -> Presenter -> Model
我们从一次搜索的过程来介绍各个部分的通信。SearchFragment中，mSearchView设置了一个对于搜索文本的监听器，当进行搜索时需要调用Presenter注册的回调MovieController.MovieUiCallbacks的search方法(注册回调在SearchFragment的父类BasePhilmMovieFragment中)。至此，View接受用户输入的流程结束。

在SearchFragment的父类BasePhilmMovieFragment中调用attachUi挂载到MovieController并在MovieController中被设置了MovieController.MovieUiCallbacks回调。

响应点击事件业务逻辑的MovieController.MovieUiCallbacks的search方法在MovieController(也就是Presenter)中实现。search通过ui.getMovieQueryType()拿到具体搜索的类型，之后通过fetchMovieSearchResults进行搜索。搜索最后通过FetchTmdbSearchMoviesRunnable类实现，在获取到搜索结果后，通过mMoviesState.setSearchResult(searchResult)更新Model。
##2.2 Model -> Presenter -> View
Model的数据被更新后，需要通过Presenter(MovieController)来更新View（Model与View不能直接通信）。Philm通知Presenter的方式是通过时间总线来实现的。这里使用的是[Square的otto](http://square.github.io/otto/)。可以看到setSearchResult中，除了将mSearchResult更新为获取的数据外，还向事件总线发送了一个SearchResultChangedEvent事件。

{% codeblock lang:java %}
public final class ApplicationState implements BaseState, MoviesState, UserState {
    ...
    @Override
    public void setSearchResult(SearchResult result) {
        mSearchResult = result;
        mEventBus.post(new SearchResultChangedEvent());
    }
    ...
}

{% endcodeblock%}

在MovieController中，通过@Subscribe以及一个有唯一SearchResultChangedEvent的方法来接收SearchResultChangedEvent事件，最后通过populateUisFromQueryTypes更新UI。
{% codeblock lang:java %}
public class MovieController extends BaseUiController<MovieController.MovieUi,
        MovieController.MovieUiCallbacks> {
    ...
    @Subscribe
    public void onSearchResultChanged(MoviesState.SearchResultChangedEvent event) {
        populateUisFromQueryTypes(MovieQueryType.SEARCH, MovieQueryType.SEARCH_MOVIES,
                MovieQueryType.SEARCH_PEOPLE);
    }
    ...
}

{% endcodeblock%}

#3.最后
本文是Philm MVP模式分析系列的最后一篇。Philm中除了MVP模式，还有很多值得分析的地方，比如对依赖注入的使用等等，之后如果有时间会针对Philm的其他部分进行更为细致的研究。


