---
layout: post
title: CollapsingToolbarLayout 介绍和简单使用
tags:
- AppBarLayout
categories: Android CollapsingToolbarLayout
description: CollapsingToolbarLayout 介绍和简单使用
---

​	上次我们介绍了AppBarLayout,这一次我们介绍CollapsingToolbarLayout。	

## CollapsingToolbarLayout 介绍

​	顾名思义，这是一个可折叠的Toolbar；不过它的使用必须在AppBarLayout的基础之上，它必须作为AppBarLayout的直接子类元素使用；否则起不到应用的效果。

​	在Android Studio 里创建module/Activity时，就会提供一个`ScrollActivity`的模板，这个模板就使用了AppBarLayout 和 CollapsingToolbarLayout 实现了可折叠的Toolbar。这里就不在具体介绍如何去创建这个模板了。我们先看一下这个模板实现的大概效果：

![ScrollingActivity模板效果](http://7xrxe7.com1.z0.glb.clouddn.com/CollapsingToolbarLayout-1)

xml的设置如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:fitsSystemWindows="true"
        >

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            >

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:title="标题"
                />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"/>

</android.support.design.widget.CoordinatorLayout>	
```

首先我们看一下AppBarLayout ,它设置了app:layout_scrollFlags="scroll|exitUntilCollapsed"，关于'exitUntilCollapsed'，会涉及到minHeight；当发生向上滚动事件时，AppLayout向上滚动，直到我们设置的minHeight，然后我们的滑动View才开始滚动。就算我们滑动的view完全上滑完毕，我们的AppBarLayout也会一直保留我们设置的民Height显示在屏幕的上方。但是我们并没有去设置minHeight呀？从表面上看确实是这样的。我们去看一下CollapsingToolbarLayout的源码来一探究竟。在onLayout方法里我们可以看到这么一段代码：

```Java
@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);

    // ……省略若干代码
    // Finally, set our minimum height to enable proper AppBarLayout collapsing
    if (mToolbar != null) {
        if (mCollapsingTitleEnabled && TextUtils.isEmpty(mCollapsingTextHelper.getText())) {
            // If we do not currently have a title, try and grab it from the Toolbar
            mCollapsingTextHelper.setText(mToolbar.getTitle());
        }
        if (mToolbarDirectChild == null || mToolbarDirectChild == this) {
            setMinimumHeight(getHeightWithMargins(mToolbar));
            mToolbarDrawIndex = indexOfChild(mToolbar);
        } else {
            setMinimumHeight(getHeightWithMargins(mToolbarDirectChild));
            mToolbarDrawIndex = indexOfChild(mToolbarDirectChild);
        }
    } else {
        mToolbarDrawIndex = -1;
    }
    updateScrimVisibility();
}
```

在上面代码里我们可以看到它Toolbar的高度作为minHeight了，当我们上滑时，滑到只剩下Toolbar高度时，CollapsingToolbarLayout 就不会再折叠了，会一直保持在屏幕的顶端。

### app:layout_collapseMode

​	除此之外，我们还可以发下在Toolbar里还设置了`app:layout_collapseMode="pin"`，这里涉及到了折叠模式。我们依次了解一下.

1. none：这个是默认属性，Toolbar会正常显示，不会有折叠行为；
2. pin : Toolbar 折叠以后会放到屏幕顶端，固定不动；
3. parallax：意为视差，当我们上滑时会有视差效果，但是我们之前设置在Toolbar上的东西会滑出屏幕。

具体我们示例：

当我们把折叠模式设置为 `pin` 时：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:fitsSystemWindows="true"
        >

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorAccent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            >

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:navigationIcon="@mipmap/ic_launcher"
                />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"/>

</android.support.design.widget.CoordinatorLayout>
```

为了看Toolbar的效果，我在Toolbar上加了一个navigationIcon；效果如下：

![](http://7xrxe7.com1.z0.glb.clouddn.com/CollapsingToolbarLayout-2)

这里ToolBar的标题会有折叠效果，但是Toolbar会一直在屏幕的顶端不动。

接下来我们看 `parallax`:

上代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:fitsSystemWindows="true"
        >

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorAccent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            >

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="parallax"
                app:title="标题"
                app:navigationIcon="@mipmap/ic_launcher"
                />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"/>

</android.support.design.widget.CoordinatorLayout>
```

这里我们只改了一个`app:layout_collapseMode`，我们看一下效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/CollapsingToolbarLayout-3)

这里只有折叠的效果，但是Toolbar会滑出屏幕。至于还有 `none `，就不在介绍了，没有任何效果。

### app:contentScrim

​	不知道有有没有细心同学看到这个`app:contentScrim`属性。这个是用来设置折叠时Toolbar所在位置的颜色,并且在滑动的时候会有一定的过渡。

刚开始研究的时候，这个属性老是不起作用，后来发现，当CollapsingToolbarLayout里只有的一个Toolbar时，会使用Theme指定的值。**当我们在CollapsingToolbarLayout里添加其它的控件，并且这个控件必须是在Toolbar的上方时，这个属性才会起作用。** 接下来我们看一下这个属性的效果。

先上代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        >

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorAccent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            >
            <ImageView
                android:background="@drawable/header_bar"
                android:layout_width="match_parent"
                android:layout_height="260dp"/>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:title="标题"
                app:navigationIcon="@mipmap/ic_launcher"
                />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/text"/>
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"/>

</android.support.design.widget.CoordinatorLayout>
```

在Toolbar上面加了一个ImageView，然后在CollapsingToolbarLayout上设置了app:contentScrim="?attr/colorAccent",是一个粉红色。我们看效果：

![这里写图片描述](http://7xrxe7.com1.z0.glb.clouddn.com/CollapsingToolbarLayout-4)


关于CollapsingToolbarLayout的基本的几个点就差不多完了，我们需要注意的是：

1. app:layout_scrollFlags 要放到AppBarLayout的直接子控件上
2. app:contentScrim 要放到CollapsingToolbarLayout上
3. app:contentScrim 这个属性，需要我们在CollapsingToolbarLayout里添加其它的控件，并且这个控件必须是在Toolbar的上方时，这个属性才会起作用