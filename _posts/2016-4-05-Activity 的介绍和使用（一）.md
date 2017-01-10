---
layout: post
title: Activity 的介绍和使用（一） 
tags:
- Android
categories: Android,Activity
description: Activity 的介绍和使用（一）
---


#### Activity是什么
Activity是Android的四大组件之一。是用户操作的可视化界面；它为用户提供了一个完成操作指令的窗口。当我们创建完毕Activity之后，需要调用`setContentView()`方法来完成界面的显示；以此来为用户提供交互的入口。在Android App 中只要能看见的几乎都要依托于Activity，所以Activity是在开发中使用最频繁的一种组件。

#### Activity的生命周期
在Android中会维持一个Activity Stack（Activity栈），当一个新的Activity创建时，它就会放到栈顶，这个Activity就处于运行状态。当再有一个新的Activity被创建后，会重新压人栈顶，而之前的Activity则会在这个新的Activity底下，就像枪梭压入子弹一样。而且之前的Activity就会进入后台。

一个Activity实质上有四种状态：

1. **运行中(Running/Active)**:这时Activity位于栈顶，是可见的，并且可以用户交互。
2. **暂停(Paused)**:当Activity失去焦点，不能跟用户交互了，但依然可见，就处于暂停状态。当一个新的非全屏的Activity或者一个透明的Activity放置在栈顶，Activity就处于暂停状态；这个时候Activity的各种数据还被保持着；只有在系统内存在极低的状态下，系统才会自动的去销毁Activity。
3. **停止(Stoped)**:当一个Activity被另一个Activity完全覆盖，或者点击HOME键退入了后台，这时候Activity处于停止状态。这里有些是跟暂停状态相似的：这个时候Activity的各种数据还被保持着；当系统的别的地方需要用到内容时，系统会自动的去销毁Activity。
4. **销毁(Detroyed)**:当我们点击返回键或者系统在内存不够用的情况下就会把Activity从栈里移除销毁，被系统回收，这时候，Activity处于销毁状态。

上面的四种状态稍微笼统的介绍了一下Activity在不同时期的状态，下面我们通过具体的回调方法看一下Activity的生命周期，下面是官网提供的一张Activity在各个状态下回调用的回调方法：![](http://7xrxe7.com1.z0.glb.clouddn.com/activity_lifecycle.png)

这张图包含了Activity从被创建到被销毁的几乎所有生命周期的回调函数。从上面的方法名我们就可以猜到其大概的功能。一会我们会细讲。我们上面说到了Activities的四种状态，但是这些方法是在什么状态下调用的呢？下面看图
![](http://7xrxe7.com1.z0.glb.clouddn.com/Activity%E5%90%84%E4%B8%AA%E7%8A%B6%E6%80%81%E5%AF%B9%E5%BA%94%E7%9A%84%E5%9B%9E%E8%B0%83%E6%96%B9%E6%B3%95.png)

在Activity运行之前，会通过onCreate(),onStart(),onResume();当运行完onResume()之后，Activity就处于上面说的Running/Active 状态了。

当Activity处于暂停状态时，会调用onPause()，这个时候Activity还是见的。当在这个时候Activity恢复到运行状态时，会重新调用onResume()。

当Activity处理停止状态时，会调用onStop()，这个时候如果要恢复到运行状态就会调用一个新的方法onRestart(),然后去调用onStart(),onResume()。

当Activity被销毁时，就会调用onDestroy()，那么如果要恢复Activity的显示就需要重新创建这个Activity；重新去走onCreate(),onStart(),onResume()这三个方法。

虽然通过字面意思可以大概了解这些方法的作用，但是我们还是需要说一下，它们的具体功能。

**onCreate**: 当Activity第一次被创建时调用。是生命周期开始的第一个方法。在这里我们可以做一些初始化的操作，比如:调用setContentView()方法去加载界面，绑定布局里的一些控件，初始化一些Activity需要用到的数据。之后会调用onStart方法.

**onStart**:当Activity正在变为可见时调用。这个时候Activity已经可见了，但是还没有出现在前台还不能跟用户交互。可以简单理解为Actvity已经可见但是还没有出现在前台。之后会调用onResume.

**onResume**:当Activity可以跟用户交互时调用，这个时候，这个Activity位于栈的顶部。跟onStart相比,它们都是表示Activity已经可见，但是onStart调用时Activity还在后台，而调用onResume时，Activity已经进入了前台，可以跟用户交互了。之后会调用 onPause.

**onPause**：当Activity暂停时调用这个方法；在这里我们可以用来保存数据，关闭动画和其它比较耗费CPU的操作；但是在这里做的操作绝对不能耗时，因为如果当前Activity要启动一个新的Activity，这个新的Activity会在当前Activity执行完毕onPause之后才能进入可见状态。这个方法之后一般会调用的方法有onStop或者onResume.

> 注意:在Android3.0之前，调用这个方法之后，Activity可能会在系统内存紧张时被系统回收。

**onStop**:当Activity进入后台，并且不会被用户看到时调用。当别的Activity出现在前台时，或者Activity会被销毁时，调用此方法；在这个方法调用之后，系统可能会在内存不够的情况下回收Activity；在这个方法之后一般会调用onRestart或者onDestroy.

**onDestroy**:这个方法是Activity生命周期中调用的最后一个方法。它会在Activity被销毁之前调用；Activity销毁原因一般是我们调用Activity的finish方法手动销毁，另一个就是系统在内存紧张的情况下去销毁Activity，以用来节省空间。我们可以通过方法 isFinishing 来判断Activity是否正在被销毁。

**onRestart**:这个方法是在Activity处于停止状态后，又回到可视状态时调用。之后会调用onResume.

以上是Activity的生命周期方法，除此之外Activity还有两个方法感觉大家也应该知道，虽然它们不属于Activity的生命周期方法。这俩方法是：*onSaveInstanceState*  和 *onRestoreInstanceState*。，

*onSaveInstanceState*：
它不属于Activity的生命周期方法，不一定会被触发，但是当应用遇到意外情况，比如：**内存紧张**，**用户点击了Home键**或者**用户按下电源键关闭屏幕**等这些情况，这时系统可能会去销毁Activity，这时这个方法就会被调用。这里强调的是系统去回收Activity，而不是我们去手动的销毁Activity。这个方法提供了一个Bundle对象，我们可以通过它来保存一些临时性的状态或者数据。通常这个方法只适合保存临时数据，如果需要数据的持久化保存等操作，还是要在onPause方法里执行才好。当 Activity再次被创建时，Activity会通过onCreate(Bundle)或者onRestoreInstanceState(Bundle)执行的时候，会将提供一个的Bundle对象来恢复这些数据。

*onRestoreInstanceState*：
这个方法调用的前提是，Activity必须是被系统销毁了，在Activity被再次创建时它会在onStart()方法之后被调用。

下面总结一下Activity的生命周期：<br/>

正常情况下Activity的生命周期是: 
onCreate->onStart->onResume->onPause->onStop->onDestroy

1. 对于一个正常的Activity，第一次启动，会依次回调以下方法： 
onCreate->onStart->onResume
2. 当我们打开一个新的Activity或者点击Home键回到桌面后，会依次回调以下方法： 
onPause->onStop。<br>
上面提到过,如果新的Activity是透明的(采用的透明主题)，当前的Activity不会回调onStop.
3. 当我们再次回到原Activity，会依次回调以下方法： 
onRestart->onStart->onResume.
4. 当我们点击返回键后，会依次回调以下方法： 
onPause->onStop->onDestroy.
5. 当Activity被系统回收后，再次被打开，会跟第一次启动的时回调生命周期方法一样（不包含 onSaveInstanceState 和 onRestoreInstanceState）。
6. 我们可以注意到 其中onCreate 跟 onDestroy 是相对的。一个创建一个销毁。并且其只可能被调用一次。按照这种配对方式，我们也可以看出 onStart跟onStop 是配对的，这两个方法可以被多吃调用。onResume 和 onPause 也是配对的，它们一个获取焦点和用户交互，一个正好相反。
7. onStart和onResume,onPause和onStop，这两对方法在功描述差不多，那为什么还要重复存在呢？<br/>其实这两对方法分别代表不同的意义，onStart和onStop 是Activity是否可见的标志，而onResume和onPause是从Activity是否位于前台的标志，它们针对的角度不同。
8. 在onPause里不能做耗时操作，因为如果要启动一个新的Activity，新的Activity必须要在前一个Activity的onPause 方法执行完毕之后才会启动的新的Activity。


上面是正常情况下的生命周期方法调用，下面简单说一下异常情况下Activity的生命周期：

在系统内存不够时会根据优先级杀死Activity。怎么判断Activity的优先级呢？

1. 最高的优先级：在前台显示并且跟用户交互的Activity，优先级最高，
2. 暂停状态的Activity优先级次之：如果Activity没有在前台，但是可见，不可与用户交互，比如弹出一个对话框等。
3. 处于后台Activity优先级最低：执行了onStop方法的Activity优先级最低。它不可见，并且无法跟用户交互。

当系统内存不足，就会按照优先级去销毁Activity，在销毁Activity时会额外的在onPause和onStop之间调用onSaveInstanceState；当要重新创建这个Activity 时，会在onStart方法之后调用onRestoreInstanceState方法。



