---
layout: post
title: Android View 测量过程
tags:
- Android
categories: Android,View测量,measure,onMeasure
description: Android View 测量过程
---

###  为什么要测量

​	我们在xml里设置布局时，必须要设定它的宽和高，不设置的话就会报错。 这是因为我们必须要告诉系统我们的View/ViewGroup需要多大的区域去绘制它。当我们给它设置好宽高后，系统需要测量一下才能知道它的尺寸，从而确定需要多大的区域去绘制它。

​	在View类里，使用了`measure(int widthMeasureSpec, int heightMeasureSpec)`测量一个View有多大，具体的测量是在`onMeasure(int widthMeasureSpec, int heightMeasureSpec)`,但是 `measure(int widthMeasureSpec, int heightMeasureSpec)`是final的，我们无法重写，我们只能去重写`onMeasure(int widthMeasureSpec, int heightMeasureSpec)`。 

​	在测量时会用到一个类 `MeasureSpec`,它会一直贯穿始终；它是一个非常重要的类，我们首先了解一下这个类。

###  MeasureSpec

这个类名的字面翻译为测量细则或者测量规格；对于该类官方注释给出的介绍如下：

> MeasureSpec 封装了父布局对子View的布局要求，这个具体的要求父布局会传递给子View。每个MeasureSpec可以表示宽或者高。MeasureSpec由尺寸(size)和模式(mode)组成。

MeasureSpec 使用的是 一个 32 位的int 数据实现。它将size和mode打包成了一份int型数值；其中高两位表示测量的模式，低30位表示该测量模式下的大小。它使用了各种位运算来计算这些值；如果有兴趣可以去看一下MeasureSpec的源码。那么我们怎么获取到mode 和 size 呢？

```Java
//获取宽的测量模式
int widthMode = MeasureSpec.getMode(widthMeasureSpec);
//获取高的测量模式
int heightMode = MeasureSpec.getMode(heightMeasureSpec);
//获取宽测量出的大小
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
//获取高测量出的大小
int heightSize = MeasureSpec.getSize(heightMeasureSpec);
```

​	当然如果你获取到了测量规格和大小之后也可以自己生成一个新的测量规格：

```java
int newMeasureSpec=MeasureSpec.makeMeasureSpec(size, mode);
```

上面提到的测量规格一共分为三种：

1. **UNSPECIFIED**，不限定。子View想要多大，父布局不会对它有任何限制，当然它不会超过父View的尺寸它的值为 0 。这个模式我们一般不会用到。我们遇到时最好给它一个确切的值。
2. **EXACTLY**，精确的。它的父布局会决定它的大小。不管它想要多大，它都会被限制在指定的大小范围内。它的父布局会检查它的精确尺寸。一般我们指定了View的具体尺寸或者`match_parent`时才会用到它。如果子View的宽/高设置的是 `match_parent`，那么这个子View的尺寸就是确定的，父View的size就是它的size。
3. **AT_MOST**，至多。它的父布局检测不到它的精确的尺寸， 那父布局会指定这个子View可以获取到额定的尺寸，这里的额定大小一般指恰好能包裹它的内容。一般我们在给View设置`wrap_content`时使用。如果子View的宽/高设置的`wrap_content`，那么说明它的尺寸是由它的内容决定的；但是它的尺寸最多跟父View的尺寸一致，无法超出父View。这样我们是无法得到它的具体尺寸的。只有等父View测量完成后才会的到子View的确切尺寸（父View会调用child.measure(int childWidthMeasureSpec,int childHeightMeasureSpec)方法去测量子View的尺寸）。比如我们的TextView，它的父View会去先测量一下它的文字的内容的尺寸才会给它设置尺寸；它的尺寸是由父View 指定的，它不会超过父View的尺寸。如果它的父View只有50px，而TextView的内容则占了100px,那么多出来的文字则会被截取掉不会再显示。


上面提到的 widthMeasureSpec 和 heightMeasureSpec 都是父类传递过来的，我们只需要在测量时通过它们获取size和mode  就OK了；这与细节大家不用多关心，如果感兴趣可以去看一下ViewGroup的measureChildren和 measureChild方法。

### View 的测量

 说完 MeasureSpec 我们回到View的 onMeasure(int widthMeasureSpec, int heightMeasureSpec) 。它的代码很简单：

```Java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(
      getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
      getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec)
    );
}		
```

其中的 `setMeasuredDimension(int measuredWidth, int measuredHeight)`方法是用来保存测量的尺寸，如果我们需要重写onMeasure，那么一定要在这里调用这个方法；否者会抛出异常。它的两个参数就是测量出来的宽和高，当然这里我们可以在代码里写死，比如setMeasuredDimension(50，50)，当然这是不提倡的。

其中的 `getDefaultSize(int size, int measureSpec)`会根据MeasureSpec去计算默认的尺寸。它的代码如下：

```Java
//这里的size为建议的最小宽/高
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
	//获取测量的模式
    int specMode = MeasureSpec.getMode(measureSpec);
    //获取测量的尺寸
    int specSize = MeasureSpec.getSize(measureSpec);
    switch (specMode) {
    case MeasureSpec.UNSPECIFIED://如果在UNSPECIFIED模式下，返回最最小宽/高
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY://AT_MOST、EXACTLY模式返回测量后的宽/高
        result = specSize;
        break;
    }
    return result;
}
```

根据上面onMeasure方法的代码，可以知道getDefaultSize中传入的size是建议的最小宽/高；如果在UNSPECIFIED模式下，返回最最小宽/高，其它的模式就是返回测量后的宽/高。这里的最小宽/高 ，指的是我们在XML 布局中设置的 `android:minHeight/android:minWidth`,或者是View背景图的宽/高。当然我们也可重写getSuggestedMinimumWidth()/getSuggestedMinimumHeight()方法指定最小宽/高。

​	如果我们需要重写onMeasure 方法，那么上面的代码就是我们可以参考的模板代码。我们可以通过三种模式去分别设定不同的尺寸。

### ViewGroup的测量

我们上面提到 `measure(int widthMeasureSpec, int heightMeasureSpec)`方法就是在ViewGroup里调用的。当ViewGroup测量子View的尺寸是就会调用 `child.measure(childWidthMeasureSpec, childHeightMeasureSpec)`; 通过ViewGroup的源码一共找到两处调用这个方法的地方；一个是 `measureChild()`一个是`measureChildWithMargins()`这两个方法代码类似，只是`measureChildWithMargins()`方法测量的时候会将 我们设置的margin 值计算在内:

```java
/**
 * 测量一个子View，它测量时会将padding 计算在内
 * @param child 测量的View
 * @param parentWidthMeasureSpec 父类传递给该View的宽的测量规格的要求
 * @param parentHeightMeasureSpec 父类传递给该View的高的测量规格的要求
 */
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

/**
 * 测量一个子View，它测量时不经会将padding计算在内，也会将margin计算在内
 *
 * @param child 测量的子View
 * @param parentWidthMeasureSpec 父类传递给该View的宽的测量规格的要求
 * @param widthUsed 父View已使用的在横向上的额外空间，也可能是其他的子View使用横向上的额外空间
 * @param parentHeightMeasureSpec 父类传递给该View的高的测量规格的要求
 * @param heightUsed 父View已使用的在竖向上的额外空间，也可能是其他的子View使用的竖向上的额外空间
 */
protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
    //包含子View margin 数据的 类
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

这个两个方法会在不同的布局中使用，在 `FrameLayout`，`LinearLayout`等布局里就是用了measureChildWithMargin。它们在使用时会遍历所有的子View，只要不是gone的都会去测量一遍。它会先把MeasureSpec传递给子View, 然后根据子View的 LayoutParams 再计算它的最最大宽度和最大高度等需要的数值，如果还有子View它会重复走这个流程。具体代码可以参看FrameLayout的onMeasure方法。
