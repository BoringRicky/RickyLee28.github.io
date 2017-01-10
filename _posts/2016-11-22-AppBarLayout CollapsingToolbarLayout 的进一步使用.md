---
layout: post
title: AppBarLayout CollapsingToolbarLayout 的进一步使用
tags:
- AppBarLayout
- CollapsingToolbarLayout
categories: Android CollapsingToolbarLayout AppBarLayout
description: AppBarLayout CollapsingToolbarLayout 的进一步使用
---

​	最近有个项目，虽然暂时停了，但是有效果还是想做一下；一方面是自己好奇，另一方面又怕领导突然一拍脑门，又重新做起来。正好利用到之前说过的AppBarLaout,CollapsingToolbarLayout，所以趁着之前的热乎劲一块搞出来就完了。关于这两个控件的使用请看一下[AppBarLayout 介绍和简单实用](http://blog.csdn.net/litengit/article/details/52948574)和 [CollapsingToolbarLayout 介绍和简单使用](http://blog.csdn.net/litengit/article/details/52948618)

​	首先看一下要实现的大概效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout%20CollapsingToolbarLayout-1)

​	首先我们分析一下这个效果，应该使用什么控件。这里有折叠效果，肯定会有CollapsingToolbarLayout；而且ToolBar 跟 AppBarLayout 也肯定少不了，要不然使用CollaspingToolbarLayout 就如同鸡肋了。下面的滑动标签使用的是TabLayout,最下面的内容使用的是ViewPager+Fragment。

​	好。分析完了，布局基本就出来了：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="@color/colorAccent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed" >

            <ImageView
                android:id="@+id/imageView"
                android:layout_width="match_parent"
                android:layout_height="260dp"
                android:background="@drawable/header_bar"/>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:navigationIcon="@mipmap/ic_launcher"
                app:title="标题"/>
        </android.support.design.widget.CollapsingToolbarLayout>

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:background="@android:color/white"/>

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:id="@+id/nestedScrollView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <android.support.v4.view.ViewPager
            android:id="@+id/viewPager"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </android.support.v4.widget.NestedScrollView>

</android.support.design.widget.CoordinatorLayout>
```

​	尤其需要我们注意一下的是，我们的TabLayout没有被折叠起来，只是随着被滑动到顶部，所以，我们把它放到了CollapsingToolbarLayout的下面。

​	我们可以看到我们的布局顶到了状态栏，这是4.4之后才可以用的设置，需要我们注意一下。设置的属性为`android:fitsSystemWindows`,设置为true时，系统会为顶部的状态栏留出空间；当设置为false时，我们的布局就会占顶到顶部状态栏。还有我们需要设置一下`android:windowTranslucentStatus"`，这个设置在4.4和5.x之后有一些不同；4.4的设置之后状态栏是渐变的，而5.x之后就是半透明的。但是这个属性是在Android API19 之后出来的，所以我们需要在另创建两个文件夹`vaules-19`,`vaules-21`，里面创建一个style；因为我创建的style使用`values`下的一些东西，所以我们先看一下values下的style文件：

```xml
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
    <style name="AppTheme.NoActionBar">
        <item name="android:windowActionBar">false</item>
        <item name="android:windowNoTitle">true</item>
    </style>
</resources>
```

然后我们去看一下`vaules-19`,`vaules-21`下的style文件，它们两个是一样的：

```xml
<resources>
    <style name="AppTheme.NoActionBar">
        <item name="android:fitsSystemWindows">false</item>
        <item name="android:windowTranslucentStatus">true</item>
    </style>
</resources>
```

接下来我们看一下代码：

```Java
public class MainActivity extends AppCompatActivity {
    private NestedScrollView mNestedScrollView;
    private TabLayout mTabLayout;
    private ViewPager mViewPager;

    private String[] mTitles = {"西游记", "西游记", "西游记"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mViewPager = (ViewPager) findViewById(R.id.viewPager);
        mTabLayout = (TabLayout) findViewById(R.id.tabs);
        mNestedScrollView = (NestedScrollView) findViewById(R.id.nestedScrollView);
        //设置 NestedScrollView 的内容是否拉伸填充整个视图，
        //这个设置是必须的，否者我们在里面设置的ViewPager会不可见
        mNestedScrollView.setFillViewport(true);

        mTabLayout.setupWithViewPager(mViewPager);
        MyAdapter adapter = new MyAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(adapter);
    }

    private class MyAdapter extends FragmentPagerAdapter {
        public MyAdapter(FragmentManager fm) {
            super(fm);
        }
        @Override
        public Fragment getItem(int position) {
            return new MyFragment();
        }
        @Override
        public int getCount() {
            return 3;
        }
        @Override
        public CharSequence getPageTitle(int position) {
            return mTitles[position];
        }
    }

    public static class MyFragment extends Fragment {
        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            String string = getString(R.string.text);
            List<String> strings = new ArrayList<>();
            for (int i = 0; i < 3; i++) {
                strings.add(string);
            }
            RecyclerView recyclerView = new RecyclerView(getActivity());
            recyclerView.setLayoutManager(new LinearLayoutManager(getActivity(), LinearLayoutManager.VERTICAL, false));
            recyclerView.setAdapter(new MyListAdapter(strings));
            return recyclerView;
        }


        class MyListAdapter extends RecyclerView.Adapter {
            private List<String> mStrings;
            public MyListAdapter(List<String> strings) {
                this.mStrings = strings;
            }
            @Override
            public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
                View convertView = LayoutInflater.from(getActivity()).inflate(R.layout.item, parent, false);
                return new MyListViewHolder(convertView);
            }
            @Override
            public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
                MyListViewHolder viewHolder = (MyListViewHolder) holder;
                viewHolder.mTextView.setText(mStrings.get(position));
            }
            @Override
            public int getItemCount() {
                return 3;
            }
        }

        class MyListViewHolder extends RecyclerView.ViewHolder {
            private TextView mTextView;
            public MyListViewHolder(View itemView) {
                super(itemView);
                mTextView = (TextView) itemView.findViewById(R.id.tv_content);
            }
        }
    }
}
```

​	这里需要强调一下，Fragment里的列表必须使用RecyclerView，而不能使用ListView，否者列表不能滑动，具体原因有时间我们在探讨。

​	还有我们的 NestedScrollView 调用了一个setFillViewport这个方法，这个方法会将NestedScrollView里填充内容的高拉伸，用来填充整个Viewport。如果不设置，那我们的Veiwpager则不会显示。其它的代码没有什么难度这里就不在说明了。

​	现在我们运行一下这些代码，会看到如下效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout%20CollapsingToolbarLayout-2)

我们可以看到，顶部的Toolbar的顶到了状态栏，这样不太好，我们在Toolbar上给它设置一个`android:layout_marginTop`，这个值设置为`25dp`(状态栏大概就25dp)，为了适配各个API Level ，我们分别在`vaules-19`,`vaules-21`里创建一个`dimens`文件，然后配置一下：

```xml
<dimen name="dimen_status_height">25dp</dimen>
```

我们在布局引用一下就OK了。

然后我们再看一下效果：

![](http://img.blog.csdn.net/20161028172221087)

上图是在API 22上的效果，下面的是在API 19 上的效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout%20CollapsingToolbarLayout-3)



可能大家以为到这里就结束了，当然不会的。有没有人看到标题栏里的`标题`那两个字随着上滑就感觉特别讨厌呢？是的，有的，我们的产品大人就很讨厌，所以她把标题放到的最上面变成字体颜色渐变。她要求当折叠结束时文字显示，下滑后文字逐渐消失。怎么办呢？我们慢慢来实现。

​	首先我们需要修改Toolbar里文字的透明度，所以我们需要修改一下布局，在Toolbar里放一下TextView：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="@color/colorAccent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/imageView"
                android:layout_width="match_parent"
                android:layout_height="260dp"
                android:background="@drawable/header_bar"/>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:layout_marginTop="@dimen/dimen_status_height"
                app:layout_collapseMode="pin"
                app:navigationIcon="@mipmap/ic_launcher">
                <TextView
                    android:id="@+id/tv_title"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="标题"/>
            </android.support.v7.widget.Toolbar>
        </android.support.design.widget.CollapsingToolbarLayout>

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:background="@android:color/white"/>
    </android.support.design.widget.AppBarLayout>
  
    <android.support.v4.widget.NestedScrollView
        android:id="@+id/nestedScrollView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" >
        <android.support.v4.view.ViewPager
            android:id="@+id/viewPager"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

​	相较于之前的布局我们只是在Toolbar里添加了一个TextView。不是特别的麻烦。

​	接下来我们需要去代码动态设置标题文字的透明度，我们知道当我们上滑时的折叠效果是通过CollapsingToolbarLayout 实现的。那我们怎么监听它的折叠呢？我去看了看它的源码，发现它还是使用的AppBarLayout 的 `OffsetUpdateListener`。

​	首先看它如何绑定的监听：

```Java
@Override
protected void onAttachedToWindow() {
    super.onAttachedToWindow();、
    // Add an OnOffsetChangedListener if possible
    final ViewParent parent = getParent();
    if (parent instanceof AppBarLayout) {
        // Copy over from the ABL whether we should fit system windows
        ViewCompat.setFitsSystemWindows(this, ViewCompat.getFitsSystemWindows((View) parent));
		
        if (mOnOffsetChangedListener == null) {
            mOnOffsetChangedListener = new OffsetUpdateListener();
        }
      //看这里，它还是使用的AppBarLayout的 mOnOffsetChangedListener
        ((AppBarLayout) parent).addOnOffsetChangedListener(mOnOffsetChangedListener);

        // We're attached, so lets request an inset dispatch
        ViewCompat.requestApplyInsets(this);
    }
}
```

其它暂时不再深究了，既然我们知道了它也是使用的AppBarLayout的OffsetChangedListener，那我们也使用它。首先看看这个监听：

```Java
mAppBarLayout.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
    @Override
    public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
    }
});
```

我们看一下这个回调方法理由有一个 `verticalOffset`，这个就是我们竖直方向上AppBarLayout偏移量，也可以理解为AppBarLayout移动距离的变化。知道这些我们只要根据这个值，动态的去计算一下 透明度就OK了。我们去代码去计算一下：

```Java
mAppBarLayout.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
    @Override
    public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
        int scrollRangle = appBarLayout.getTotalScrollRange();
        //初始verticalOffset为0，不能参与计算。
        if (verticalOffset == 0) {
            mTvTitle.setAlpha(0.0f);
        } else {
            //保留一位小数
            float alpha = Math.abs(Math.round(1.0f * verticalOffset / scrollRangle) * 10) / 10;
            mTvTitle.setAlpha(alpha);
        }
    }
});
```

​	需要注意的是，在未滑动时verticalOffset为0，在计算时一定要注意。

然后我们看一下效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/AppBarLayout%20CollapsingToolbarLayout-4)

现在基本效果实现了，可能还有一些瑕疵，希望大家给多提建议。

最后[源码](https://github.com/TengLeeIT/TopBarTest)奉上!!!