---
layout: post
title: Android 进程间通讯之AIDL
tags:
- Android
categories: Android AIDL
description: Android 进程间通讯之AIDL
---

## Android AIDL的使用

####什么是AIDL？
AIDL是Android Interface Definition Language 的英文缩写，，通过AIDL定义的程序接口可以实现服务端与客户端的通信。其实所谓的服务端跟客户端都是我们开发的APP，我们可以简单理解为，其中一个APP提供服务数据，另一个APP可以去获取数据服务提供的数据。我们知道两个APP分属于两个不同的进程，所以说AIDL是用于Android进程之间通信的接口定义语言。

####适用场合
官方文档是这样说的：如果你允许客户端从不同的应用程序为了进程间的通信而去访问你的service，以及想在你的service处理多线程时，才必须使用AIDL；如果不需要进程之间通讯，使用AIDL是多此一举。当我们只要进程之间进行通讯，但不需要在Service里处理多线程时，我们使用Messager会更好。当然Messager也是使用AIDL实现的。

####定义AIDL
AIDL接口使用后缀名为.aidl的文件定义，在AndroidStudio中会有一个专门的文件夹去存放AIDL文件。而eclipse下必须自己去新建一个以.aidl为后缀的包名用来存放AIDL文件。我们这里以AndroidStudio为例。

AIDL 文件使用 Java 语法编写，描述其方法以及方法的参数和返回值，类似Java里的接口，这些参数和返回值可以是任何类型，甚至是其他AIDL生成的接口。
AIDL有一些内置的数据类型
1. Java的8大基本数据类型
2.  String，CharSequence
3. List：是一个列表，可以理解成Java 的list 。用法也类似.但是必须使用内置的已经定义过的数据类型
4. Map：类似Java的Map，但是必须使用内置的已经定义过的数据类型

如果需要使用我们自己定义的对象，需要在AIDL文件里导入这个对象的路径，并且需要定义一个该对象命名的AIDL文件，并且在该文件里声明一下这个类。例如我们需要在AIDL里使用到我们自己定一个User类，那么我们就需要导入这个类的路径：`import com.liteng.app.User;`,并且创建一个`User.aidl`,并在该文件里声明:"parcelable User;"。

 在服务端跟客户端所有的操作都是一样的，包括AIDL文件的路径，声明的代码都必须一样，这样我们才可以使两个APP之间进行通讯。下面我们通过一个简单的demo来看一下。以处理`User`的数据为例。

#####**服务端**
**1. 定义AIDL文件**
	我们首先创建一个服务端名为`appa`.
	在AndroidStudio中有一个快捷的方式去创建AIDL文件,右键点击工程目 录，会弹出以下对话框
	![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%E5%88%9B%E5%BB%BA%E6%96%B9%E5%BC%8F1.png),
		我们点击红框中的`AIDL File`就可以创建我们的AIDL文件，创建完毕之后，会出现一个名叫`AIDL`的文件夹，里面包含我们创建的AIDL文件。
		我们首先创建一个处理`User`数据的AIDL，UserManager.aidl。创建完毕后如下图：
		![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%E5%88%9B%E5%BB%BA%E5%AE%8C%E6%AF%95.png),
		会有一个跟java同级的AIDL文件夹被创建，里面包含我们创建的`User.aidl`。
**2. 编写AIDL文件**
	我们的操作不会太复杂，只要添加`User`，和获取所有的`User`,这两个操作，因为我们涉及到一个User对象，最好提前创建一个User的AIDL。来声明一个User
			
```
// User.aidl
package com.liteng.app.aidl;
//声明一个User对象，如果需要使用User对象需要在其它的AIDL文件中导入该对象
//这里的parcelable可以看做是一个类型，全部为小写，主要作用是告诉AIDL如何找到它，并且知道它实现了parcelable协议
parcelable User;
```
接下来为我们的User操作的AIDL，UserManager.aidl
```
// UserManager.aidl
package com.liteng.app.aidl;
//导入User，必须包含所有路径
import com.liteng.app.aidl.User;
interface UserManager {
    //添加User
    User addUser(in User user);
    //获取所有的User
    List<User> getAllUsers();
}
```

> 注意： 在AIDL文件里输入参数时，需要在前面加上 in ,输出参数时需要加上out。

>AndroidStudio创建AIDL文件时，默认的目录是以我们创建工程时的目录为根目录的，这里我为了在创建Java类时跟其它的普通类区分，在后面又加了一个子包‘aidl’.

**3. 传递数据** 

因为我们之前的用到了一个我们自己的对象User,所以我们也需要声明一个User类，如果我们需要跟我们的AIDL定一个User.aidl对应，那么必须跟它的包名一样，而且必须实现`Parcelable`接口，并且实现里面的`CREATOR`,`writeToParcel`,`describeContents`等。关于`Parcelable`,它是Android自定义对象的序列化接口，类似`Serializable`，只不过它更轻便，更适用于Android。
```
public class User implements Parcelable{
    public String mName;
    public String mAge;
    public int mGender;
    public String mAddress;

    protected User(Parcel in) {
        mName = in.readString();
        mAge = in.readString();
        mGender = in.readInt();
        mAddress = in.readString();
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(mName);
        dest.writeString(mAge);
        dest.writeInt(mGender);
        dest.writeString(mAddress);
    }
}
```
>关于Parcelable的内容大家可以去网上查一下非常简单。`
>要注意是的readFromParcel和writeToParcel`
>readFromParcel : 从parcel中读取对象`
>writeToParcel ：将对象写入parcel`
>readFromParcel和writeToParcel 读取和写入变量的顺序必须一样

**4. 创建服务端的Service** 
当我们创建完毕AIDL之后，我们重新构建一下程序，![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL_rebuild.png)
点击图中的`Rebuild Project`之后，AndroidStudio就会根据我们编写的AIDL，去创建相应的Java类，我们可以在以下路径中找到这个类。
![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%E7%94%9F%E6%88%90%E7%9A%84Java%E7%B1%BB.png)
关于它创建的类的内容，我们就不细说了，只要知道里面有个叫`Stub`的抽象的内部类就OK了，一会我们需要实现这个类未实现的方法。
接下来我们创建一个Service：
```
public class UserMangerService extends Service {
    private final String TAG = "User Manager Service";
    private List<User> mUsers ;

    public UserMangerService() {
        Log.e(TAG,"构造方法");
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e(TAG,"onCreate");
        mUsers = new ArrayList<>();
    }

    @Override
    public IBinder onBind(Intent intent) {
        Log.e(TAG,"onBind");
        return iBinder;
    }

	//创建内部类实现Stub未实现的方法，也就是我们在AIDL文件里定义的方法
    private UserManager.Stub iBinder = new UserManager.Stub() {
        @Override
        public User addUser(User user) throws RemoteException {
            synchronized (mUsers){
                if (mUsers == null || !mUsers.contains(user)) {
                    mUsers.add(user);
                    return user;
                }
            }
            return null;
        }

        @Override
        public List<User> getAllUsers() throws RemoteException {
            synchronized (mUsers){
                if (mUsers != null) {
                    return mUsers;
                }
            }
            return null;
        }
    };
}
```

**5. 在AndroidMenifest.xml里声明Service**
因为我们需要在另一个App里来调用我们写的Service，我们需要隐式调用，所以我们需要在声明Service时给它一个action,通过隐式调用的方式来起activity或者service，需要把category设为default，这是因为，隐式调用的时候，intent中的category默认会被设置为default。

```
        <service
            android:name="com.liteng.app.UserMangerService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.liteng.app.service" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
```
> android:enabled:表示该Service是允许被实例化，true表是允许，false表是不允许，默认值为true。
> android:exported:表示该Service是否允许其它应用调用当前组件。true为允许，false为不允许。

到这里 服务端 就搞定了，接下来我们创建客户端。

#####**客户端**
**1. 新建一个Module**
我们新建一个module`appb`作为客户端。
在客户端，我们需要新建一个module，然后将在服务端创建的aidl文件以及其目录，还有相关的User类拷贝过来；那么你会得到如小图所示
![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%E7%9B%AE%E5%BD%95.png)
这里我创建了两个Module，`appa`和`appb`,其中`appa`为服务端,`appb`为客户端，在图中分别标识的`1`和`2`,在两个Module里要完全一样才行。

 **2. 去绑定Service** 
因为我们需要在Service中用到binder,我们肯定要使用bindService。接下来我们在Activity里去绑定Service。

```
public class MainActivity extends AppCompatActivity {
    private UserManager mManager;
    //我们在服务端的Service里声明的action
    private String action="com.liteng.app.service";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //创建Intent,并声明Action
        Intent intent = new Intent(action);
        //设置服务端的包名（在5.0之后要制定以下包名）
        intent.setPackage("com.liteng.appa");
        bindService(intent,mConnection,BIND_AUTO_CREATE);
    }

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //通过服务端onBind方法返回的binder对象得到UserManager的实例，这里UserManager是通过AIDL自动创建的代码
            //获取到UserManager的实例，我们就可以去调用之前在服务端实现的方法了。
            mManager = UserManager.Stub.asInterface(service);

            //我们通过服务端的方法去添加 一些用户
            try {
                for (int i = 0; i < 5; i++) {
                    User user = new User("李腾" + i, (10 * i) + "", 1, "火星" + i + "号");
                    mManager.addUser(user);
                }
            }catch (RemoteException e){
                e.printStackTrace();
            }

            //然后通过服务器端的方法去获取到所有添加过的用户
            try {
                List<User> userList = mManager.getAllUsers();
                Log.e("添加的患者",userList+"");
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mManager = null;
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //一定要取消绑定Service
        unbindService(mConnection);
    }
}
```
注意：在绑定Service的时候
```
	   //创建Intent,并声明Action
       Intent intent = new Intent(action);
       //设置服务端的包名（在5.0之后要制定以下包名）
       intent.setPackage("com.liteng.appa");
       bindService(intent,mConnection,BIND_AUTO_CREATE);
```
如果要兼容5.0的系统，一定要在intent里设置一下包名，不然后抛出`IllegalArgumentException`异常：
![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%E6%8A%A5%E9%94%99.png)。

那么到这里我们就全部写完了，我们运行一下，先运行 `appa`,然后运行`appb`，那么我们就可以看到下面的log：
![](http://7xrxe7.com1.z0.glb.clouddn.com/AIDL%20log.png)
正好式我之前打印的log的内容。

另外补充一下：
我们之前在AndroidMenifest.xml里声明Service的时候，只声明了`android:enabled` 和`android:exported`，其实还有一个`android:process`;这个属性会给当前的Service指定一个进程;默认是以当前app的包名为进程名。

当我们需要给Service指定进程时，可以在`android:process`设定自己想要的进程名称；例如：

```
        <service
            android:name="com.liteng.appa.UserMangerService"
            android:process=":remote"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.liteng.app.service" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
```
我们在设定的`:remote`,这里的`remote`是随便设定，也可以使用其它的名字，冒号`:`,这个前缀会将`remote`附加到你的包所运行的标准进程名字的后面作为新的进程名称。比如我们现在的设定的进程就就是`com.liteng.appa:remote`。

>另外注意：  (这段话来自于 [骑猪追大象的AIDL学习笔记这篇博客](http://blog.csdn.net/luzhenyuxfcy/article/details/50773954)）
          如果被设置的进程名是以一个冒号开头的，则这个新的进程对于这个应用来说是私有的，当它被需要或者这个服务需要在新进程中运行的时候，这个新进程将会被创建。如果这个进程的名字是以小写字符开头的，则这个服务将运行在一个以这个名字命名的全局的进程中，当然前提是它有相应的权限。这将允许在不同应用中的各种组件可以共享一个进程，从而减少资源的占用。

[整个工程的下载地址](http://download.csdn.net/download/litengit/9603776)
