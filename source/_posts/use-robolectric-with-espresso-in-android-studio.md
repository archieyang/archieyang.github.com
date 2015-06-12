title: "在Android Studio中使用Roboletric和Espresso"
date: 2015-05-27 18:38:33
categories: Android
tags: [Test, Robolectric, Espresso, Android Studio, Android]

---
#0. 前言
在Android开发中，测试是一个很少被提及的话题。在早期，Android并没有一个很好的测试框架，你也很难找到一个测试全面的优秀开源项目。近些年，随着Android社区的成熟，出现了诸如Robotium，Robolectric等的优秀测试框架。而Google也在近期推出了自己的UI测试框架Espresso。

本文基于最简单的Hello World项目进行测试，重点在于介绍Robolectric和Espresso的配置.。项目完整的示例代码在[这里](https://github.com/archieyang/EspressoAndRobolectricTestDemo)。
#1. 使用Espresso进行Instrumentaion Test
首先创建一个最简单的Android项目，包含一个Hello World的TextView。打开Build Variant，选择Android Instumentation Tests。在build.gradle中配置Espresso，增加的代码如下：
{% codeblock lang:groovy %}
apply plugin: 'com.android.application'

android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    
    packagingOptions {
        exclude 'LICENSE.txt'
    }
}

dependencies {
    ...
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.1'
    androidTestCompile 'com.android.support.test:runner:0.2'
}
{% endcodeblock %}
之后点击Gradle同步。

到src/androidTest删除自动生成的ApplicationTest.java，新建MainActivityTest.java如下。这段代码主要是测试Hello world!这段文字是否显示到了界面上。

{% codeblock lang:java %}
@LargeTest
public class MainActivityTest extends ActivityInstrumentationTestCase2<MainActivity> {

    public MainActivityTest() {
        super(MainActivity.class);
    }

    @Override
    public void setUp() throws Exception {
        super.setUp();
        getActivity();
    }

    public void testHelloWorldOnView() {
        onView(withText("Hello world!")).check(matches(isDisplayed()));
    }

}
{% endcodeblock %}
<!-- more -->
右击MainActivityTest.java，选择Run，如下图所示。
![](/images/run-espresso.png)
选择运行的设备之后可以到在下方控制台看到打印的测试成功信息如下：
{% codeblock lang:java %}
Testing started at 下午7:08 ...
Installing me.codethink.testdemo
DEVICE SHELL COMMAND: pm install -r "/data/local/tmp/me.codethink.testdemo"
pkg: /data/local/tmp/me.codethink.testdemo
Success


Uploading file
	local path: /Users/archie/AndroidStudioProjects/TestDemo/app/build/outputs/apk/app-debug-androidTest-unaligned.apk
	remote path: /data/local/tmp/me.codethink.testdemo.test
Installing me.codethink.testdemo.test
DEVICE SHELL COMMAND: pm install -r "/data/local/tmp/me.codethink.testdemo.test"
pkg: /data/local/tmp/me.codethink.testdemo.test
Success


Running tests
Test running startedFinish
{% endcodeblock%}
此时到Run -> Edit Configurations查看，可以看到增加了MainActivityTest，如下图所示。
![](/images/run-espresso-config.png)
至此完成了一个简单的Espresso测试。
#2. 使用Robolectric进行Test
首先复制androidTest文件夹到同一个目录下，重命名为test。打开Build Variant，选择Unit Tests。
![](/images/android-studio-build-variants.png)
修改build.gradle，增加Robolectric依赖如下。
{% codeblock lang:groovy %}
{
    ...
    dependencies {
        ...
        testCompile 'junit:junit:4.12'
        testCompile "org.assertj:assertj-core:1.7.0"
        testCompile('org.robolectric:robolectric:3.0-rc2') {
            exclude group: 'commons-logging', module: 'commons-logging'
            exclude group: 'org.apache.httpcomponents', module: 'httpclient'
        }
}
{% endcodeblock %}

删除test目录下的MainActivityTest.java，新建MainActivityUnitTest.java代码如下：
{% codeblock lang:java %}
@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class, emulateSdk = 21)
public class MainActivityUnitTest {
    private MainActivity activity;

    @Before
    public void setUp() throws Exception {
        activity = Robolectric.setupActivity(MainActivity.class);
    }

    @Test
    public void testHelloWorld() throws Exception {
        TextView helloWorldTextView = (TextView) activity.findViewById(R.id.textview_id);
        assertThat(helloWorldTextView.getText().toString()).isEqualTo("Hello World!");
    }

}
{% endcodeblock %}
右键点击test目录下的MainActivityUnitTest.java，选择Run。
![](/images/run-robolectric.png)
之后可以在控制台看到测试成功的信息。
此时到Run -> Edit Configurations查看，可以看到增加了MainActivityUnitTest，如下图所示。
![](/images/run-robolectric-config.png)
把assertThat(helloWorldTextView.getText().toString()).isEqualTo("Hello world!")中的world改为World，再次Run改测试，可以在控制台看到错误信息以及生成的错误报告的地址，错报告内容如下图所示：
![](/images/robolectric-test-report.png)


#3. 小结
Espresso作为Google推出的Instrumentation UI测试框架，在API支持方面有着天然的优势，在推出后很大程度上替代了Robotium。而Robolectric由于只在Java虚拟机中运行，速度很快，虽然在API支持上无法和Espresso相比，但速度有很大优势，适合单元测试，尤其是TDD时使用。