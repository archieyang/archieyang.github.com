---
layout: post
title: "DroidWorks: 在ActionBar中嵌入TabBar"
date: 2014-09-14 19:02:09 +0800
comments: true
categories: Android
tags: [Android, TabBar]
---
#前言
Android中默认的TabBar位于ActionBar的下方，在ActionBar横屏有足够空间时，会将TabBar嵌入到ActionBar中，节省纵向空间。一些UI设计强制将TabBar嵌入到ActionBar，并去掉默认的HOME LOGO，起到节省屏幕空间的作用，如网易云音乐2.1.1中。

![](/images/netease-cloud-music.png)
<!-- more -->

#1. ViewPager+Tabs实现侧滑
##实现ViewPager
##关联ViewPager与Tabs
这部分代码在Google官方的Training中有详细的阐述（[Creating Swipe Views with Tabs](http://developer.android.com/training/implementing-navigation/lateral.html)），这里就不详细解释了，它实现的这个侧滑和Tab联动的交互是上文所述效果的基础。
{% codeblock lang:java %}
package me.codethink.embeddedtabsactionbar;

import android.app.ActionBar;
import android.app.Activity;
import android.app.Fragment;
import android.app.FragmentManager;
import android.graphics.Color;
import android.os.Bundle;
import android.support.v13.app.FragmentStatePagerAdapter;
import android.support.v4.view.ViewPager;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;

import java.lang.reflect.Method;

public class MainActivity extends Activity {
    private TabContentPagerAdapter mTabContentPagerAdapter;
    private ViewPager mViewPager;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        final ActionBar actionBar = getActionBar();
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        mTabContentPagerAdapter = new TabContentPagerAdapter(getFragmentManager());
        mViewPager = (ViewPager) findViewById(R.id.pager);
        mViewPager.setAdapter(mTabContentPagerAdapter);

        mViewPager.setOnPageChangeListener(new ViewPager.SimpleOnPageChangeListener(){
            @Override
            public void onPageSelected(int position) {
                actionBar.setSelectedNavigationItem(position);
            }
        });

        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);

        ActionBar.TabListener tabListener = new ActionBar.TabListener() {

            @Override
            public void onTabSelected(ActionBar.Tab tab, android.app.FragmentTransaction fragmentTransaction) {
                mViewPager.setCurrentItem(tab.getPosition());
            }

            @Override
            public void onTabUnselected(ActionBar.Tab tab, android.app.FragmentTransaction fragmentTransaction) {

            }

            @Override
            public void onTabReselected(ActionBar.Tab tab, android.app.FragmentTransaction fragmentTransaction) {

            }
        };

        for (int i = 0; i < 3; i++) {
            actionBar.addTab(
                    actionBar.newTab()
                            .setText("Tab " + (i + 1))
                            .setTabListener(tabListener));
        }

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    public static class TabContentPagerAdapter extends FragmentStatePagerAdapter {

        public TabContentPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int i) {
            Fragment fragment = new DemoObjectFragment();
            Bundle args = new Bundle();
            switch (i) {
                case 0:
                    args.putInt(DemoObjectFragment.ARG_OBJECT, Color.RED);
                    break;
                case 1:
                    args.putInt(DemoObjectFragment.ARG_OBJECT, Color.YELLOW);
                    break;
                case 2:
                    args.putInt(DemoObjectFragment.ARG_OBJECT, Color.BLUE);
                    break;
            }

            fragment.setArguments(args);
            return fragment;

        }

        @Override
        public int getCount() {
            return 3;
        }

        @Override
        public CharSequence getPageTitle(int position) {
            return "OBJECT " + (position + 1);
        }

    }

    public static class DemoObjectFragment extends Fragment {
        public static final String ARG_OBJECT = "object";

        @Override
        public View onCreateView(LayoutInflater inflater,
                                 ViewGroup container, Bundle savedInstanceState) {

            View rootView = inflater.inflate(
                    R.layout.fragment_main, container, false);
            Bundle args = getArguments();
            rootView.setBackgroundColor(args.getInt(ARG_OBJECT));
            return rootView;
        }
    }

}


{% endcodeblock%}

效果如下：

![](/images/viewpager-and-tabs.png)
#2. 嵌入TabBar到ActionBar
通过反射机制强制嵌入TabBar是实现这个效果的核心部分，其代码如下：

{% codeblock lang:java %}
    @Override
    protected void onCreate(Bundle savedInstanceState) {
		......
		
        for (int i = 0; i < 3; i++) {
            actionBar.addTab(
                    actionBar.newTab()
                            .setText("Tab " + (i + 1))
                            .setTabListener(tabListener));
        }

        enableEmbeddedTabs(actionBar);
    }

    private void enableEmbeddedTabs(Object actionBar) {
        try {
            Method setHasEmbeddedTabsMethod = actionBar.getClass().getDeclaredMethod("setHasEmbeddedTabs", boolean.class);
            setHasEmbeddedTabsMethod.setAccessible(true);
            setHasEmbeddedTabsMethod.invoke(actionBar, true);
        } catch (Exception e) {
            Log.v("EmbeddedTabs", e.getMessage().toString());

        }
    }
{% endcodeblock %}
效果如下：

![](/images/raw-embedded-tabs.png)

#3. 修改ActionBar样式
##3.1 删除左侧默认的App Logo
通过设置定制的ActionBar style中的displayOptions为none可以去掉左侧默认的Logo
{% codeblock lang:xml %}
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="android:Theme.Holo.Light">
        <!-- Customize your theme here. -->
        <item name="android:actionBarStyle">@style/EmbeddedActionBar</item>
    </style>

    <style name="EmbeddedActionBar" parent="@android:style/Widget.Holo.Light.ActionBar.Solid">
        <item name="android:displayOptions">none</item>
    </style>


</resources>
{% endcodeblock %}

##3.2 修改TabBar样式
这一部分主要包括三个步骤

1.修改Tab前景

2.修改Tab背景

3.去掉TabBar的divider

###3.2.1 修改Tab前景
Tab前景主要是icon被选中时候颜色的改变，这里icon在选中的时候为绿色，未选中为白色，通过selector建立对应的Tab前景样式如下（这里以tag这个标签为例）：
`actionbar_tag.xml`
{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>

<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Selected states -->
    <item android:drawable="@drawable/actionbar_tag_green" android:state_focused="false" android:state_pressed="false" android:state_selected="true"/>
    <!-- Focused states -->
    <item android:drawable="@drawable/actionbar_tag_green" android:state_focused="true" android:state_pressed="false" android:state_selected="false"/>
    <item android:drawable="@drawable/actionbar_tag_green" android:state_focused="true" android:state_pressed="false" android:state_selected="true"/>
    <!-- Pressed -->
    <item android:drawable="@drawable/actionbar_tag_green" android:state_pressed="true"/>

    <item android:drawable="@drawable/actionbar_tag_white" android:state_focused="false" android:state_pressed="false" android:state_selected="false"/>
</selector>
{% endcodeblock %}

将三个前景图作为数组写到arrays.xml
`arrays.xml`
{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="actionbar_icons">
        <item>@drawable/actionbar_home</item>
        <item>@drawable/actionbar_tag</item>
        <item>@drawable/actionbar_account</item>
    </string-array>
</resources>
{% endcodeblock %}

在onCreate函数中设置Tab的前景：
`MainActivity.java`
{% codeblock lang:java %}
......

        TypedArray iconIds = getResources().obtainTypedArray(R.array.actionbar_icons);
        for (int i = 0; i < 3; i++) {
            View view = getLayoutInflater().inflate(R.layout.actionbar_tab_layout, null);
            ImageView imageView = (ImageView) view.findViewById(R.id.icon);
            imageView.setImageResource(iconIds.getResourceId(i, -1));

            actionBar.addTab(
                    actionBar.newTab()
                            .setCustomView(view)
                            .setTabListener(tabListener));
        }

        enableEmbeddedTabs(actionBar);
......
{% endcodeblock %}
这里的action_tab_layout如下
`action_tab_layout.xml`
{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
        >
    <ImageView android:id="@+id/icon"
               android:layout_gravity="center"
               android:layout_width="50dp"
               android:layout_height="50dp"/>

</LinearLayout>
{% endcodeblock %}

在styles.xml中设置Tab的padding和gravity
`styles.xml`
{% codeblock lang:xml %}
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="android:Theme.Holo.Light">
        <!-- Customize your theme here. -->
        <item name="android:actionBarStyle">@style/EmbeddedActionBar</item>
        <item name="android:actionBarTabStyle">@style/EmbeddedTabStyle</item>
    </style>

    <style name="EmbeddedActionBar" parent="@android:style/Widget.Holo.Light.ActionBar.Solid">
        <item name="android:displayOptions">none</item>
    </style>

    <style name="EmbeddedTabStyle" parent="@android:style/Widget.Holo.Light.ActionBar.TabView">
        <item name="android:padding">0dp</item>
        <item name="android:gravity">center</item>
    </style>
</resources>

{% endcodeblock %}
修改后的样式如下：

![](/images/front-tab-styled.png)
###3.2.2 设置Tab背景
Tab的背景是指Tab选中时，背景色随之改变的样式，同样通过selector实现，实现xml如下：

`actionbar_tabs_bg.xml`
{% codeblock lang:xml %}
<?xml version="1.0" encoding="utf-8"?>

<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Selected states -->
    <item android:drawable="@color/actionbar_selected" android:state_focused="false" android:state_pressed="false" android:state_selected="true"/>
    <!-- Focused states -->
    <item android:drawable="@color/actionbar_selected" android:state_focused="true" android:state_pressed="false" android:state_selected="false"/>
    <item android:drawable="@color/actionbar_selected" android:state_focused="true" android:state_pressed="false" android:state_selected="true"/>
    <!-- Pressed -->
    <item android:drawable="@color/actionbar_selected" android:state_pressed="true"/>

    <item android:drawable="@android:color/transparent" android:state_focused="false" android:state_pressed="false" android:state_selected="false"/>

</selector>

{% endcodeblock %}
在Tab的样式中指定background为actionbar_tags_bg.xml
{% codeblock lang:xml %}
<resources>
......
    <style name="EmbeddedTabStyle" parent="@android:style/Widget.Holo.Light.ActionBar.TabView">
        <item name="android:background">@drawable/actionbar_tabs_bg</item>
        <item name="android:padding">0dp</item>
        <item name="android:gravity">center</item>
    </style>

......
</resources>

{% endcodeblock %}



###3.2.3删除TabBar的divider
{% codeblock lang:xml %}
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="android:Theme.Holo.Light">
        <!-- Customize your theme here. -->
        <item name="android:actionBarStyle">@style/EmbeddedActionBar</item>
        <item name="android:actionBarTabStyle">@style/EmbeddedTabStyle</item>
        <item name="android:actionBarTabBarStyle">@style/customActionBarTabBarStyle</item>
    </style>
......
    <style name="customActionBarTabBarStyle" parent="@android:style/Widget.Holo.Light.ActionBar.TabBar">
        <item name="android:divider">@null</item>
    </style>
</resources>

{% endcodeblock %}

###3.2.4 修改ActionBar颜色

{% codeblock lang:xml %}
......
    <style name="EmbeddedActionBar" parent="@android:style/Widget.Holo.Light.ActionBar.Solid">
        <item name="android:displayOptions">none</item>
        <item name="android:background">@color/actionbar_dark_grey</item>
    </style>
...... 
{% endcodeblock %}

最终完成后效果如下：

![](/images/styled-embedded-tabs.gif)

代码地址：
[EmbeddedTabsActionBar](https://github.com/archieyang/EmbeddedTabsActionBar)






















