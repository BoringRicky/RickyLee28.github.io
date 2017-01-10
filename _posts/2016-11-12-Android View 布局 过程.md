​---
layout: post
title: Android View 布局过程
tags:
- View
categories: Android View 
description: Android View 布局过程
---	

当Android 的View测量完毕之后，才可以去布局。我们在测量时获取到的`MeasuredHeight`和`MeasuredWidth`,在布局的时候就会用到，毕竟只有知道尺寸才好确定位置。

布局相对于测量来说就相对简单了很多。布局的过程就是确定View在界面显示位置的过程。View是一个矩形，决定一个View在界面显示的位置的信息主要是四个边到父View的边距，这里会有四个变量来记录各个边到父View的距离：

- mLeft:View左边缘距离父View左部边缘的距离
- mTop:View上边缘距离父View上部边缘的距离
- mBottom:View底边缘距离父View上部边缘的距离
- mRight:View右边缘距离父View左部边缘的距离

>  注意:这里的距离单位是像素;Android 屏幕的像素原点在屏幕的左上角，所以View的宽高计算如下:
>
> View的宽=mRight-mLeft
> View的高=mBottom-mTop

这里的宽高和测量来的MeasuredWidth和MeasuredHeight是不同的，不要混淆.看它们的源码：

测量得到的高

```Java
public final int getMeasuredHeight() {
    return mMeasuredHeight & MEASURED_SIZE_MASK;
}
```

View实际的高

```Java
public final int getHeight() {
    return mBottom - mTop;
}
```

width和MeasuredWidth代码跟上面的代码类似就不在贴了。

布局是绘制的前提，只有布局完毕才可以去绘制。View布局时调用的方法是`layout(int l, int t, int r, int b)`它的是个参数就是代表View四个边距离父View的距离。虽然View了layout 方法，但是不提倡去重写它，如果有需要我们应该去重写 `onLayout(boolean changed, int left, int top, int right, int bottom)`,我们先看一下layout方法：

```Java
public void layout(int l, int t, int r, int b) {
  	//判断onMeasure是否应该跳过,(变量mPrivateFlags3存着一些关于Layout的信息，应该是用来判断View是否测量过，具体的没研究过)
    if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
        onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    }
    int oldL = mLeft;
    int oldT = mTop;
    int oldB = mBottom;
    int oldR = mRight;
  	//根据布局模式来判断是调用setOpticalFrame还是setFrame；
  	//setOpticalFrame内部也会再次调用setFrame,所以不论isLayoutModeOptical是false还是true，都会调用setFrame
	//setFrame用来分配View的位置和大小，如果新传入的位置和尺寸与之前的位置和尺寸有差别就会返回true。
	//所以这里获取的是View的位置或者尺寸是否发生变化
    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
      	//如果View的位置或者尺寸发生变化或者满足(mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED,就会首先调用onLayout
      	//onLayout是空的，没有任何实现，需要我们自己去实现。
      	//凡是继承ViewGroup的类一般都要重写这方法，来确定其中的子view的位置和尺寸
        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;
		//这里设置 OnLayoutChangeListener 监听事件
      	//ListenerInfo里面专门有个ArrayList来存储OnLayoutChangeListener
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLayoutChangeListeners != null) {
          	//对ArrayList中的OnLayoutChangeListener中的事件监听器进行克隆
            ArrayList<OnLayoutChangeListener> listenersCopy =
                   (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
              //遍历添加进来的事件监听器，依次调用onLayoutChange方法；这样layout就能在收到布局发生变化时得到响应了
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }
    mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
    mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
```

`onLayout(boolean changed, int left, int top, int right, int bottom)`方法很简单，首先会判断一下是需要测量一下，如果不需要，记录一下传入的四个边到父View的边距;然后判断View的尺寸和位置是否变化过，如果变化去就调用onLayout方法，并且遍历OnLayoutChangedListener监听器。

`onLayout(boolean changed, int left, int top, int right, int bottom)`这个方法是空的，是需要子类去重写的。它的主要作用就是确定该View中子view的位置。通常会遍历该View中所有的子view，然后依次调用子view的layout方法，来确定子View的位置和尺寸。我们看一下FrameLayout中部分代码：

```Java
@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    layoutChildren(left, top, right, bottom, false /* no force left gravity */);
}

void layoutChildren(int left, int top, int right, int bottom, boolean forceLeftGravity) {
  	//获取子View的个数
    final int count = getChildCount();
	//获取父View的四边
    final int parentLeft = getPaddingLeftWithForeground();
    final int parentRight = right - left - getPaddingRightWithForeground();
    final int parentTop = getPaddingTopWithForeground();
    final int parentBottom = bottom - top - getPaddingBottomWithForeground();
	//遍历所有的子view
    for (int i = 0; i < count; i++) {
        final View child = getChildAt(i);
      	//只有在子view可见并且不占据位置的时候才去设置子view的尺寸和位置
        if (child.getVisibility() != GONE) {
			//……省略子View在各个Gravity情况下四边距离屏幕边距的数值……
            child.layout(childLeft, childTop, childLeft + width, childTop + height);
        }
    }
}
```

所以我们在继承ViewGroup时，重写onLayout方法也可以做类似的处理。

​	在上面我们提到布局变化监听，其实在View里提供了一个`onSizeChanged(newWidth, newHeight, oldWidth, oldHeight)`；当View的尺寸发生变化时就会调用这个方法。不过它刚加入的View树中时，它使用还之前的尺寸，都是0。

​	在源码里，它是在 ` setFrame(int left, int top, int right, int bottom)`中调用的。上面提到了setFrame是用来分配View的位置和大小，如果新传入的位置和尺寸与之前的位置和尺寸有差别就会返回true。简单看一下setFrame调用onSizeChanged的过程：

```Java
protected boolean setFrame(int left, int top, int right, int bottom) {
    boolean changed = false;
	//……省略log
	//新传入边距跟之前记录的边距不同，就说明View的尺寸或者位置发生变化
    if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
        changed = true;
      
        int drawn = mPrivateFlags & PFLAG_DRAWN;
        int oldWidth = mRight - mLeft;
        int oldHeight = mBottom - mTop;
        int newWidth = right - left;
        int newHeight = bottom - top;
      	//根据新边距和旧边距，判断尺寸是否发生变化
        boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);
		//重新绘图
        invalidate(sizeChanged);
		//将新边距赋值
        mLeft = left;
        mTop = top;
        mRight = right;
        mBottom = bottom;
        mRenderNode.setLeftTopRightBottom(mLeft, mTop, mRight, mBottom);
        mPrivateFlags |= PFLAG_HAS_BOUNDS;
      	//如果尺寸发生变化就会调用sizeChange，onSizeChanged就是在sizeChange方法里调用的
        if (sizeChanged) {
            sizeChange(newWidth, newHeight, oldWidth, oldHeight);
        }
        if ((mViewFlags & VISIBILITY_MASK) == VISIBLE || mGhostView != null) {
            mPrivateFlags |= PFLAG_DRAWN;
            invalidate(sizeChanged);
            invalidateParentCaches();
        }
        mPrivateFlags |= drawn;
        mBackgroundSizeChanged = true;
        if (mForegroundInfo != null) {
            mForegroundInfo.mBoundsChanged = true;
        }
        notifySubtreeAccessibilityStateChangedIfNeeded();
    }
    return changed;
}

private void sizeChange(int newWidth, int newHeight, int oldWidth, int oldHeight) {
 	//调用onSizeChanged方法，这个方法是空的；如果有操作需要在尺寸发生变化时，可以重写这个方法
	onSizeChanged(newWidth, newHeight, oldWidth, oldHeight);
	if (mOverlay != null) {
  	  mOverlay.getOverlayView().setRight(newWidth);
   	 mOverlay.getOverlayView().setBottom(newHeight);
   }
   rebuildOutline();
}
```

上面代码没有什么难度，在setFrame方法里根据新边距和旧边距，判断尺寸是否发生变化，如果变化就去调用sizeChange方法，onSizeChanged就是在sizeChange中调用的。
