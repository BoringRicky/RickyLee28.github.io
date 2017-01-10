---
layout: post
title: Android 进程间通讯之Messenger
tags:
- Android
categories: Android,Messenger
description: Android 进程间通讯之Messenger
---


#### Android进程间通讯的方式

​	当我们需要执行 IPC（进程间通信）时，一般有两种方式：AIDL和Messenger。关于AIDL的介绍请参看[Android进程间通讯之AIDL](http://blog.csdn.net/litengit/article/details/52193984)。我们这里只介绍`Messenger`。

​	使用`Messenger`要比使用 AIDL 实现它更加简单，因为 `Messenger` 会将所有服务调用排入队列，而纯粹的 AIDL 接口会同时向服务发送多个请求，服务随后必须应对多线程处理。对于大多数应用，服务不需要执行多线程处理，因此使用 `Messenger` 可让服务一次处理一个调用。如果您的服务必须执行多线程处理，则应使用 [AIDL](https://developer.android.com/guide/components/aidl.html) 来定义接口。当然采用`Messenger`的方法实现IPC，实际上是以 AIDL 作为其底层结构。

#### Messenger 简单介绍

​	`Messenger`会持有一个Handler的引用，我们可以通过使用`Messenger`向Handler发送消息。而且它是机遇消息(Message)实现的进程间的通信。我们可以在一个进程里通过Handler创建一个`Messenger`，并发送消息；然后在另一个进程里，通过Messenger接收并处理发送的消息。`Messenger`的实现是通过Binder进行一些封装；其底层也是使用AIDL实现的。使用Messenger不会出现并发读写问题，因为Messenger是以串行方式工作的，所以如果有大量的请求，不适合使用Messenger。


#### Messenger进程间通信的简单示例

​	跨进程通信，必然包含客户端和服务端。首先创建一个服务端，接收客户端发送的消息

**创建服务端**

```java
public class MyService extends Service {

    private static final int MSG_RECEIVE_CLIENT = 0x001;
    //用来接收客户端发送的消息的Messenger
    private Messenger mMessenger;

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        //通过Handler创建Messenger,用来接收客户端发送的消息
        mMessenger = new Messenger(new ReceiveClientMessageHandler());
    }

    @Override
    public IBinder onBind(Intent intent) {
        //当我们绑定Service时,获取一个IBinder,用来跟跟它相关的Handler通信
        return mMessenger.getBinder();
    }

    private  class ReceiveClientMessageHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what){
                case MSG_RECEIVE_CLIENT:
                    //接收客户端发来的信息
                    //Messenger发送的内容必须是经过序列化过的对像,如果要传递对象需要实现Parcelable接口
                    String clientMsg = msg.getData().getString("key");
                    Toast.makeText(getApplicationContext(), clientMsg, Toast.LENGTH_SHORT).show();
                    break;
            }
        }
    }
}
```

然后在AndroidMenifest.xml里声明一下Service。

```xml
<service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.liteng.service.messenger"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
</service>
```

这样我们的服务端就完成了，接下来我们来完成客户端.

**创建客户端**

为了看起来直观一些，我们添加一个按钮。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.liteng.client.MainActivity">

    <Button
        android:onClick="sendMsg"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bind Service"/>
</RelativeLayout>
```

然后我们在去Activity里去绑定Service；

```java
public class MainActivity extends AppCompatActivity {
    public String action = "com.liteng.service.messenger";
    private static final int MSG_SEND = 0x001;
    private Messenger mMessenger;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent intent = new Intent(action);
        //设置服务端的包名
        intent.setPackage("com.liteng.service");
        //绑定Service
        bindService(intent,mConnection,BIND_AUTO_CREATE);
    }

    public void sendMsg(View view) throws RemoteException {
        //创建Message对象
        Message msg = new Message();
        msg.what = MSG_SEND;
        Bundle bundle  = new Bundle();
        bundle.putString("key","i am from client");
        //Messenger发送的内容必须是经过序列化过的对像,如果要传递对象需要实现Parcelable接口
        msg.setData(bundle);
        mMessenger.send(msg);
    }

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //当跟Service的连接建立时,我们通过IBinder创建一个Messenger来跟服务端进行通信
            mMessenger = new Messenger(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mMessenger = null;
        }
    };
}
```

我们在客户端向服务端发送一句话“i am from client”。接下来我们去运行一下程序，去看看服务端是否会弹出Toast显示这句话。

![](http://7xrxe7.com1.z0.glb.clouddn.com/Messenger%20%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

我们可以看到成功的弹出了我们想要的结果。

上面我们完成了客户端请求，服务端获取客户端的消息。那么服务端接收到消息之后，处理完消息之后如何再发消息给客户端的呢？我们接着说。

**服务端发消息给客户端**

​	首先，我们是在Handler里获取到的客户端发来的消息，那么我们可以根据Message对象去获取发送消息的Messenger，然后将消息发送给客户端。

​	Message提供了一个变量`replyTo`,用来存储发送和接收消息的`Messenger`；具体如何使用呢?我们只需要获取到Messenger之后，重复发送消的步骤就OK。关键我们去看看客户端是如何接收服务端回应消息。

​	既然我们需要在客户端里接收服务端发送的消息，那么我们就必须在客户端实现一个接收消息的`Messenger`,并且提供一个接收消息的`Handler`，如下：

```java
 private Messenger mClientMessenger = new Messenger(new Handler(){
        @Override
        public void handleMessage(Message msgFromService) {
            super.handleMessage(msgFromService);
            switch (msgFromService.what){
                case MSG_RECEIVE:
                    //获取服务端发送过来的消息
                   String fromService=msgFromService.getData().getString("fromService");
                    //将获取到的消息显示到Button上
                 ((Button)findViewById(R.id.btnSendMsg)).setText(fromService);
                    break;
            }
        }
    });
```

然后我们在发送消息的地方设置发送／接收消息的Messenger：

```java
//设置我们在客户端接收消息的Messenger.
msg.replyTo = mClientMessenger;
```

我们设置完毕客户端之后，我们去服务端获取我们接收消息的Messenger：

```java
//当服务端接收到客户端发送的消息之后,我们回应给客户端
//获取到发送给服务端消息的Messenger
Messenger clientMessenger = msgFromClient.replyTo;
Message message = new Message();
message.what = MSG_SEND_CLIENT;
Bundle bundle = new Bundle();
bundle.putString("fromService", "i am from service");
message.setData(bundle);
```

这样我们就可以从服务端往客户端发送消息了；完整代码如下：

**客户端**：

我们只需要一个Activity就可以了：

```java
public class MainActivity extends AppCompatActivity {
    public String action = "com.liteng.service.messenger";
    private static final int MSG_SEND = 0x001;
    private static final int MSG_RECEIVE = 0x002;

    private Messenger mServiceMessenger;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent intent = new Intent(action);
        //设置服务端的包名
        intent.setPackage("com.liteng.service");
        //绑定Service
        bindService(intent,mConnection,BIND_AUTO_CREATE);

    }

    public void sendMsg(View view) throws RemoteException {
        //创建Message对象
        Message msg = Message.obtain();
        msg.what = MSG_SEND;
        Bundle bundle  = new Bundle();
        bundle.putString("key","i am from client");
        //Messenger发送的内容必须是经过序列化过的对像,如果要传递对象需要实现Parcelable接口
        msg.setData(bundle);
        //设置我们在客户端接收消息的Messenger.
        msg.replyTo = mClientMessenger;

        mServiceMessenger.send(msg);
    }

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //当跟Service的连接建立时,我们通过IBinder创建一个Messenger来跟服务端进行通信
            mServiceMessenger = new Messenger(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mServiceMessenger = null;
        }
    };


    private Messenger mClientMessenger = new Messenger(new Handler(){
        @Override
        public void handleMessage(Message msgFromService) {
            super.handleMessage(msgFromService);
            switch (msgFromService.what){
                case MSG_RECEIVE:
                    //获取服务端发送过来的消息
                    String fromService = msgFromService.getData().getString("fromService");
                    //将获取到的消息显示到Button上
                    ((Button)findViewById(R.id.btnSendMsg)).setText(fromService);
                    break;
            }
        }
    });
}
```

**服务端**:

我们的操作几乎都在Service里面：

```java
public class MyService extends Service {

    private static final int MSG_RECEIVE_CLIENT = 0x001;
    private static final int MSG_SEND_CLIENT = 0x002;
    //用来接收客户端发送的消息的Messenger
    private Messenger mMessenger;

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        //通过Handler创建Messenger,用来接收客户端发送的消息
        mMessenger = new Messenger(new ReceiveClientMessageHandler());
    }

    @Override
    public IBinder onBind(Intent intent) {
        //当我们绑定Service时,获取一个IBinder,用来跟跟它相关的Handler通信
        return mMessenger.getBinder();
    }

    private class ReceiveClientMessageHandler extends Handler {
        @Override
        public void handleMessage(Message msgFromClient) {
            super.handleMessage(msgFromClient);
            switch (msgFromClient.what) {
                case MSG_RECEIVE_CLIENT:
                    //接收客户端发来的信息
                    //Messenger发送的内容必须是经过序列化过的对像,如果要传递对象需要实现Parcelable接口，不能直接传递String
                    String clientMsg = msgFromClient.getData().getString("key");
                    Toast.makeText(getApplicationContext(), clientMsg, Toast.LENGTH_SHORT).show();

//                  当服务端接收到客户端发送的消息之后,我们回应给客户端
//                  获取到发送给服务端消息的Messenger
                    Messenger clientMessenger = msgFromClient.replyTo;
                    Message message = new Message();
                    message.what = MSG_SEND_CLIENT;
                    Bundle bundle = new Bundle();
                    bundle.putString("fromService", "i am from service");
                    message.setData(bundle);
                    try {
                        clientMessenger.send(message);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                    break;
            }
        }
    }
}
```

最后我们看一下效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/Messenger%E4%BA%92%E5%8F%91%E6%B6%88%E6%81%AF.gif)

当我们点击按钮时，先向服务端发送消息，弹出吐司“i am from client”,然后从服务端发送消息给客户端，更改按钮的文字"I am from Service"。

这样，客户端服务端互发消息就实现了，注意事项有一下几点：

1. Messenger发送的内容必须是经过序列化过的对像,如果要传递对象需要实现Parcelable接口,否则会有以下异常：![](http://7xrxe7.com1.z0.glb.clouddn.com/Messenger%20%E5%8F%91%E9%80%81%E9%9D%9EPParcelable%20%E9%94%99%E8%AF%AF.png)
2. 不要执行多线程处理,如果您的服务必须执行多线程处理，则应使用 [AIDL](https://developer.android.com/guide/components/aidl.html) 来定义接口
3. 如果需要客户端往服务端发送消息，一定要调用`replyTo`设置发送和接收消息的`Messenger`.



>先写这些吧，其它想起来再写。有什么问题，希望看到的同学给予提示，拜谢！
