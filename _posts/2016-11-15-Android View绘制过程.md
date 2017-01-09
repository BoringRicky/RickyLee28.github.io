​	View经过测量和布局之后，就剩下绘制了；测量和布局是为了确定View尺寸和位置，而绘制就是就是将我们想显示到界面上的东西画到View上。

​	View的绘制过程在`draw(Canvas)`方法中完成的;这个方法有690多行；但是它做的工作其实很简单，可以总结以下几

 - 列表内容

点：

	1. 绘制背景
	2. 绘制当前View的内容（调用onDraw）
	3. 绘制当前View的所有子View
	4. 绘制装饰部分 (前景色，进度条)

而官方注释给出的步骤是6条：

1. 绘制背景
2. 如果必要，保存画布图层，以防止衰减(不知道这样翻译对不对，原话：If necessary, save the canvas' layers to prepare for fading）
3. 绘制当前View的内容（调用onDraw）
4. 绘制当前View的所有子View
5. 如果必要，绘制衰减的边和保存的图层（即第2步保存的图层）
6. 绘制装饰部分 (前景色，进度条)

具体看一下代码，这里做了一些删减：

```Java
public void draw(Canvas canvas) {
   //……省略对 mPrivateFlags  的赋值
    /* 绘制的6个步骤
     *      1. Draw the background
     *      2. If necessary, save the canvas' layers to prepare for fading
     *      3. Draw view's content
     *      4. Draw children
     *      5. If necessary, draw the fading edges and restore layers
     *      6. Draw decorations (scrollbars for instance)
     */
    // 第一步：Step 1, draw the background, if needed
    int saveCount;
    if (!dirtyOpaque) {
        drawBackground(canvas);
    }

    //如果需要跳过第2步和第5步：skip step 2 & 5 if possible (common case)
    final int viewFlags = mViewFlags;
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
    if (!verticalEdges && !horizontalEdges) {
        //第三步： Step 3, draw the content
        if (!dirtyOpaque) onDraw(canvas);
        //第四步：Step 4, draw the children
        dispatchDraw(canvas);
      
        // Overlay is part of the content and draws beneath Foreground
        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // 第6步：Step 6, draw decorations (foreground, scrollbars)
        onDrawForeground(canvas);

        // we're done...
        return;
    }

    //……省略 第2-6步；这里把所有的步骤都走完了；这种情况会比较少见
}
```

通过源码，我们可以看到绘制只需要4步就可以。

我们调用draw()方法就是将我们需要内容画到UI界面上。draw()方法会依次调用onDraw(Canvas),dispatchDraw(Canvas)；在View 中这两个方法都是空的。所以根据我们在测量和布局的经验，我们一般就是要重写这种空的方法。恩，对的，我们如果想自定义控件时，还是不要去重写draw(Canvas)方法，还是乖乖重写onDraw(Canvas)或者dispatchDraw(Canvas)方法。这两个方法的调用的时机呢？

- 我们需要自定义一个View，而不是一组view，那么我们只需要重写onDraw就OK了。onDraw只是绘制本身的内容。
- ViewGroup已经重写了dispatchDraw(Canvas)方法，它会依次遍历所有的子view，并且会依次调用子View的draw(Canvas)方法。如果我们自己自定义布局，我们就不需要再去重写这个方法了；包括系统提供的那些Layout都没有重写。除非你有的需求比较变态。


ViewGroup重写的dispatchDraw(Canvas)方法，除了会遍历子View的draw(Canvas)方法，去绘制子View，它还会处理它们的动画。

#### invalidate

​	另外这里还涉及一个` invalidate(boolean)`方法，这里传入的是`true`,其实还有一个方法叫做`invalidate()`,是一个没有参数的方法，它就是调用的`invalidate(true)`。这个方法的字面意思是 使作废，那么是使谁作废呢？它的意思是当View可见时，使整个View都作废，那作废之后屏幕不可能什么都不显示，所以之后它会去掉用 draw(Canvas)方法重新绘制。就相当于重新刷新了一下View。当我们有新的内容需要绘制上去的时候，就可以去调用这个方法。

​	这个方法只能在UI线程下调用，如果实在子线程调用，可以调用postInvalidate方法。

#### requestLayout

​	当View发生变化时，比如尺寸和位置；之前的View已经作废了，那么这个方法就会去安排去重新走一遍布局的流程，如果布局的流程正在走，它会等到布局完成再重新走一遍。跟invalidate方法相比，requestLayout方法会重新从 measure—>onMeasure—>layout—>onLayout 重新走一遍,而不会再走绘制流程。而invalidate则只会从dispatchDraw—>draw—>onDraw走一遍。






