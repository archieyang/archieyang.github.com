title: "实现画卷式展开的TextView"
date: 2015-04-20 19:01:36
categories: Android
tags: [Android, TextView]
---

#0. 前言

上周朋友拿美团外卖客户端的店铺页面，问我顶部文字详情展开的实现方式，这个效果本身就是一个已经有maxline设置的TextView展开的实现，但是没有对控件的跳变做更多的处理，感觉体验不是很好。于是我先仿写一个类似效果，又在原来的基础上把跳变改成了动画，效果如图所示。
![](/images/animated-flip-desc.gif)
<!-- more -->
关于这个交互设计的一点思考：之前在看一些色彩搭配的书籍时，有个观点说实际UI设计中看起来搭配的舒服的颜色，都是在自然界中可以出现的色彩搭配。同理到这个跳变，实际上是在自然界中没有什么物体有这么快的速度，因此这个速度（跳变）给人的体验并不好。
   
#1. 实现
##1.1 布局实现
布局的核心部分是图片背景及被折叠的详情文字。其实现如下。

{% codeblock lang:xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              tools:context=".MainActivity">

    <ScrollView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <FrameLayout

                android:layout_width="match_parent"
                android:layout_height="wrap_content">
                <ImageView
                    android:id="@+id/bg"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:src="@drawable/text_bg"
                    android:scaleType="centerCrop"/>
                <LinearLayout
                    android:id="@+id/content"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:background="#cc333333"
                    android:orientation="vertical">
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="150dip"
                        />
                    <TextView
                        android:id="@+id/descriptions"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_gravity="center_horizontal"
                        android:paddingLeft="10dip"
                        android:paddingRight="10dip"
                        android:textColor="#ffffff"
                        android:paddingTop="@dimen/activity_vertical_margin"/>

                    <ImageView
                        android:id="@+id/show_detail_button"
                        android:layout_width="30dip"
                        android:layout_height="30dip"
                        android:layout_gravity="bottom|center_horizontal"
                        android:src="@drawable/arrow_down"/>
                </LinearLayout>

            </FrameLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="600dip"
                android:background="#333333"></LinearLayout>
        </LinearLayout>
    </ScrollView>
</LinearLayout>
{% endcodeblock%}

由于直接设置背景对于文字展开时，图片的效果可控性很差，这里选择了使用FrameLayout将这一部分分为前景和背景。前景为包含被折叠TextView的content，背景为一个scaleType为centerCrop的ImageView。

##1.2 展开效果
位于content中的TextView在onCreate是被设置了最大行数descriptions.setMaxLines(currentLines);
展开时，只需要将最大行数设为Integer.MAX_VALUE即可其核心代码如下：

{% codeblock lang:java %}
showDetailButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                currentLines = currentLines == LIMIT ? Integer.MAX_VALUE : LIMIT;
                descriptions.setMaxLines(currentLines);
                showDetailButton.setRotation(180 - showDetailButton.getRotation());

                contentView.requestLayout();


            }
        });
{% endcodeblock%}

这里的showDetailButton.setRotation(180 - showDetailButton.getRotation());将自身翻转，使箭头反向。

这时点击showDetailButton，文字会全部展开，但是背景不会随文字展开而展开。因此需要在content发生变化时，通知背景ImageView进行变化，在onCreate中增加对contentView　layout 的监听。这部分的核心代码如下：
{% codeblock lang:java %}
contentView.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
            @Override
            public void onLayoutChange(View view, int i, int i2, int i3, int i4, int i5, int i6, int i7, int i8) {
                bg.getLayoutParams().height = contentView.getHeight();
                bg.requestLayout();
            }
        });
{% endcodeblock%}

至此实现了美团外卖店铺详情的效果，完整代码在[这里](https://github.com/archieyang/FlipDesc/tree/95c2865e2e22e04646d14afb3abbe392583c41a2)，效果如下图所示。
![](/images/normal-flip-desc.gif)

美团的效果如下图所示。
![](/images/meituan-waimai.gif)



#2. 更好的实现
这一部分是我对原来效果的优化，主要包括button和详情展开两部分。
##2.1 Button旋转
在设置监听器时的代码
showDetailButton.setRotation(180 - showDetailButton.getRotation());比较简单粗暴，直接将箭头调转方向，这里我未showDetailButton增加了一个旋转动画，代码如下
{% codeblock lang:java %}
        showDetailButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                ...
                Animation animation = new RotateAnimation(
                        currentLines == LIMIT ? 0 : 180,
                        currentLines == LIMIT ? 180 : 360,
                        Animation.RELATIVE_TO_SELF,
                        0.5f,
                        Animation.RELATIVE_TO_SELF,
                        0.5f);

                animation.setDuration(300);
                animation.setFillAfter(true);
                showDetailButton.startAnimation(animation);
                ...


            }
        });
{% endcodeblock%}
这里根据设置之后的TextView状态对showDetailButton进行旋转。
##2.2 展开动画
实现展开效果的expand函数如下所示，这里通过让高度随时间变化实现逐渐展开的效果。此外需要注意的是这里为动画设置了监听器。为了避免文字被卷起的边缘压到，分别在展开后和收缩前对TextView的maxLine进行修改。
{% codeblock lang:java %}
    public void expand(final View v, final  float originHeight, final float targetHeight) {
        Animation a = new Animation(){
            @Override
            protected void applyTransformation(float interpolatedTime, Transformation t) {
                v.getLayoutParams().height = (int)(originHeight + (targetHeight - originHeight) * interpolatedTime);
                v.requestLayout();
            }

            @Override
            public boolean willChangeBounds() {
                return true;
            }
        };

        a.setDuration(600);
        a.setFillAfter(true);
        a.setAnimationListener(new Animation.AnimationListener() {
            private boolean maxLineDidSet = false;
            @Override
            public void onAnimationStart(Animation animation) {
                if(currentLines == Integer.MAX_VALUE) {
                    currentLines = LIMIT;
                    descriptions.setMaxLines(currentLines);
                    maxLineDidSet = true;
                }

            }

            @Override
            public void onAnimationEnd(Animation animation) {
                if(!maxLineDidSet && currentLines == LIMIT) {
                    currentLines = Integer.MAX_VALUE;
                    descriptions.setMaxLines(currentLines);
                }


            }

            @Override
            public void onAnimationRepeat(Animation animation) {

            }
        });
        v.startAnimation(a);
    }

{% endcodeblock%}
下一步需要拿到TextView在收缩时和展开时的高度。在contentView.addOnLayoutChangeListener中增加获取高度的代码（初始高度设置为-1）：
{% codeblock lang:java %}
        contentView.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
            @Override
            public void onLayoutChange(View view, int i, int i2, int i3, int i4, int i5, int i6, int i7, int i8) {
					...
                if(THUMBNAIL == -1) {
                    THUMBNAIL = contentView.getHeight();
                }

                if(FULLSIZE == -1) {
                    FULLSIZE = THUMBNAIL + getTextViewHeight(descriptions) - descriptions.getHeight();
                }
                ...
            }
        });
{% endcodeblock%}

布局刚展示时，contentView的高度就是收缩时的高度。收缩时的高度加上TextView展开和收缩时的高度差即为展开时的contentView高度。这里的核心函数getTextViewHeight用于获取文字全部展开TextView的高度，其核心代码如下。这部分代码把TextView高度的计算分为两部分，文字高度desired和边距padding。getLineTop(n)函数返回第ｎ行顶端的位置，当n为行数时，为最后一行底部的位置。这里写成layout.getLineTop(pTextView.getLineCount())- layout.getLineTop(0)更好理解一些。

{% codeblock lang:java %}
    private int getTextViewHeight(TextView pTextView) {
        Layout layout = pTextView.getLayout();
        int desired = layout.getLineTop(pTextView.getLineCount());
        int padding = pTextView.getCompoundPaddingTop() + pTextView.getCompoundPaddingBottom();
        return desired + padding;
    }
{% endcodeblock%}

最后，在设置showDetailButton监听器时，增加展开和收缩的代码
{% codeblock lang:java %}
        showDetailButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                if(currentLines == LIMIT) {
                    expand(contentView, THUMBNAIL, FULLSIZE);
                } else {
                    expand(contentView, FULLSIZE, THUMBNAIL);
                }
                ...
            }
        });
{% endcodeblock%}

至此，对这个展开效果的优化结束，完整的代码在[这里](https://github.com/archieyang/FlipDesc)。


