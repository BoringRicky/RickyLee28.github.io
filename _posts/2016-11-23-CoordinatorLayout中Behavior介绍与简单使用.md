---
layout: post
title: CoordinatorLayout中Behavior介绍与简单使用
tags:
- CoordinatorLayout
- Behavior
categories: Android CoordinatorLayout Behavior
description: CoordinatorLayout 中 Behavior 介绍与简单使用
---

​	我们之前简单介绍了AppBarLayout,CollapsingToolbarLayout的使用，他们的都是作为CoordinatorLayout的子View 使用的。如果没有CoordinatorLayout作为父View，它们是没有任何效果的。今天我们介绍一下CoordinatorLayout中的Behavior。在CoorindatorLayout中其实没有做太多的事情，它就是一个ViewGroup，类似FrameLayout；它的核心就在于Behavior类。我们给CoordinatorLayout的子View设置一个Behavior，就可以接管该View自身的一些事件与其他的View之间的交互事件。包括 touch,measure,layout等事件。

​	我们使用的AppBarLayout有属于自己的Behavior，它是在AppBarLayout中声明的内部类，有兴趣的去看看AppBarLayout的代码就很容易发现。

### 简单定义Behavior

​	我们没有办法直接使用Behavior，需要我们自己定义自己的Behavior。如何定义自己的内部类呢？只需要继承Behavior就OK了。就像下面这样：

```Java
public class MyBehavior extends CoordinatorLayout.Behavior<View> {
    //用来在代码里实例化Behavior的构造
    public MyBehavior() {
    }

    //如果要在布局中使用Behavior，这个构造是必须的
    public MyBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

​	这里有个注意地方，Behavior只能作用在View上的，所以泛型上用的是View 。我们都知道View中	用`setTag`,`getTag`用来保存临时数据，也有`onSaveInstanceState`,`onRestoreInstanceState`来保存相关实例的状态，Behavior也有，用法跟View相当类似，这里不介绍了，一般用不到。

### 使用自定义的Behavior

​	第一种方式：我们自定义的可以通过在布局中使用`app:layout_behavior`的来指定，例如，我们在NestedScrollView设定的Android提供的Behavior,`app:layout_behavior="@string/appbar_scrolling_view_behavior">`,其中的string是Behavior的绝对路径。

​	第二种方式：如果你要用到自定义View/ViewGroup中，可以通过`@CoordinatorLayout.DefaultBehavior()`注解指定Behavior；这样我们的View就会默认使用这个Behavior。比如在AppBarLayout的使用方式：

```Java
@CoordinatorLayout.DefaultBehavior(AppBarLayout.Behavior.class)
public class AppBarLayout extends LinearLayout {
	//省略具体实现代码
}
```

​	第三种方式：通过代码设定Behavior，前提是你的指定的View必须是CoordinatorLayout的子View，如下：

```Java
MyBehavior myBehavior = new MyBehavior();
//我们的View必须是CoordinatorLayout的子View，否则我们获取不到CoordinatorLayout.LayoutParams
CoordinatorLayout.LayoutParams params = (CoordinatorLayout.LayoutParams) 我们的View.getLayoutParams();
params.setBehavior(myBehavior);
```

### Behavior中的依赖

​	Behavior可以设置使用Behavior的View依赖的View。比如我们的View A 使用了我们自定义的Behavior MyBehavior,我们可以通过使用MyBehavior设定我们的 View A依赖于某个View B，这样我们可以获取到View B的各种状态，根据View B的状态去设定我们的View A 。

​	我们通过 覆写Behavior`layoutDependsOn`去指定依赖的View。例如像这样：

```Java
/**
 * 指定依赖的View,在这里指定依赖的View之后，
 * 我们去{@linkplain #onDependentViewChanged(CoordinatorLayout, View, View)}里去处理这两个依赖的View之间的关系
 *
 * @param parent
 * @param child      使用该Behavior的View
 * @param dependency 依赖的View
 * @return 当指定的View是我们需要的View时，返回true
 */
@Override
public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
    return dependency instanceof AppBarLayout;
}
```

也可以像这样指定某个View，通过id去验证是不是我们依赖的View :

```java
@Override
public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
    return dependency.getId() == R.id.dependency_view_id;
}
```

​	指定完依赖的View 之后，我们去覆写`onDependentViewChanged`方法去获取我们依赖的View的各个状态，并且设置我们自己的View状态，例如：

```Java
/**
 * 当我们指定依赖的View 有变化时，调用这个方法
 * @param parent
 * @param child 使用该Behavior的View
 * @param dependency 依赖的View
 * @return
 */
@Override
public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
    final View finalChild = child;
    AppBarLayout finalDependency = (AppBarLayout) dependency;
    finalDependency.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
        @Override
        public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
            float tranlationY = Math.abs(verticalOffset / 3);
            finalChild.setTranslationY(tranlationY);
        }
    });
    return true;
}
```

这里我利用了 [AppBarLayout CollapsingToolbarLayout 的进一步使用](http://blog.csdn.net/litengit/article/details/52958721)中的示例，如果兴趣的同学可以看一下。那么整个Behavior就是如下这样了：

```Java
public class MyBehavior extends CoordinatorLayout.Behavior<View> {

    //用来在代码里实例化我们自定义的Behavior的构造
    public MyBehavior() {
    }

    //如果要在布局中使用Behavior，这个构造是必须的
    public MyBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 指定依赖的View,在这里指定依赖的View之后，
     * 我们去{@linkplain #onDependentViewChanged(CoordinatorLayout, View, View)}里去处理这两个依赖的View之间的关系
     *
     * @param parent
     * @param child      使用该Behavior的View
     * @param dependency 依赖的View
     * @return 当指定的View是我们需要的View时，返回true
     */
    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
        return dependency instanceof AppBarLayout;
    }

    /**
     * 当我们指定依赖的View 有变化时，调用这个方法
     * @param parent
     * @param child 使用该Behavior的View
     * @param dependency 依赖的View
     * @return
     */
    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
        final View finalChild = child;
        AppBarLayout finalDependency = (AppBarLayout) dependency;
        finalDependency.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
            @Override
            public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
                float tranlationY = Math.abs(verticalOffset / 3);
                finalChild.setTranslationY(tranlationY);
            }
        });
        return true;
    }
}
```

我们设定了依赖的View 是AppBarLayout，我们监听AppBarLayout的上下折叠的偏移，根据它的偏移来设置我们的View的状态。效果如下：

![](http://7xrxe7.com1.z0.glb.clouddn.com/CoordinatorLayout%20Behavior-1)

我们可以看到我们的下面粉红色的View，会随着我们的AppBarLayout的折叠展开会滑出滑进屏幕。根据这个效果结合我们AppBarLayout的`app:layout_scrollFlags` 就可以做出类似知乎首页的那种功能了。

### 监听滑动

​	如果我们需要监听一下CoordinatorLayout中子View的滑动，而不使用依赖View，那么怎么办呢？

要处理这个问题我们至少需要了解两个方法`onStartNestedScroll`,`onNestedPreScroll`这两个方法：

```Java
/**
 * 这里返回true，才会接收到后续滑动事件
 *
 * @param coordinatorLayout
 * @param child             使用该Behavior的View
 * @param directTargetChild Coordinator的子View，它可能包含一些滑动的操作
 * @param target            初始化滑动动作的View
 * @param nestedScrollAxes  滑动的轴，根据这个值来判断是纵向滑动还是横向滑动;
 *                          {@linkplain ViewCompat#SCROLL_AXIS_VERTICAL} 竖向；
 *                          {@linkplain ViewCompat#SCROLL_AXIS_HORIZONTAL} 横向
 * @return
 */
@Override
public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
    return  super.onStartNestedScroll(coordinatorLayout,child,directTargetChild,target,nestedScrollAxes);
}

/**
 * 这里可以获取到CoordinatorLayout的滑动
 *
 * @param coordinatorLayout
 * @param child             使用该Behavior的View
 * @param target            执行滑动动作的View
 * @param dx                横向滑动的距离,以像素为单位
 * @param dy                纵向滑动的距离,以像素为单位
 * @param consumed
 */
@Override
public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
    super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
}
```

如果现在我们有一个Button想让它随着一个ScrollView 的滑动而滑动，怎么办呢？通过上面的两个方法就可以实现。我们自己定义一个`ScrollBehavior`,覆写上面的方法就OK了：

```java
public class ScrollBehavior extends CoordinatorLayout.Behavior<View>{

    public ScrollBehavior() {
    }

    public ScrollBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 这里返回true，才会接收到后续滑动事件
     *
     * @param coordinatorLayout
     * @param child             使用该Behavior的View
     * @param directTargetChild Coordinator的子View，它可能包含一些滑动的操作
     * @param target            初始化滑动动作的View
     * @param nestedScrollAxes  滑动的轴，根据这个值来判断是纵向滑动还是横向滑动;
     *                          {@linkplain ViewCompat#SCROLL_AXIS_VERTICAL} 竖向；
     *                          {@linkplain ViewCompat#SCROLL_AXIS_HORIZONTAL} 横向
     * @return
     */
    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL;
    }

    /**
     * 这里可以获取到CoordinatorLayout的滑动
     *
     * @param coordinatorLayout
     * @param child             使用该Behavior的View
     * @param target            执行滑动动作的View
     * @param dx                横向滑动的距离,以像素为单位
     * @param dy                纵向滑动的距离,以像素为单位
     * @param consumed
     */
    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
        final View finalChild = child;
        if (target instanceof NestedScrollView){
            NestedScrollView view = (NestedScrollView) target;
            view.setOnScrollChangeListener(new NestedScrollView.OnScrollChangeListener() {
                @Override
                public void onScrollChange(NestedScrollView v, int scrollX, int scrollY, int oldScrollX, int oldScrollY) {
                    finalChild.setTranslationY(scrollY/5);
                }
            });
        }
    }
}
```

这里我们在onNestedPreScroll 里对我们的ScrollView进行的了滑动监听，这样就避免去拦截它的`onNestedFling`了，`onNestedPreScroll`中的 target 表示的是CoordinatorLayout子View中执行滑动操作的View。

具体效果如下：

![](http://7xrxe7.com1.z0.glb.clouddn.com/CoordinatorLayout%20Behavior-2)

其实还有其它的几个方法还是比较重要的，比如：`onNestedFling`，`onNestedPreFling`，等等，有兴趣的朋友可以去研究一下。