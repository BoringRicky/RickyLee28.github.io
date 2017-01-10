---
layout: post
title: AppBarLayout 介绍和简单实用
tags:
- AppBarLayout
categories: Android AppBarLayout
description: AppBarLayout 介绍和简单实用
---

​	### 关于Android Design Support Library

​	在Android 5.0 时出现了 Material Design 。瞬时感觉Android 更加牛B哄哄了，可是其它老版本的Android 怎么办呢？它们也行这么牛B哄哄，走到哪里都耀眼夺目。怎么办呢？Google很贴心的提供了Android Design Support Library，这样就可以支持 Android 2.1 以上的设备了。

​	Android Design Support Library 其实就是一个AAR包，里面包含了navigation drawer view, TextInputEditText,FloatingActionButton, Snackbar, TabLayout, CoordinatorLayout 等。

​	今天我们介绍的是AppBarLayout这个控件。

### AppBarLayout

​	AppBarLayout 继承自LinearLayout，子控件默认为竖直方向显示，可以用它实现Material Design 的Toolbar；它支持滑动手势；它的子控件可以通过在代码里调用`setScrollFlags(int)`或者在XML里`app:layout_scrollFlags`来设置它的滑动手势。当然实现这些的前提是它的根布局必须是 CoordinatorLayout。这里的滑动手势可以理解为：当某个可滚动View的滚动手势发生变化时，AppBarLayout内部的子View实现某种动作。

​	AppBarLayout的子控件不仅仅可以设置为Toolbar，也可以包含其他的View。

​	下面我们具体来看看它的使用方法，我们先看以下的布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            app:title="标题"
            app:layout_scrollFlags="scroll|enterAlways"/>
    </android.support.design.widget.AppBarLayout>

	<android.support.v4.widget.NestedScrollView
        android:id="@+id/nested_scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="@string/text"/>
    </<android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

可以看到在Toolbar上添加了一个`app:layout_scrollFlags`,并且把它的值设置为`scroll|enterAlways`  我们不说为什么我们先看一下实现的效果:

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout-1)

我们可以看到当我向上文字时，Toolbar移出屏幕，向下滑动Toolbar进入屏幕。这就是之前的说的滑动手势。

####scroll flags

下面我们就看一下scroll flag 都包含哪些：

1. scroll：设置这个flag之后 Toolbar会滚出屏幕外部，如果不设置这个Toolbar 会固定在顶部不动。

2. enterAlways：这个flag跟scroll一块使用时，向上滑动时ToolBar移出屏幕，我们向下滑动时Toolbar进入屏幕。上面的图给出的就是scroll|enterAlways这个两个flag的效果。

3. enterAlwaysCollapsed：enterAlways的附加值。这里涉及到Child View的高度和最小高度，向下滚动时，Child View先向下滚动最小高度值，然后Scrolling View开始滚动，到达边界时，Child View再向下滚动，直至显示完全。我们看一下效果:

   ![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout-2)


具体的设置如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
            app:title="标题"
            />
        <ImageView
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:minHeight="?android:attr/actionBarSize"
            android:background="@drawable/mt_head"
            app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
            />
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:id="@+id/nested_scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="@string/text"/>

    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

首先它会先去找AppBarLayout的直接子View是否设置了minHeight；如果设置了我们的View下拉时，首先AppBarLayout会下拉到我们设定的minHeight，当我们下拉的View完全下拉完毕了，才会把AppBarLayout剩下的拉下来。

4.exitUntilCollapsed：这个跟上面的enterAlwaysCollapsed相反；它也涉及到minHeight，当发生向上滚动事件时，AppLayout向上滚动，直到我们设置的minHeight，然后我们的滑动View才开始滚动。就算我们滑动的view完全上滑完毕，我们的AppBarLayout也会一直保留我们设置的民Height显示在屏幕的上方。这个效果，我们一般会跟CollapsingToolbarLayout一块使用。具体我们看一下代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="?attr/colorPrimary"
            android:minHeight="?android:attr/actionBarSize"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:title="标题"
            />
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:id="@+id/nested_scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

然后我们再看一下显示效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppbarLayout-3)

5.snap：与scroll一起使用时，我们只需要滑动一小段距离，AppBarLayout就会随着我们滑动的View向上滑出屏幕。AppBarLayout会跟随着我们滑动View的滑动，当AppBarLayout滑出屏幕的部分大于显示区域，我们松开手指，AppBarLayout就会自动滑出屏幕；当AppBarLayout滑出屏幕的部分小于显示区域，我们松开手指，AppBarLayout就会自动滑进屏幕。

代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|snap"
            app:title="标题"
            />
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:id="@+id/nested_scrollview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```
效果图如下：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout-4)

注意：我们在使用这些flag时，一定要加上scroll，否则不会有效果。我们只需要在xml里设置就OK了，Java代码不需要设置就可以达到上面的效果。