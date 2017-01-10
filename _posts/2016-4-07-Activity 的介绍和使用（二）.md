---
layout: post
title: Activity 的介绍和使用（二） 
tags:
- Android
categories: Android,Activity
description: Activity 的介绍和使用（二）
---


上一节介绍了Activity的生命周期，这一节介绍Activity的屏幕旋转对生命周期的影响，以及Activity的启动模式。


## 手机屏幕旋转
当手机屏幕旋转，有的App的屏幕会闪一下，这是为什么呢？这是因为当手机的重力感应器打开后，旋转屏幕方向，重力感应器发出旋转屏幕的感应，当前显示的Activity会被销毁，然后再重新新建。这就造成了屏幕会闪一下的原因。当然也有的App遇到这样情况下，屏幕不会闪，因为它设置了限制，在屏幕旋转的时候不让Activity重建。下面我们就简单说一下屏幕旋转对Activity的影响。
#### 手机屏幕旋转对生命周期的影响
 在一般情况下，没有特殊设置，手机屏幕旋转会造成Activity的重建。
正常情况下，当我们启动一个新的Activity时，一般会依次调用：
onCreate->onStart->onResume。
当我们旋转屏幕，Activity会销毁，依次调用：
onPause->onStop->onDestroy,
然后再重建，依次会调用： 
onCreate->onStart->onResume 。
也就是说这个Activity新建后如果旋转屏幕，会依次调用以下生命周期：
onCreate->onStart->onResume->onPause->onStop->onDestroy->onCreate->onStart->onResume。这时Activity销毁时就会在onPause之后调用 onSaveInstanceState，在重建时onStart之后 onRestoreInstanceState。以用来保存和恢复界面状态。

####如何禁止手机屏幕旋转
禁止手机的屏幕旋转，必须让Activity有一个固定朝向，要么是横屏，要么是竖屏。那么我们如何设置呢？Android 提供了一个属性:`android:screenOrientation`，用它来限制Activity的朝向。它提供很多设定值，但是我们比较常用的是，横向(`landscape`)和竖向(`portrait`),设置两个值中的一个，屏幕就会按照设定值显示，不再会变化。这个属性需要在清单文件（AndroidMenifest.xml）的`<activity></activity>`节点下设置。
####如何设置屏幕旋转但Activity不在重建
Android 提供了 `android:configChanges`用来限制Activity的重建。这个属性也需要在清单文件（AndroidMenifest.xml）的`<activity></activity>`节点下设置。

Andorid 3.2以前的SDK可以使用如下配置
android:configChanges="orientation|keyboardHidden"
而Adnroid 3.2以后的SDK必须添加一个screenSize属性，如下
android:configChanges="orientation|screenSize"。这样Activity可以随着屏幕的旋转而旋转，而不会重建。


#### 如何在代码里获取屏幕横竖的变化
Activity 提供了一个`onConfigurationChanged`,当屏幕的朝向放生变化时，获取当前屏幕方向。我们可以覆写它来监听变化：

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        switch (newConfig.orientation) {
            case Configuration.ORIENTATION_LANDSCAPE:
                LogUtils.e("当前屏幕为横屏");
                break;
            case Configuration.ORIENTATION_PORTRAIT:
                LogUtils.e("当前屏幕为竖屏");
                break;
            default:
                break;
        }
    }

#### 如何在代码里设置屏幕的旋转
Activity提供了一个叫做 `setRequestedOrientation(int)`,来设置屏幕旋转，它的参数定义在`ActivityInfo`类中，如果我们需要设置屏幕横向：

	setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);

竖向：
	
	setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);

这些参数就是我们在清单文件中的 `android:screenOrientation`的参数。

## Activity的启动模式

### Task(任务) 和 Back Stack(回退栈)
在说Activity的启动模式之前，我们需要了解一下Activity的 Tasks(任务) 和 Back Stack(回退栈)。

我们的App一般都会包含多个Activity，每个Activity都会负责不同的事情。Task就是用户为了某一个特定的工作时进行交互的Activity的集合。
这些Activity统一放到了一个栈里，这个栈就叫做回退栈，也可以叫做任务栈，调用栈等；这样我们就可以通过回退栈来管理这些的Activity。

当我们点击手机桌面的应用图标进入应用后，应用的Task就进入了前台，如果没有Task存在(这个应用最近没有被使用过)，Android就会新建一个Task，并且把主Activity作为当前任务栈的根Activity（栈底Actvitiy）。

默认情况下，我们每启动一个的Activity，就会放到Activity的回退栈中，就像是子弹压入弹夹，在栈顶的Activity才会显示到界面上。如果我们需要finish掉所有的Activity，需要从栈顶一个一个把Activity移除。就像下图：
![](http://7xrxe7.com1.z0.glb.clouddn.com/diagram_backstack.png)
当启动应用时 Activity1作为主Activity首先进入前台，它会被放在回退栈的最上面，当我们通过Activity1启动Activity2之后，Activity2被押入栈，Activity2占据了栈顶的位置，Activity1就处于停止状态(stopped)，我们再通过Activity2启动Activity3，Activity就会占据栈顶的位置，当我们点击返回键，栈顶的Activity3就会弹出栈顶，被销毁回收掉。如果再依次点击返回，剩下的两个Activity也会被依次销毁，直到回到桌面。当所有的Activity从栈中移除，Task就不再存在了。

Task是一个有粘性的组合，可以回到后台，当我们开启一个新的Task或者点击Home键回到桌面，原有的Task就会进入后台，当前Task的回退栈里所有的Activity都出与Stopped状态，但是这个回退栈还是完整存在的，这个Task的位置被新的Task占据，只是失去了焦点而已。当然既然Task回到后台也能重新回到前台。举个例子：当我们启动一个App A，就开启一个Task A，我们在这个App里做了一些操作，开启了两个Activity A1 和 A2，这时Task A 的回退栈里就有了两个Activity。然后我们点击Home键回到桌面，选择另一个App B，这是就开启了Task B，我们也在App B里做了一些操作，开启了两个Activity B1 和 B2，这时Task B的回退栈里也有了两个Activity;这时Task B 就占据了Task A的位置，进入了前台，而Task B回到后台；当我们再次点击Home键回到桌面，再次选择点击App A，这样Task A又重新回到了前台，而它的回退栈还是原来状态。

注意：Android可以在后台运行多个Task(任务)，但是如果后台同时运行了太多的Task,系统有可能会为了重新获取内存而销毁它们，使Activity之前状态消失，所以我们一定要特别注意Activity状态的保存。

像上面这种默认的情况，如果我们重复启动了同一个Activity多次，那么在Activity的调用栈中就存在多个相同的Activity，势必会引发很多问题。为了解决这个问题，就有了Activity的启动模式。当然不仅仅是为了解决这个问题，这只是其中之一而已。我们稍后讨论这个，我们先来总结一下Task和Activity在不设置任何条件下的默认行为：

1. 当Activity A启动了Activity B，Activity A 处于stopped状态，但是系统还是保留着Activity A 的界面的状态，如果我们点击返回键销毁Activity B，Activity A 就会重新回到前台，恢复到可见状态，还保留着之前的状态。
2. 当我们点击Home键回到桌面，之前已启动的Task和它所属的Activity都会回到后台，处于stopped状态；系统还会保留着这个Task和其每个Activity的状态。如果我们再次通过点击桌面的应用图标恢复这个Task，它会回到前台，其栈顶的Activity将会显示在屏幕上。
3. 如果我们我们点击返回键，当前的Actvity就会被弹出回退栈并且会被销毁，之前启动的Activity就会回到栈顶，显示到屏幕上；如果Activity销毁后，Android系统不会保留它的任何状态。
4. 我们从别的Task同样可以启动当前Task中的Activity，并且可以启动多次。
5. Activity可以启动自己。

### 管理Task
通过上面我们可以知道Android系统Task和回退栈管理Activity。可是我们如果有一些比较怪异的需求，怎么办呢？如果我们希望一个Activity在启动的时候就单独开辟一个新的Task，或者我们希望当我们启动一个Activity的时候，我们不想重新新建一个Activity，我们希望使用之前实例化好的Activity，或者我们希望除了Task根Activity之外，清空回退栈的所有Activity等等这些需求，那怎么办？？？ 这就要需要用到我之前提过的Activity启动模式了。

### 如何设置启动模式
在AndroidManifest.xml <activity>标签的一个属性中设置`launchMode`。
	
	<activity
	   android:name=".activities.OtherActivity"
	   android:label="@string/title_activity_other"
	   android:launchMode="standard"
	   android:theme="@style/AppTheme.NoActionBar"/>

或者我们可以在使用Intent跳转的时候，使用`setFlags(int flags)`设置特定的flag。下面我们介绍一下Activity提供的四种启动模式。

#### standard 模式（默认模式）
默认的模式，如果我们在AndroidManifest不设置`launchMode`属性,就是默认模式，当然我们也可以显式设置launchMode="standard"。这模式下的Activity可以被实例化多次，也可以属于不同的Task。我们没启动一次Activity，它都会实例化一次;其中的生命周期也会重写走一遍。当我们从别的App启动这个Activity时，它就业属于另外的Task了。

#### singleTop 模式 （栈顶复用）
这个模式名称中含有一个"Top",当然它的特点也与这个单词有关。如果一个Activity已经存在于当前的Task了，并且在栈的顶端，当我们通过Intent开启Activity时，不会在实例化一个新的Activity对象，而是复用了栈顶的Activity。并且会在Activity内部回调 `onNewIntent`方法，而不会再重走它的的生命周期方法。但是如果我们能点击返回键，则不会回到调用`onNewIntent`方法之前的状态。虽然设置了`singleTop`模式，当Activity不在栈顶时，还是会创建新的实例；而且它还可以属于多个Task,并且同一个Task可以含有多个设置了`singleTop`模式的Activity实例。

举个例子，现在我们有一个Task,它的回退栈里有Activity A,B,C,D;其中A是根Activity,在栈底，D在栈顶，如果默认情况下我们再启动D，就会有`A-B-C-D-D`,Activity会重新实例化出一个新的实例，并且会压入栈顶，这就有了两个Activity D。当我们把Activity D 的launchMode设置为singleTop之后，只要Activity D在栈顶，无论我们启动D多少次，都不会再次再次创建，而是调用D中的`onNewIntent`方法，接收启动它的Intent数据。它的回退栈始终都会保持 `A-B-C-D`不变。

#### singleTask 模式 
又叫做 单任务模式和栈内复用模式。既然它的名字中带有一个Task，它必定跟Task有关。它会重新创建一个Task，把设置launchMode为singleTask的Activity单独放到这个Task里。如果启动的Activity已经在新建Task的回退栈中，那么它不会重新实例化这个Activity，而是调用它的`onNewIntent`方法。不管多少次启动这个Activity，都不会重新创建实例。

`singleTask`还有一个非常重要的一个特性，下面我们举个例子说明:现在假设我们有四个Activity A,B,C,D;A和B的launchMode设置为默认模式，而C,D的launchMode设置为singleTask，那么现在就有了两个任务栈，singleTask 的栈中有Activity C，D；而普通的Task栈中有Activity A，B。如果现在Activity A，B所在的栈在前台，而Activity C，D所在栈在后台，如果我们把Activity C,D 所在栈切换到前台，那么整个App的回退栈就会变成一个，它们的排列就像这样：(栈底)A-B-C-D（栈顶），如果我们点击返回键，那么就会依次从栈顶把Activity弹出销毁。如果我们调整一下Activity顺序，显示假设栈内情况是这样的C-D-A-B,如果我们从B中启动Activity C,那么会有什么情况呢？首先Activity C会首先回到栈顶，而且之前在Activity C之上的Activity，都会被销毁。那么最后栈中就只剩下Activity C 了。

#### singleInstance 模式
字面翻译过来就是 单例模式，这个模式表现非常接近于singleTask，系统中只允许一个Activity的实例存在。区别是，如果一个Activity设置了singleInstance模式，那么系统就会新建一个Task，并且这个Task只能有这一个Activity。如果重复启动这个Activity，均不会重建这个Activity，它会实行栈内复用，跟singleTask一样。

**另**：关于启动模式还牵扯到两个Activity的另两个参数 taskAffinity 和 allowTaskReparenting，它们组合起来的情况比较复杂，如果有时间，我们后面再单独讨论。

上面曾经说到，通过`Intent`也可以设置类似的启动模式，我们可以设置Flag来实现这个目的。`Intent`中定义了很多的flag,其中就有能设置Activity启动模式的flag。使用Intent设置这些flag跟在AndroidManifest中设置launchMode功能是一样的。

`FLAG_ACTIVITY_NEW_TASK`:官方文档说相当于在xml里设置 singleTask；但不尽然，具体的大家可以试验一下

`FLAG_ACTIVITY_SINGLE_TOP`:相当于在xml里设置 singleTop。

`FLAG_ACTIVITY_CLEAR_TOP`:设置这个flag之后，当它启动时，在同一个任务栈中的所有位于它上面的Activity都会被销毁。它一般要跟`FLAG_ACTIVITY_NEW_TASK`一块用。

还有其它的flag,我们之后会单独开出来一个 Intent 专题介绍。

该博客参考了Android官方文档：
[Tasks and Back Stack](http://developer.android.com/guide/components/tasks-and-back-stack.html)

