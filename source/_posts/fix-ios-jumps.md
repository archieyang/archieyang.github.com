title: "iOS开发中3种UI跳变的处理"
date: 2015-05-19 19:02:08
categories: iOS
tags: [iOS, UI]
---

最近在使用豆瓣api仿写一个豆瓣同城的app，过程中遇到了3中界面的跳变，解决方式整理如下。
#1. UIPageViewController的内容跳变
这里现象如图所示。
![](/images/pvc-jump.gif)
造成这种跳变的原因主要在于，作为UIPageViewController内容的Controller，其View的边界被设置为了父View的margin。如图所示。
![](/images/relative-to-margin.png)
<!-- more -->
滚动完成后，margin消失，内容View宽度扩展到没有margin的宽度，导致界面跳动。修复方法如下图所示，只需要在内容Controller中将leading和trailing设置为父View的leading和trailing，不包含margin即可。
![](/images/without-margin.png)

修复后效果如图所示。
![](/images/pvc-fixed.gif)


#2.UITableViewController的Top Bars跳变
在实现一个简单的点击UITableViewController item展示详情的界面时，在top bar出出现了如下图的跳变，容易看出是由于被top bar覆盖区域从有内容，到没有内容导致其跳动。

![](/images/tableview-under-top-bars.gif)

这时UITableViewController关键属性设置如下：

![](/images/under-top-bars.png)

而此时详情ViewController并没有勾选Under Top Bars这个选项。这里的Under Top Bars表示视图的上边界是否被Top Bars遮挡，出现UI跳变是因为UITableViewController的View绘制了Top Bars的背景，而详情页面没有。而由于勾选了Ajust Scroll View Insets，其内容会适应边界，不被Top Bars遮挡，视觉上和没有勾选Under Top Bars一样，下图是只勾选Under Top Bars，而不选择Ajust Scroll View Insets的界面，可以清楚的看到UITableView实际上已经上移到了Top Bars下方。
![](/images/real-under-top-bars.png)

取消UITableViewController的这个选项后，跳变修复，效果如下图所示。

![](/images/tableview-under-top-bars-fixed.gif)
#3.Hide Bottom Bar on Push
在从精选页面进入详情页面时，希望隐藏底部的Tab Bar，因此勾选了详情Controller的Hide Bottom Bar on Push选项。此时，进入详情界面时，底部出现黑色从有到无的跳变，效果如下图。
![](/images/bottom-bar-jump.gif)
此时的详情View底部约束如下：
![](/images/bottom-top.png)
可以看出，详情View（这里是ScrollView）的底部是于Bottom Layout Guide的顶部对齐的，而在push过程中，由于隐藏了bottom bar，Bottom Layout Guide的顶部高度发生了变化（从bottom bar上方变为屏幕底部），因此详情View高度随之变化。

这里的修复方式是让详情View的底部始终与Bottom Layout Guide的底部对齐，如下图所示。
![](/images/bottom-bottom.png)
这样详情View的底部在任何时候都会绘制原来被bottom bar占有的区域，修复后效果如下图所示。
![](/images/bottom-bar-fixed.gif)

