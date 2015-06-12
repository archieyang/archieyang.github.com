title: "Philm项目源码分解析(2): View与Presenter的设计"
date: 2015-03-27 18:46:13
categories: Android
tags: [Android, MVP]
---

#0.前言
本文是Philm源码分析系列的第二篇，重点分析Philm MVP结构中View与Presenter的设计。为了便于分析，本文选取了Philm中最简单的部分About，这部分用于展示Philm的关于信息，不包含任何Model代码。

#1.绑定View和Presenter
在Philm中，View由Fragment和Activity实现，由于应用启动时首先启动Activity,因此，绑定Presenter的工作要在View中进行（也就是Activity/Fragment）。Philm在AboutFragment的onResume方法和onPause方法中进行View与Presenter的绑定与解绑。具体代码如下：
{% codeblock lang:java %}
public class AboutFragment extends ListFragment<ListView>
        implements AboutController.AboutListUi {
    ...
    @Override
    public void onResume() {
        super.onResume();
        getController().attachUi(this);
    }

    @Override
    public void onPause() {
        getController().detachUi(this);
        super.onPause();
    }
    ...
}


{% endcodeblock%}

对于Philm的About部分，它的Presenter就是MainController中的AboutController（是的，Chris Banes将Presenter命名为了Controller）。
<!-- more -->
#2.设置回调
下面来看Presenter(BaseUiController)中attachUi的实现：
{% codeblock lang:java %}
abstract class BaseUiController<U extends BaseUiController.Ui<UC>, UC>
        extends BaseController {

    public interface Ui<UC> {

        void setCallbacks(UC callbacks);

        boolean isModal();
    }
    ...
    public synchronized final void attachUi(U ui) {
        Preconditions.checkArgument(ui != null, "ui cannot be null");
        Preconditions.checkState(!mUis.contains(ui), "UI is already attached");

        mUis.add(ui);

        ui.setCallbacks(createUiCallbacks(ui));

        if (isInited()) {
            if (!ui.isModal() && !(ui instanceof SubUi)) {
                updateDisplayTitle(getUiTitle(ui));
            }

            onUiAttached(ui);
            populateUi(ui);
        }
    }
    ...
}
{% endcodeblock%}
这里最重要的两处代码是1）通过ui.setCallbacks(createUiCallbacks(ui))设置回调；和2）绘制Ui populateUi(ui)。

Philm在设置View回调是采用了一个非常精巧的设计：通过泛型来处理ViewInterface与回调。BaseUiController的泛型可以表示为BaseUiController<ViewInterface<ViewCallbacks>, ViewCallbacs>，其中前一个泛型代表了前文所述用于解耦View和Presenter而实现的ViewInterface，而后一个是提供给ViewInterface的回调。ViewInterface本身提供了setCallbacks方法来进行ViewCallbacks与View的绑定。这样，所有的业务逻辑都可以在Presenter中实现，之后通过回调提供给View。View仅仅负责UI的表现，在需要处理业务逻辑时，通过调用之前Presenter设置的回调实现。

这里可以看一下About中响应List中某个item被点击的实现。这里onListItemClick继承自ListFragment<ListView>，用于响应某个item的点击事件，AboutFragment将这个事件交给了Presenter设置的回调来处理。
{% codeblock lang:java %}
public class AboutFragment extends ListFragment<ListView>
        implements AboutController.AboutListUi {
    ...
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        AboutController.AboutItem item = (AboutController.AboutItem) l.getItemAtPosition(position);
        if (item != null && hasCallbacks()) {
            getCallbacks().onItemClick(item);
        }
    }
    ...
}

{% endcodeblock%}

#3.Display
这是Philm中一处比较新颖的设计，它本身应该算是Presenter的一部分，具体的实现在AndroidDisplay这个类中。它的作用是负责app中**所有**UI切换的工作，比如Activity跳转，现实Fragment，弹出Drawer等。听起来很不可以思议，但是由于这些代码都很短小，实际上这个类并不大，也不复杂。



