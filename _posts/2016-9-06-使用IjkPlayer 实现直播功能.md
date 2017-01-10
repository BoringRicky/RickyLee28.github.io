---
layout: post
title: Android 使用IjkPlayer实现直播
tags:
- Android
categories: Android,直播,ijkplayer
description: Android 使用IjkPlayer实现直播
---

最近直播很火，是的，很火很火！

我也想搞一下这个很火很火的东西，怎么办？

那懂ffmpeg吗？

好吧，我不懂，所以我就瞄上了哔哩哔哩的ijkplayer了。

ijkplayer是哔哩哔哩开源的一个播放器，可以支持本地播放，视频直播等功能。

如果你时间很充裕，并且富有学习精神，那你要先去[Github](https://github.com/Bilibili/ijkplayer)上把它的源码clone下来，是的你自己要先去编译它的源代码。那么你需要安装NDK，因为有些复杂，这里不介绍了。我们直接搞现成的。怎么样？你们喜欢吗？不喜欢也没办法，我们就直接搞！

这里使用的是AndroidStudio 2.1.3，我们只需要在build.gradle里声明一下依赖，如下：

```groovy
dependencies {
    # required, enough for most devices.
    compile 'tv.danmaku.ijk.media:ijkplayer-java:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.6.1'

    # Other ABIs: optional
    compile 'tv.danmaku.ijk.media:ijkplayer-armv5:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-arm64:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-x86:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-x86_64:0.6.1'

    # ExoPlayer as IMediaPlayer: optional, experimental
    compile 'tv.danmaku.ijk.media:ijkplayer-exo:0.6.1'
}
```

这里涉及到了64位的处理器；ijkplayer在64位下要求最低SDK版本为21。

我们只需要下面这些，把64位的剔出去：

```groovy
//    # required, enough for most devices.
    compile 'tv.danmaku.ijk.media:ijkplayer-java:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.6.1'

//    # Other ABIs: optional
    compile 'tv.danmaku.ijk.media:ijkplayer-armv5:0.6.1'
    compile 'tv.danmaku.ijk.media:ijkplayer-x86:0.6.1'

//    # ExoPlayer as IMediaPlayer: optional, experimental
    compile 'tv.danmaku.ijk.media:ijkplayer-exo:0.6.1'
```

然后我们需要导入一些代码，这些代码就在这里：

![](http://7xrxe7.com1.z0.glb.clouddn.com/ijkplayer%E5%8C%85%E7%BB%93%E6%9E%84%E5%9B%BE.png)

还有一些string，导入进来就OK，这样我们的工作完成了一大半了，剩下的就简单了。

首先我们在布局里引入ijkplayer的播放视频的控件`IjkVideoView`:

```xml
    <tv.danmaku.ijk.media.widget.media.IjkVideoView
        android:id="@+id/video_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center"/>		
```

然后在代码里这样写：

```java
mVideoPath = getIntent().getStringExtra("stream_addr");
mVideoView = (IjkVideoView) findViewById(R.id.video_view);
// init player
IjkMediaPlayer.loadLibrariesOnce(null);
IjkMediaPlayer.native_profileBegin("libijkplayer.so");
if (mVideoPath != null) {
    mVideoView.setVideoPath(mVideoPath);
}
mVideoView.start();
```

最后声明一下权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

好了。基本步骤完了。

我们去哪里搞数据呢？我们去哪里搞美女的直播视频呢？这里对映客直播说声对不起，我要在你那里搞数据。具体怎么抓的数据，我就不告诉你我用的Charles。具体怎么抓的就不讲了，网上很多讲这个的。

最后我们的效果是这样的：

![](http://7xrxe7.com1.z0.glb.clouddn.com/ijk%E7%9B%B4%E6%92%AD%E6%BC%94%E7%A4%BA.gif)

关于有些头像不显示，因为映客直播的这些图使用的是不同的地址，我没有深究，所有大家凑合着看吧。

还有我改了一些代码，删掉一些不用的代码：

比如AndroidMediaController，调整视频的播放比例。可能有的人在集成ijkplayer的时候遇到视频播放的时候，两边会有白边(蓝边)的事情，其实就是修改一下视频的播放比例就OK了。代码都在`IjkVideoView`里，去设置一下`mCurrentAspectRatio`这个属性就OK。

[源码在这里](https://github.com/TengLeeIT/Living)
> 不知道这样要不要负法律责任，如果犯法，我就删掉源码了！作为法盲，麻烦看到提示一下。
